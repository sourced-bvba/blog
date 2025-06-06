---
layout: post
category: article
title: "Implementing Clean Architecture with Quarkus"
comments: true
tags : [quarkus]
---

Quarkus is quickly becoming a framework to be reckoned with and because of this, I decided to give it another go and see to what degree it's possible to uphold Clean Architecture (CA) principles when writing Quarkus applications.

My starting point is a basic Maven project that has the 5 standard modules for a CRUD REST application when doing CA: 

- `domain`: The domain entities and the gateway interfaces for those entities
- `app-api`: The use case interfaces of the application
- `app-impl`: The implementation of those use cases using the domain. Depends on `app-api` and `domain`.
- `infra-persistence`: Implementing the gateways defined by the domain with a database API. Depends on `domain`.
- `infra-web`: Exposing the use cases to the outside world using REST. Depends on `app-api`.

Additionally, we'll create a `main-partition` module that will serve as the deployable artifact for the application.

When you want to use Quarkus, the first thing you want to do is add the BOM to the parent POM of your project. This BOM will manage all the versions of the dependencies you'll be using. You'll want to configure the standard plugins for maven projects in your plugin management as well, like the compile and surefire plugin. As we'll be using Quarkus, you also configure the Quarkus plugin here in plugin management. Last but not least you are going to configure a plugin to run for every module (so in `<build><plugins>...</plugins></build>`), which is the Jandex plugin. Because Quarkus uses CDI, the Jandex plugin adds an index file to every module that contains every annotation used in that module and the link to where it's used. This will make life with CDI a lot easier and lead to less work later down the line.

Now that the base structure is built, we can start by building a functioning application. To accomplish this, we'll first make sure the `main-partition` creates an executable Quarkus application. This mechanism can be found in every quickstart example Quarkus provides.

First, you'll configure the build to use the Quarkus plugin:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>io.quarkus</groupId>
      <artifactId>quarkus-maven-plugin</artifactId>
      <executions>
        <execution>
          <goals>
            <goal>build</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

Next, you'll want to add dependencies to every module of your application, in addition to the `quarkus-resteasy` and `quarkus-jdbc-mysql` dependencies. You can change that last dependency to the database of your choice (keeping in mind that if you want to go the native route later, you cannot use an embedded database like H2).



Optionally, you can add a profile that allows you to build a native application later on. This does require you to have additional tooling on your development rig (GraalVM, `native-image` and XCode if you're using OSX).

```xml
<profiles>
  <profile>
    <id>native</id>
    <activation>
      <property>
        <name>native</name>
      </property>
    </activation>
    <properties>
      <quarkus.package.type>native</quarkus.package.type>
    </properties>
  </profile>
</profiles>
```

Now if you run `mvn package quarkus:dev` from the project root, you'll have a running Quarkus application! You won't see a lot, because we don't have any controllers or content yet.

## Adding a REST controller

For this exercise we'll work from the outside-in. First, we'll create a REST controller which will return customer data (which for this example consists of only a name).

In order to use the JAX-RS API, you need to add a dependency to the `infra-web` POM:

```xml
<dependency>
  <groupId>io.quarkus</groupId>
  <artifactId>quarkus-resteasy-jackson</artifactId>
</dependency>
```

The basic controller code looks like this:

```java
@Path("/customer")
@Produces(MediaType.APPLICATION_JSON)
public class CustomerResource {
    @GET
    public List<JsonCustomer> list() {
        return getCustomers.getCustomer().stream()
                .map(response -> new JsonCustomer(response.getName()))
                .collect(Collectors.toList());
    }

    public static class JsonCustomer {
        private String name;

        public JsonCustomer(String name) {
            this.name = name;
        }

        public String getName() {
            return name;
        }
    }
```

If we now run the application, you'll be able to call `http://localhost:8080/customer` and see `Joe` in JSON format.

## Adding a use case

Next up, we're going to add a use case and an implementation for that use case. In `app-api` we'll define the following use case:

```java
public interface GetCustomers {
    List<Response> getCustomers();

    class Response {
        private String name;

        public Response(String name) {
            this.name = name;
        }

        public String getName() {
            return name;
        }
    }
}
```

In `app-impl` we'll create an basic implementation for that interface.

```java
@UseCase
public class GetCustomersImpl implements GetCustomers {
    private CustomerGateway customerGateway;

    public GetCustomersImpl(CustomerGateway customerGateway) {
        this.customerGateway = customerGateway;
    }

    @Override
    public List<Response> getCustomers() {
        return Arrays.asList(new Response("Jim"));
    }
}
```

In order for CDI to see the `GetCustomersImpl` bean, you'll need to use a custom `UseCase` annotation as defined below. You can also use the standard `ApplicationScoped` and `Transactional` annotation, but creating your own annotation allows you to group those logically and decouple your implementation code from frameworks like CDI. 

```java
@ApplicationScoped
@Transactional
@Stereotype
@Retention(RetentionPolicy.RUNTIME)
public @interface UseCase {
}
```

In order to use the CDI annotations, you'll have to add the following dependencies to the POM of `app-impl` in addition to dependencies to `app-api` and `domain`.

```xml
<dependency>
  <groupId>jakarta.enterprise</groupId>
  <artifactId>jakarta.enterprise.cdi-api</artifactId>
</dependency>
<dependency>
  <groupId>jakarta.transaction</groupId>
  <artifactId>jakarta.transaction-api</artifactId>
</dependency>
```

Now, we need to change the REST controller to use the `app-api` use cases.

```java
...
private GetCustomers getCustomers;

public CustomerResource(GetCustomers getCustomers) {
    this.getCustomers = getCustomers;
}

@GET
public List<JsonCustomer> list() {
    return getCustomers.getCustomer().stream()
            .map(response -> new JsonCustomer(response.getName()))
            .collect(Collectors.toList());
}
...
```

If you now run the application and call `http://localhost:8080/customer`, you'll see `Jim` in JSON format.

## Defining and implementing the domain

So, next up: the domain. The `domain` here is quite simple, it consists of `Customer` and a gateway interface to get customers.

```java
public class Customer {
	private String name;

	public Customer(String name) {
		this.name = name;
	}

	public String getName() {
		return name;
	}
}
```

```java
public interface CustomerGateway {
	List<Customer> getAllCustomers();
}
```

We also need to provide an implementation of the gateway before we can start using the gateway. In `infra-persistence`, we'll provide such an interface.

For this implementation, we'll use the JPA support in Quarkus and also use the Panache framework to make life a bit easier. You'll have to add the following dependency to `infra-persistence` in addition to `domain`:

```xml
<dependency>
  <groupId>io.quarkus</groupId>
  <artifactId>quarkus-hibernate-orm-panache</artifactId>
</dependency>
```
First, we define the JPA entity for the customer.

```java
@Entity
public class CustomerJpa {
	@Id
	@GeneratedValue
	private Long id;
	private String name;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
}
```

With Panache, you can opt to either have your entities extend `PanacheEntity` or use a repository/DAO pattern. I'm not really a fan of the ActiveRecord pattern, so I opt to use the repository, but the choice is up to you.

```java
@ApplicationScoped
public class CustomerRepository implements PanacheRepository<CustomerJpa> {
}
```

Now that we have our JPA entity and a repository, we can implement the `Customer` gateway.

```java
@ApplicationScoped
public class CustomerGatewayImpl implements CustomerGateway {
	private CustomerRepository customerRepository;

	@Inject
	public CustomerGatewayImpl(CustomerRepository customerRepository) {
		this.customerRepository = customerRepository;
	}

	@Override
	public List<Customer> getAllCustomers() {
		return customerRepository.findAll().stream()
				.map(c -> new Customer(c.getName()))
				.collect(Collectors.toList());
	}
}
```

We can now change the code in our use case implementation to use the gateway.

```java
...
private CustomerGateway customerGateway;

@Inject
public GetCustomersImpl(CustomerGateway customerGateway) {
    this.customerGateway = customerGateway;
}

@Override
public List<Response> getCustomer() {
    return customerGateway.getAllCustomers().stream()
            .map(customer -> new GetCustomers.Response(customer.getName()))
            .collect(Collectors.toList());
}
...
```

We cannot run our application yet, because we now need to configure our Quarkus application with the necessary parameters for persistence. In the `src/main/resources/application.properties` in `main-partition`, add the following parameters.

```properties
quarkus.datasource.url=jdbc:mysql://localhost/test
quarkus.datasource.driver=com.mysql.cj.jdbc.Driver
quarkus.hibernate-orm.dialect=org.hibernate.dialect.MySQL8Dialect
quarkus.datasource.username=root
quarkus.datasource.password=root
quarkus.datasource.max-size=8
quarkus.datasource.min-size=2
quarkus.hibernate-orm.database.generation=drop-and-create
quarkus.hibernate-orm.sql-load-script=import.sql
```

In order to see some initial data, we'll also add an `import.sql` file in the same directory that adds some data.

```sql
insert into CustomerJpa(id, name) values(1, 'Joe');
insert into CustomerJpa(id, name) values(2, 'Jim');
```

If you now run the application and call `http://localhost:8080/customer`, you'll see `Joe` and `Jim` in JSON format. We now have a complete application from REST to DB.

## Native

If you want to build a native application, you'll need to build your application using the `mvn package -Pnative` command. This can take a couple of minutes, depending on your development rig. Quarkus is quite fast when starting up without native support, around 2 to 3 seconds, but when compiled to a native executable using GraalVM, this is reduces to under 100 milliseconds. That's blazingly fast for a Java application.

## Testing

Testing a Quarkus application can be done by using the Quarkus test framework. If you annotate a test using `@QuarkusTest`, JUnit will start up a Quarkus context before executing a test. A test for the entire application in `main-partition` would look like this:

```java
@QuarkusTest
public class CustomerResourceTest {
	@Test
	public void testList() {
		given()
				.when().get("/customer")
				.then()
				.statusCode(200)
				.body("$.size()", is(2),
						"name", containsInAnyOrder("Joe", "Jim"));
	}
}
```

## Conclusion

Quarkus is in many regards a fierce competitor for Spring Boot. Some things, in my opinion, it does even better. Even though there is a framework dependency in my `app-impl`, it's only a dependency for annotations (with Spring, adding `spring-context` to get `@Component` means adding a lot of core Spring dependencies as well). If you do not like this, you can also add a Java file to the main-partition that uses CDI's `@Produces` and creates a bean there, in which case you don't need any additional dependencies in `app-impl`. But for some reason, I mind the `jakarta.enterprise.cdi-api` dependency less than I would the `spring-context` dependency there.

Quarkus is fast, really fast. It's faster than Spring Boot for this type of applications. As Clean Architecture forces most if not all framework dependencies to the edge of the application, choosing between Quarkus and Spring Boot becomes a non-event. The advantage that Quarkus has at this point is that is built with GraalVM support in mind and therefor can become a native application with little effort. With Spring Boot, your mileage may vary at this point in time, but I'm quite certain that they will catch up soon.

However, playing around with Quarkus also made me realize in what world of trouble classic Jakarta EE application servers are in with players like Quarkus. There is not a lot you cannot do yet with Quarkus. [The Quarkus code generator](https://code.quarkus.io) has support for a lot of technologies, some of which are non-trivial to do in a Jakarta EE context with a traditional application server. It has all the bases covered for people familiar with Jakarta EE and the development experience is much smoother. It'll be interesting to see how the ecosystem will handle competition like this.

There's a lot on Quarkus I still have to learn. It apparently also has out of the box support for AWS Lambda, so creating serverless applications with it should prove to be an interesting experience as well. More to come. 

The code for this article can be found on [Github](https://github.com/lievendoclo/clean-arch-quarkus).