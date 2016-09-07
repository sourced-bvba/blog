---
layout: post
category : article
title: "Java RPC dead in the REST age? Neh!"
comments: true
tags : [java]
---

When you're writing webservices nowadays, you can be sure without a doubt that REST will be your first choice and probably your only choice. However, those familiar with REST services also know how hard it is to sometimes map all your business concepts to resources. Sometimes you just need to quickly build something RPC-like that can be invoked with a simple HTTP call and uses JSON like all the cool kids on the block. Enter [JSON-RPC](http://www.jsonrpc.org). <!--more-->

## JSON-RPC

RPC really got a bad name thanks to some of the standards that were used to achieve it. Most developers shudder when confronted with WSDLs and SOAP envelopes. However, RPC still has a lot of use cases, for example for remoting between a business front-end and a dedicated back-end.
Most Spring users are familiar with the remoting features it provides, including HTTP invocation (using Java serialization), JMS and plain old RMI. JSON-RPC can be a very nice drop-in replacement in these circumstances and provide an easy way to test and invoke your API's using only a browser.

JSON-RPC is an official standard, now in its 2.0 version. It uses JSON payloads to define both the request and the response of the RPC call. A standard JSON-RPC call looks like this:

{% highlight json %}
{
  "id":1234,
  "method":"myRpcMethod",
  "params":["test"]
}
{% endhighlight %}

With JSON-RPC you can choose to have a dedicated endpoint per service or a single endpoint, differentiating between the services on the server level by prefixing your method name with a service identifier.

The response will either be a result of the call or a structure returning information of the error in case the call fails.

## Using JSON-RPC with Java

There are a couple of JSON-RPC libraries out there. However, as I've found out, only one is actually worth looking at, especially if you're using Spring: [jsonrpc4j](https://github.com/briandilley/jsonrpc4j). It uses Jackson to provide the mapping between POJOs and JSON and can therefor be easily extended to support a plethora of Java libraries such as serialization support for Joda Time and the new Money API.

With jsonrpc4j exposing a service as a JSON service is dead easy. I'm going to give you the basic setup to expose a service in Spring Boot, but you can find more information in the documentation of the project.

For example, say we have a service that needs to be exposed that looks like this:

{% highlight java %}
public interface MyService {
  String sayHelloWorld(String name);
}

public class MyServiceImpl implements MyService {
  public String sayHelloWorld(String name) {
    return "Hello world, " + name;
  }
}
{% endhighlight %}

To expose this service to JSON-RPC with Spring Boot, this is the config you need with jsonrpc4j:

{% highlight java %}
@SpringBootApplication
public class RpcApplication {
  public static void main(String[] args) {
    SpringApplication.run(RpcApplication.class);
  }

  @Bean
  public MyService myService() {
    return new MyServiceImpl();
  }

  @Bean(name = "/rpc/myservice")
  public JsonServiceExporter jsonServiceExporter() {
    JsonServiceExporter exporter = new JsonServiceExporter();
    exporter.setService(myService());
    exporter.setServiceInterface(MyService.class);
    return exporter;
  }
}
{% endhighlight %}

And that's it. Start up your Boot application and execute the following `curl` command:

{% highlight bash %}
curl -v -X POST -d '{"id":0, "method":"sayHelloWorld", "params":["John Doe"]}' http://localhost:8080/rpc/myservice
{% endhighlight %}

You should then receive the following response:

{% highlight json %}
{"response": "Hello world, John Doe"}
{% endhighlight %}

And that's it for exposing JSON-RPC methods with Spring Boot. Very simple, very quick and very powerful. Personally I like JSON-RPC APIs a lot for internal usage because it has such a small learning curve. While it's definitely not REST, it does allow you to quickly expose a service over an HTTP interface with JSON datastructures. JSON-RPC APIs can be an excellent addition to your REST APIs. There's no doubt that REST APIs are preferred for external web services, but for internal communication or internal APIs, JSON-RPC can provide a quick alternative to externalize services without having to worry about mapping everything to RESTful resources.
