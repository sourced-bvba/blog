---
layout: post
category : article
title: "Testing generic REST APIs with RAML"
comments: true
tags : [java]
---

In an [earlier post](http://www.insaneprogramming.be/blog/2014/08/18/rest-documentation-specification) I showed you how to create contract-driven REST services using RAML as a specification framework. However, the RAML tester library does not limit itself to being able to test Spring MVC controllers. You can also use it to test any REST API and check whether it adheres to a RAML specification.<!--more-->

## The CheckingWebTarget class

The raml-tester library has a utility class that checks whether a REST call and the result from that call adhere to a given specification: CheckingWebTarget. You need a JAX-RS client API implementation for this to work, which is a generic client API for REST services. In my example, I'm using Jersey, which is the reference implementation, but you're free to choose another implementation. This is the test:

{% highlight java %}
public class RestApiTests {

    private static final RamlDefinition raml = RamlLoaders.fromClasspath(RestApiTests.class).load("helloworld.raml")
        .assumingBaseUri("http://localhost/");
    private JerseyClient client = new JerseyClientBuilder().build();

    private CheckingWebTarget checking;

    @Before
    public void createTarget() {
        checking = raml.createWebTarget(client.target("http://localhost"));
    }

    @Test
    public void testHelloEndpoint() {
        checking.path("/hello").request().get();
        Assert.assertThat(checking.getLastReport(), RamlMatchers.hasNoViolations());
    }
}
{% endhighlight %}

If for example you want to use RestEASY, you just replace the client declaration with this:

{% highlight java %}
ResteasyClient client = new ResteasyClientBuilder().build();
{% endhighlight %}

The bare bones specification for this service is

{% highlight yaml %}
#%RAML 0.8
---
title: Hello world REST API
baseUri: http://localhost/
version: v1

/hello:
  get:
    responses:
      200:
        body:
          application/json:
{% endhighlight %}

This test will do a call to `http://localhost/hello` and check whether the call is valid according to the RAML specification. It will also check whether the result is valid, so for example the test will fail in this case if the result isn't a JSON document, or when the service returns something else than a HTTP status code 200. If you want a more extensive example, I suggest trying the article I mentioned at the beginning of this article and try to implement it this way, using either Spring MVC or JAX-RS. Or if you want to try something different and quick, try doing it with [Spark](http://sparkjava.com/).

This system allows you to use test just about every REST API using a RAML specification independent of the mechanism you're using to implement that REST API. If you're looking into doing the same thing with API Blueprint or Swagger you'll have to do a bit more work. API Blueprint does not have decent Java support (snowcrash is NOT decent support) and Swagger still really support a design-first methodology. At the moment, if you want test-by-contract-driven development, RAML at the moment is to only way to go in Javaland.
