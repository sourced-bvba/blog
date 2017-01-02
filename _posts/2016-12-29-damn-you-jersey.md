---
layout: post
category : article
title: "Damn you, REST libraries"
comments: true
tags : [java]
---

Recently I've been bitten by the clean architecture movement. It's just something that feels
right and lives side-by-side with my existing research and beliefs. So, to test my hypotheses, I
made an example application that tries to implement the purest form of clean architecture.

I already made several infrastructure component, which enables me to seamlessly integrate JPA, plain
JDBC or MongoDB without any impact on my domain or application layers. So far, so good. I also made
a Spring MVC REST infrastructure component to expose my application to the outside world and to actually
test my application on an integration level front to back. This worked flawlessly.

Then I made a small feature addition: I added an optional query parameter to one of my endpoints. My application
layer (and me) really dislikes nullable parameters so I provided an overloaded method that only allows for non-nullable
parameters, so that I never have to send null and can actually check whether null is not being passed (avoiding NPEs).

The problem with this is that neither Spring MVC or Jersey have support for overloaded methods when it comes to endpoints.
Spring MVC sees parameters as required by default, Jersey sees them as non-required by default (and you need to add support
for bean validation annotations if you want it to be required, by the way, using @NotNull). But if you define a parameter
as optional, you need to have icky `if` statements checking for null to handle the non-null behavior of my application layer.
That's not cool.

When I first saw this, I wrote this piece of code, which would have actually been the elegant solution.

{% highlight java %}
@GetMapping
public List<BuildingJson> find(@RequestParam(value = "nameStartsWith") String name)  {
	return listBuildings.execute(new ListBuildingsRequest(name), new JsonBuildingResponseModelPresenter());
}

@GetMapping
public List<BuildingJson> find()  {
	return listBuildings.execute(new ListBuildingsRequest(), new JsonBuildingResponseModelPresenter());
}
{% endhighlight %}

And as you might have guessed, my integration tests all failed, the entire controller was marked invalid
because of this construct. You cannot overload methods in Spring MVC if they share the same endpoint, even
if they have different query variables (which are required). Shit.

Same thing with Jersey, the resource fails to register because of an overloaded method with the same REST endpoint.

So I had to write this.

{% highlight java %}
@GetMapping
public List<BuildingJson> find(@RequestParam(value = "nameStartsWith", required = false) String name)  {
	// YUCK.. I really don't want to pass nullable fields but this stinks...
	if(name == null) {
		return listBuildings.execute(new ListBuildingsRequest(), new JsonBuildingResponseModelPresenter());
	} else {
		return listBuildings.execute(new ListBuildingsRequest(name), new JsonBuildingResponseModelPresenter());
	}
}
{% endhighlight %}

I really, really dislike this. This just reads awful. Bear in mind, if I had multiple filters, I would employ a builder
pattern for the request (because of the combinations), but the `if` statements would remain. You'd get something like this.

{% highlight java %}
@GetMapping
public List<BuildingJson> find(@RequestParam(value = "nameStartsWith", required = false) String name)  {
  ListBuildingRequestBuilder requestBuilder = new ListBuildingRequestBuilder();
  if(name != null) {
    requestBuilder.setNameStartsWith(name);
  }
  // if ...
  // if ...
  return listBuildings.execute(requestBuilder.build(), new JsonBuildingResponseModelPresenter());
}
{% endhighlight %}

You get the point.

But what other options do I have? Well, I could use `java.util.Optional` in the parameter in my request. This would
reduce this to this.

{% highlight java %}
@GetMapping
public List<BuildingJson> find(@RequestParam(value = "nameStartsWith", required = false) String name)  {
	return listBuildings.execute(new ListBuildingsRequest(Optional.ofNullable(name)), new JsonBuildingResponseModelPresenter());
}
{% endhighlight %}

This actually reads a lot better, but unfortunately IntelliJ disagrees. See, if you try to use an Optional field as a method
parameter, it'll mark this as a warning. Turns out, this is for a reason. Brian Goetz, the guy that was actually part of the
team that designed the API, warned that this was not the intended usage of Optional and that you should be better off writing
overloaded methods. He probably doesn't write a lot of REST APIs.

Spring MVC has quite good support for Optional, because you can simplify the above code like this.

{% highlight java %}
@GetMapping
public List<BuildingJson> find(@RequestParam(value = "nameStartsWith") Optional<String> name)  {
	return listBuildings.execute(new ListBuildingsRequest(name), new JsonBuildingResponseModelPresenter());
}
{% endhighlight %}

That's clean code that actually communicates intent. But with a big, fat, ugly warning backed by the guys who invented the API in the first place. You can do the same thing in Jersey, but it's not supported out of the box, unfortunately, so need to add some magic classes to support `Optional`.

So I'm at a loss. Do I either add the `Optional` the signature of my application layer method, abandoning overloaded methods? Do I create various if statements
to handle null values and call the overloaded methods or build up the request object like that? Or Be more lenient towards null values in my application layer (I'd rather not)? I really don't know. To me, using `Optional` here seems like a good idea, but I tend to agree with most that using overloaded methods makes for more readable code. But if the REST framework doesn't allow you to write nice, clean code, what the least bad alternative? It seems to me like the boundaries of clean code in a hexagonal (or clean) architecture are tested in the infrastructure layer, because of the specific APIs being used there and the limitations those APIs bring to the scene.

Feel free to comment on my Twitter.
