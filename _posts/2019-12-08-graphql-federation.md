---
layout: post
category : article
title: "GraphQL federation: how to split up your GraphQL APIs internally"
comments: true
tags : [graphql]
---

GraphQL is becoming a trendy alternative to REST when it comes to providing an API to the outside world. For me, it bridges the gap between the old RPC-style APIs and a modern HTTP+JSON approach. Its specification is unambiguous, and there are many implementations of GraphQL APIs out there.

One of the difficulties with GraphQL, however, is the fact that a schema can quickly become very big for your model, and you end up aggregating many services in your GraphQL API. 

For example, let's assume your application wants to expose customers and the invoices of those customers. You may have 2 different services that each provide a part of the data, say a "customer" service, and an "invoice" service. In the classic approach, your GraphQL becomes a BFF (back-end for front-end), and you'll consume both services to provide a single graph. While this seems like an acceptable approach, once your model becomes bigger and bigger, the GraphQL schema and the BFF in itself becomes less maintainable and a convoluted mess of different aggregated services.  

Now, there was an approach called "schema stitching," which allowed for combining different schemas into a single schema, but the process was somewhat complicated, and not all technology stacks supported this approach. Luckily for us, schema stitching is now considered deprecated, and there is a new approach called "federation." The federation specification allows for your APIs to be split up into separate parts, each providing the data they are responsible for. A federation server combines these parts into a single GraphQL API and figures out which services it needs to call to fulfill a query. You can find some information on federation [here](https://www.apollographql.com/docs/apollo-server/federation/introduction/).

Say we have 2 Spring Boot services, implementing a GraphQL API using `graphql-kotlin`. This requires the following dependency:

```xml
<dependency>
    <groupId>com.expediagroup</groupId>
    <artifactId>graphql-kotlin-spring-server</artifactId>
    <version>2.0.0-SNAPSHOT</version>
</dependency>
```

The first service provides customer data, the second the invoice data for a customer. In the end, we want a single API where we can do a query like this:

```
query {
  customerById(id: 1) {
    id
    name
    invoices {
      invoiceNumber
      invoiceAmount
    }
  }
}
```

The implementation of the first service, using the federation support in `graphql-kotlin`, looks like this:

```kotlin
import com.expediagroup.graphql.federation.directives.FieldSet
import com.expediagroup.graphql.federation.directives.KeyDirective
import com.expediagroup.graphql.spring.operations.Mutation
import com.expediagroup.graphql.spring.operations.Query
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication
import org.springframework.context.annotation.Bean
import org.springframework.stereotype.Component

@KeyDirective(fields = FieldSet("id"))
class Customer(
    val id: Int,
    val name: String? = null
)

@Component
class CustomerQuery : Query {
    suspend fun customerById(id: Int): Customer? = Customer(id, "John Doe")
}

@Component
class CustomerMutation : Mutation {
    suspend fun createCustomer(id: Int, name: String): Customer = Customer(id, "John Doe")
}

@SpringBootApplication
class Application {
}

@Suppress("SpreadOperator")
fun main(args: Array<String>) {
    runApplication<Application>(*args)
}
```

The vital thing to note here is the `@KeyDirective`. This annotation adds an `@key` to the GraphQL schema definition of the id field of the `Customer`. Think of this as setting a primary key, something other services can reference.

Now, the second service is where the magic happens:

```kotlin
import com.expediagroup.graphql.federation.directives.FieldSet
import com.expediagroup.graphql.federation.directives.KeyDirective
import com.expediagroup.graphql.federation.directives.ExtendsDirective
import com.expediagroup.graphql.federation.directives.ExternalDirective
import com.expediagroup.graphql.federation.execution.FederatedTypeRegistry
import com.expediagroup.graphql.federation.execution.FederatedTypeResolver
import com.expediagroup.graphql.spring.operations.Mutation
import graphql.schema.DataFetchingEnvironment
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication
import org.springframework.context.annotation.Bean

@KeyDirective(fields = FieldSet("id"))
@ExtendsDirective
class Customer(
    @property:ExternalDirective val id: Int,
    val invoices: List<Invoice>
)

class Invoice(
    val invoiceNumber: Int,
    val invoiceAmount: Int
)

class InvalidIdException : RuntimeException()

val customerResolver = object : FederatedTypeResolver<Customer> {
    override suspend fun resolve(environment: DataFetchingEnvironment, representations: List<Map<String, Any>>): List<Customer?> = representations.map {
        // Extract the 'id' from the other service
        val id = it["id"]?.toString()?.toIntOrNull() ?: throw InvalidIdException()
        val invoices = getInvoicesForCustomer(id)
        Customer(id, invoices)
    }

    suspend fun getInvoicesForCustomer(id: Int) = listOf(Invoice(1, 32), Invoice(2, 33))
}

@Component
class InvoiceMutation : Mutation {
    suspend fun createInvoice(customerId: Int, invoiceNumber: Int, invoiceAmount: Int): Boolean = true
}

@SpringBootApplication
class Application {
    @Bean
    fun federatedTypeRegistry() = FederatedTypeRegistry(mapOf("Customer" to customerResolver))
}
```

Here a couple of things are happening:

- First of all, the `Customer` is being extended using the `ExtendsDirective`. It once again defines the id as the key. However, in the definition of the id field, the `ExternalDirective` annotation is set on the field. This annotation adds `@external` to the GraphQL schema definition of the field, which hints to the federation server that this field needs to be passed in by another service (in this case, the customer service). The invoices field is added to the `Customer`, extending it with this field.
- The most important part: a customer resolver. This object resolves any external fields and allow for returning the extended version of the `Customer`. In this case, it parses the id, retrieves the customer's invoices, and returns its extended version.

One very important thing to note here is that there is no hard dependency between the original customer service and the extended version. The link is a soft one through the id. The customer class in one service does not extend the one in the other.

Both services have almost identical `application.yaml` configuration files except for the port (8080 and 8081):

```yaml
graphql:
  federation:
    enabled: true
  packages:
    - "be.sourcedbvba"

server:
  port: ...
```

Now, when you start up both services, you'll see the schema definition of both services. Combining those two services requires a third service: the federation server. This server is a NodeJS application built by the people behind Apollo. The federation happens via an ApolloGateway, which defines all the external services.

```javascript
const { ApolloServer } = require("apollo-server");
const { ApolloGateway, RemoteGraphQLDataSource } = require("@apollo/gateway");

const server = new ApolloServer({
  gateway: new ApolloGateway({
    debug: true,
    serviceList: [
      { name: "customer-app", url: "http://localhost:8080/graphql" },
      { name: "invoice-app", url: "http://localhost:8081/graphql" }
    ]
  }),
  subscriptions: false
});

server.listen().then(({ url }) => {
  console.log(`Server ready at ${url}`);
});
```

So, in short, you define the two services in a gateway (you can use a schema registry and have a managed solution) and you set up the ApolloServer (which is a GraphQL server) to use that gateway. Two things to note here are:

- `debug: true` will output the query path of any query in the console. This is interesting to see when you want to know which services are being called.
- Subscriptions are not supported with federation. You will need to disable this by setting `subscriptions` to `false`. If your application requires subscriptions, you will not be able to use federation.

When you start up the federation service (node index.js), it combines the 2 schemas into a single schema. At startup, it will also check whether it can combine all the schemas in a single one. If there are some mismatches in the keys, it will throw an error and refuse to start up. When you browse to `http://localhost:4000`, you are greeted with a GraphQL query editor where you can execute the following query, the one we set out to be able to execute:

```
query {
  customerById(id: 1) {
    id
    name
    invoices {
      invoiceNumber
      invoiceAmount
    }
  }
}
```

When you look at the output of the federation server, you'll see it calls both services for their data and it combines the two pieces of data in a single stream. However, if you execute the following query:

```
query {
  customerById(id: 1) {
    id
    name
  }
}
```

You now see that the federation service only calls a single service as you did not ask any data that would require it to call the extended service. If you look at the resulting schema, you will see it has merged both services into a single graph, including the queries and mutations defined in each separate service:

```
type Customer {
  id: Int!
  name: String
  invoices: [Invoice!]!
}

type Invoice {
  invoiceAmount: Int!
  invoiceNumber: Int!
}

type Mutation {
  createCustomer(id: Int!, name: String!): Customer!
  createInvoice(
    customerId: Int!
    invoiceAmount: Int!
    invoiceNumber: Int!
  ): Boolean!
}

type Query {
  customerById(id: Int!): Customer
}
```

With this mechanism, you can easily extend an existing data structure in your GraphQL APIs and split up your APIs into well-defined pieces that each have their responsibility. If you want more information on the specification, you can find it [here](https://www.apollographql.com/docs/apollo-server/federation/federation-spec/). If you currently have a large GraphQL APIs that from a data perspective has too many responsibilities, this might be a interesting route to take.
