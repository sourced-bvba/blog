---
layout: post
category : article
title: "RAML: How specification becomes documentation and testing"
comments: true
tags : [java]
---

In my last post I talked about what annoys me about Swagger. This evening, I took the time to see whether there are any good alternatives out there. As it seems, there are a lot of them. Two of them I found especially worth looking at: API BluePrint and RAML. This article is about RAML, but I'll definitely post another one on API BluePrint.

RAML is a specification format that looks like a YAML file. It describes how a REST webservice should look like and how it should behave with regards to return values. In short, it's what WSDL is to SOAP webservices. RAML specifications are not hard to write and is a top-down approach to REST webservices, unlike Swagger, which is a bottom-up tool mainly aimed towards documentation. You'll write the RAML specification before writing the code.<!--more--> 

The reason why I find RAML so compelling as a Java dev is the fact that the specification can actually be used in a unit test. This means you can code by contract and truly do test driven development: you now have a spec your service needs to adhere to and which can be verified.

For example, I need a service with this RAML specification:

{% highlight yaml %}
#%RAML 0.8
---
title: simple
baseUri: http://boottest/simple/{version}
version: v1

/hello:
  get:
    responses:
      200:
        body:
          application/json:
            schema: |
              {
                "type":"object",
                "properties": {
                  "stringValue":{
                    "type":"string"
                  },
                  "integerValue":{
                    "type":"integer"
                  }
                }
              }
{% endhighlight %}

Using TDD principles I write a Spock test using the raml-tester library.

{% highlight groovy %}
@ContextConfiguration(loader = SpringApplicationContextLoader, classes = BootApplication)
@WebAppConfiguration
class HelloRestServiceSpecification extends Specification {
    private static RamlDefinition api = RamlLoaders.fromClasspath(HelloRestServiceSpecification.class).load("api.raml")
        .assumingBaseUri("http://boottest/simple/v1")
    private static SimpleReportAggregator aggregator = new SimpleReportAggregator();

    @ClassRule
    public static ExpectedUsage expectedUsage = new ExpectedUsage(aggregator);

    MockMvc mockMvc

    @Autowired
    private WebApplicationContext wac;

    def setup() {
        mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
    }

    @Test
    def "check hello controller"() {
        expect:
        mockMvc.perform(get("/hello").accept(MediaType.parseMediaType("application/json")))
                .andExpect(api.matches().aggregating(aggregator))
    }
}
{% endhighlight %}

Naturally, this test will fail (it'll say a 404 isn't in the spec). Now I can write a REST controller that adheres to the specification.

{% highlight groovy %}
@RestController
public class HelloController {

    @RequestMapping("/hello")
    @ResponseBody
    public TestObject index() {
        return new TestObject("Greetings from Spring Boot!", 5);
    }

    @Canonical
    static class TestObject {
        String stringValue
        Integer integerValue
    }
}
{% endhighlight %}

If I make a mistake, for example making the integerValue field a String, the test will fail. From a TDD point of view, this is excellent. 

As for documentation, RAML is backed by MuleSoft and they have built an API console that can read a RAML file and show  HTML5 documentation (using AngularJS) for that RAML file. This documentation looks a lot like Swagger, IMHO it's even better. The API console is open source and can be embedded in an existing AngularJS application.  Like Swagger, it also allows you to try out the API through the documentation. 

While we all have bad memories with WSDL files and code by contract, RAML actually seems a great tool, especially in outsourced environments. Say you have your REST services developed in Bangalore, then RAML based unit tests can be a life saver (TDD is to me the only way globalized development can succeed). In the current age of prototype-based development and continuous delivery RAML may seem like overkill. However, continuous delivery relies heavily on testing in order to be succesful and RAML will certainly help to avoid a lot of issues. The fact that you get free, good-looking documentation is just a bonus at that point.

Next up: API Blueprint. 
