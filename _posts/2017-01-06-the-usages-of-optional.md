---
layout: post
category : article
title: "The truth about Optional"
comments: true
tags : [java]
---

In my previous article I highlighted a question I had regarding clean architecture and
clean code, and how to apply it to REST endpoints using 2 of the most prolific frameworks
currently being used in Java.

Since writing that article, I did some more digging. Not into the problem with REST but
into the reasoning behind the very limited allowed applications of `Optional`. I'll start off
with the only use-case `Optional` was intended for: as a return type.

This actually makes perfect sense. We've all learned with Clean Code that you should try to limit
returning `null`s. If your method returns a `Collection`, it's preferred to return an empty one instead
of `null`. Optional actually gives you the flexibility to do this for any object, keeping in mind that you
should never, ever return `null` for an Optional (and you should be chastised if you ever do this). However
this does not imply that your internal fields cannot use `null`. You should still use basic objects in your
private state because of a few reasons. One of them is performance. A `null` reference does not take up any
memory size on the heap, an `Optional` takes up 16 bytes, even if it points to no value. It doesn't sound like
a lot until you start using Optional everywhere for, well, optional, fields. The other reason is the fact that
`Optional` is not serializable. This has been done intentionally, because the creators of `Optional` wanted to
leave the room for making it a value object in a later version of Java.

In my article, I used `Optional` as a parameter. While this might seem like a good idea at first, this quickly
turns ugly when you have multiple parameters like that in your interface. Suddenly, you have an unreadable method
invocation like this.

{% highlight java %}
service.createProduct(name, Optional.empty(), Optional.empty(), price);
{% endhighlight %}

Most of the time, if you're using `Optional`, you should either us overloaded methods or use a parameter objects.
Both options allow you to specify which combination is valid, either through its signature or through validation
of the parameter object. Off course you'll have to implement the latter yourself.

Using a parameter object also is the solution to the problem of the last article. Both Spring MVC and Jersey can handle
complex objects to handle query parameters: Spring MVC will handle any property in a non-annotated parameter as a query
parameter, Jersey needs a `@BeanParam` annotation. With Spring MVC for example the REST endpoint would now become this.

{% highlight java %}
@GetMapping
public List<BuildingJson> find(ListBuildingsRequestParams params)  {
	return listBuildings.execute(new ListBuildingsRequest(name), new JsonBuildingResponseModelPresenter());
}
{% endhighlight %}

The params object itself looks like this.

{% highlight java %}
public class ListBuildingsRequestParams {
	private String nameStartsWith;

	private Optional<String> getNameStartsWith() {
		return Optional.ofNullable(nameStartsWith);
	}

	public void setNameStartsWith(String nameStartsWith) {
		this.nameStartsWith = nameStartsWith;
	}

	public ListBuildingsRequest toRequest() {
		final ListBuildingsRequest.Builder builder = new ListBuildingsRequest.Builder();
		getNameStartsWith().ifPresent(builder::nameStartsWith);
		return builder.build();
	}
}
{% endhighlight %}

As you can see, `Optional` is only used as a return value to wrap a nullable reference. This way in the `toRequest` method
you can use `ifPresent` to correctly build the request. You do need the ugly getter/setter combination, but that's Spring
MVC not being able to handle it otherwise. As if Spring MVC knows you might abuse this: `Optional` fields aren't supported
nicely out of the box, if you omit a query parameter, the value of the field will be `null`, not `Optional.empty()`. So
don't do it.

While you did add another class, it's actually more in line with clean code standards, especially if you have multiple
query parameters. Even though you'll never call those methods directly, the standard of a maximum amount of parameters
still applies and adding parameter objects is a good reflex anyways.

This comes to show that you always need to do double-loop learning when looking at a problem. In my case, I focussed too
much on the usage of `Optional` and was trying to use it in place where it shouldn't have been used in the first place.
It also reinforces my belief that you need to adhere to clean code standards whenever you can. If I had done so, I'd probably
have introduced a parameter object from the beginning.

So my final thought is: if you're using `Optional` for anything else than a return type: stop. You don't need it.
