---
layout: post
category: article
title: "You should choose GraphQL over REST and here's why"
comments: true
tags : [graphql]
---

When building a back-end application, a external API becomes almost a given in the requirements. As an industry, we made great strides in that field, going from CORBA, XML-RPC, SOAP to the now ubiquitous REST(ful?) APIs. Compared to those first APIs, REST was and most of the times still is a fantastic API mechanism. Making a REST API is as easy as it can be, especially in the JVM ecosystem, there are literally tens if not hundreds of REST frameworks, with JAX-RS and Spring MVC probably being amongst the most popular ones in terms of numbers of applications. But I've stepped away from REST. And it's been an interested road so far. I used to be a big fan of REST APIs, I've even often advocated HATEOAS, adding hypermedia to my REST APIs to make them more "discoverable". But as more people, I evolve and so do the tools I work this. 

I now almost exclusively use GraphQL and I'd like to give you the reasons why so you can decide for yourself whether it's the better choice in your specific case or not. Everything I'll be saying is influenced by how I write software (see my countless articles on Clean Architecture) and if you have another way of writing your software, it may or may not be a better fit. But atleast here's the information.

## It's conceptually easier

There are basically 2 types of data operations in software: read and write. When you look at REST, reading is easy, that's a GET operation. However, write has specific rules, being split up in POST, PUT and DELETE. You can even dive in the deep end of the pool and start using PATCH. It's fairly confusing to some users and more often than not, the basic rules on when to use for example POST and PUT are broken on a regular basis. 

GraphQL knows 2 basic operations: queries and mutations. Queries read data, mutations change data. That's it.

## Synthetic resources and RPC-like calls

Say you want to have an API that reboots a device. How would you do this in a truly RESTful manner? As everything in REST has to be a resource, you'll probably start introducing something like a reboot request, so you can do `POST .../device/123/reboot-request` or `POST .../device/123/powerstate`. Everything in REST has to be a noun. However, this is not how I've seen this done in the past. I've seen `POST .../device/123/reboot`, which is incorrect because print isn't a noun, but a verb. You're basically doing RPC here through a mechanism that looks like REST. I've done this very often, just because synthetic resources are just weird. But I wasn't doing REST anymore.

With GraphQL, you're basically confronted with an RPC mechanism. All mutations have clear names and indicate intent very clearly. A mutation for a reboot would look like this:

```
mutation {
    rebootDevice(deviceId: 'foo')
}
```

You don't have to invent synthetic resources in order to do something like this.

I know that some people think RPC is a step backwards, but take a good hard look at your REST APIs: there's a very big chance you're doing RPC somewhere because you had no other choice. RPC is much easier to understand, but we had some really bad experiences with RPC style web APIs like SOAP that scared away a lot of people. Even before GraphQL I used JSON-RPC in a lot of cases if REST wasn't a requirement.

## Much easier to understand link with use cases

When working in a Clean Architecture environment, use cases are the entry point to your domain. They tell you what you can do with the system. Translating those use cases to REST APIs is at times non-trivial because of the reasons I stated above. With GraphQL, it couldn't be any easier. Your read use cases can be very easily translated to GraphQL queries, your write use cases to mutations. You can use the same language in your GraphQL API, which makes the entire codebase and the data flow much easier to understand.

## GraphQL is transport agnostic

Most people that use GraphQL have an HTTP endpoint that allows for executing GraphQL queries and mutations. It's basically a single REST endpoint (hah, go figure) where you can POST a GraphQL query or mutation to. So from a consumer level, there's really not that much difference.

However, HTTP is not the only communication transport that is possible with GraphQL. Any transport that is able to send text from one place to another is suitable for use with GraphQL: messaging like JMS or AMQP, raw TCP sockets, websockets or even carrier pigeons. All of them are GraphQL compatible. This allows for way more flexibility than what is possible through REST. 

## Query granularity

With GraphQL, you decide which data you want to see returned. If you don't want a piece of data in your end result, you omit it from your query. With REST, the resource returns what the resource returns. If you only need 1 field in the JSON structure, too bad, you're getting the rest too.

More interestingly is that GraphQL allows you to think in terms of object graphs. Take the following query:

```
query {
    invoice(id: `123`) {
        invoiceNumber
        date
        dueDate
        customer {
            name
            vatNumber
            address {
                street
                number
                zipCode
                city
            }
        }
        invoiceAddress {
            street
                number
                zipCode
                city
        }
        invoiceItems {
            product {
                name
                imageUrl
            }
            price
            quantity
        }
    }
}
```

Say that your want to model a REST API for this. You have a couple of choices here: you can have `GET .../invoice/123` return the entire object graph. That's certainly a possibility, but it's very inefficient. Say that you only need the invoiceNumber, date and dueDate fields, you're forced to get all the other (something expensive to get) data. So you fix that by using references and HATEOAS in your returned data. You don't return the customer, but a reference and a link to where the data of the customer can be found. You do the same thing with invoiceItems, product, ... Now to get that one invoice, you may end up with tens of calls to your backend if you need the entire structure. Another option would be to used specific projections for specific purposes, i.e. `GET .../invoice/123` and `GET ..invoice/123/full` or worse, use MIME types to see what kind of data the consumer wants (don't do this!).

With GraphQL, you decide which data your want in a single call and the backend system will decide where it will get the data. It may get it from a single use case, perhaps from multiple use cases (using data fetchers) or maybe from an entirely different system if you're using GraphQL. But you don't have to bother your user with this, it's transparent to him: you're just telling him which data he can possibly get.

## Schema makes making clients dead easy

Not unlike SOAP and it's WSDL, GraphQL also has a schema definition that determines which queries are possible, what datastructures the queries can return, what mutations are possible and what the parameters can be.

```
type Query {
    devices: [Device!]
}

type Mutation {
    rebootDevice(deviceId: String!)
}    

type Device {
    id: String!
    name: String!
}
```

The schema tells you what you can do with the API. The really cool part is that there are a lot of tools out there that can read the schema and generate client code that allows you to easily call that API in a typesafe manner (or at least provide autocompletion in your code).

When developing your implementation, you can also use the schema system in two ways. You can either write code and have tooling generate the schema, or write the schema and write/generate the serverside code that adheres to the schema. Either way works and it doesn't really matter. I use the schema-first approach because it becomes easier to spot when someone would introduce a breaking change in an API: if I see a line changed or deleted in a GraphQL schema, it's cause for a closer look.

## Great tooling

Because of the schema inherent in GraphQL and the fact that you can even do introspection on a GraphQL API while it is running, there's a lot of good tooling out there to interact with a GraphQL API: Altair, Playground, ... They all provide a way to easily write GraphQL queries and mutations and have autocompletion thanks to the schema and introspection feature.

## Versioning

While you can version a GraphQL API just like any other REST API, GraphQL allows for for the continuous evolution of a schema in a backwards compatible manner.

Any change can be considered a breaking change when you don't have control over which data a service returns and breaking changes imply a new version. If you add features on a regular basis, a tradeoff becomes apparent between releasing often and having a fair bit of versions versus the maintainability of your API. Your consumer might not be too happy...

GraphQL only returns the data that's explicitly requested in a query, so you can add new types and fields on new and existing types without having a breaking change. This basically enables you to make a versionless API.

## Conclusion

GraphQL to me is beyond better when compared to REST. It allows for the ease of integration that REST has by having it exposed as a HTTP service, but expands beyond that with query granularity, more natural API usage (RPC style and using business language instead of a combination of a noun with an HTTP method).

From an implementation standpoint, it's the exact same amount of work as implementing a REST API is, so that's a non-differentiator for me. 

The biggest resistance you'll encounter with GraphQL is that people don't know it. People are used to (bad) REST. They resist change and fear the unknown. I've seen this myself and luckily the solution is dead simple: give a demo. It took me under 5 minutes to convince the most skeptical and resistant to change people I know. GraphQL just "clicks".

Give it a try. You won't go back.