---
layout: post
category : article
title: "A couple of thoughts on Clean Architecture"
comments: true
tags : [opinion]
---

Those who have been coding long enough, know how important good design is in a software system. It's something that isn't talked about a lot in conferences because it's just not that sexy as the new umpteenth version of framework X. But to me, it's infinitely more important because without a good design and architecture, no framework will save you from writing unmaintainable code.

In 2012 Robert C. Martin wrote an article on Clean Architecture. He pitched the term in order to group together a couple of ideas like Screaming Architecture and Hexagonal Architecture. Most of these concepts weren't new, some of these were already written about in the early 1990's. But one wonders why, oh why, we're still writing unmaintainable software well into the second decade of the 21st century. So Uncle Bob sort of made a mix of all the concepts and made a couple of simple graphics that represented a clean architectural design.

![Clean Architecture](/img/CleanArchitecture.jpg)

While this was quite conceptual, it shows the strength of the architecture and how it adheres to the basic principles of Dependency Inversion. All the dependencies are pointed inwards, while the flow of control remains as you would expect it. In later presentations on Clean Architecture, he added a more concrete representation of the design.

![Clean Architecture Design](/img/CleanArchitectureDesign.png)

In his presentation, he goes over each of the elements of the design and explains in length what their responsibilities are and why they actually have a reason to exist and I highly recommend searching YouTube for his talks on Clean Architecture.

So after looking at all the talks I looked for some concrete implementations of Clean Architecture, specifically in Java. I found a couple, but none really made it easy to translate between the paper design and the actual implementation. So I decided to [build my own](https://github.com/lievendoclo/cleanarch). It has gone through quite a few iterations, I started off doing it in Kotlin, because it's such a cool language, but eventually I turned back to Java because it translated my intent better instead of using language features. Through those iterations, I also found some parts of the original design that I really have my doubts about and that's what this article it actually about. It's only 2 things, but they deviate enough from the design to merit this article.

The first thing is the dependency from an Interactor (an implementation of a Boundary) with another Boundary. To me, this feels a bit wrong and seems like a violation against the Single Responsibility Principle. To me, a Boundary is a single business use-case, like GetBuilding or CreateBuilding. If I'd interpret the design literally, having the CreateBuildingImpl (the Interactor) call the GetBuilding use-case would be a valid interaction. Personally I think this should be the responsibility of the component calling the Boundary (or multiple) and do the orchestration there. In a REST based system, you'd probably avoid such a case altogether.

The second thing is that the Presenter is actually an implementation of a Boundary. As I said earlier, to me a Boundary is a use case. A Presenter is merely a component whose responsibility is to transform the ResponseModel of the Boundary into something else, for example a structure that is suitable for JSON serialization.

How I interpreted this was to introduce a ResponseModelConsumer. A Boundary has a dependency on a ResponseModelConsumer and passes any ResponseModel the Boundary produces to it. A standard JSON serialization Presenter is merely a stateful implementation of such a ResponseModelConsumer and can be queried for the presented datastructure after the Boundary has been called.

For example, say I have a simple Boundary like this:

{% highlight java %}
@FunctionalInterface
@Boundary
public interface GetBuilding {
  void execute(GetBuildingRequest request, Consumer<BuildingResponseModel> responseModelConsumer);
}
{% endhighlight %}

Then my Controller actually calls the Boundary and passes in a ResponseModelConsumer implementation, a Presenter.

{% highlight java %}
@GET
@Produces("application/json")
@Path("/{buildingId}")
public BuildingJson get(@PathParam(("buildingId")) String buildingId)  {
  final JsonBuildingResponseModelPresenter presenter = new JsonBuildingResponseModelPresenter();
  getBuilding.execute(new GetBuildingRequest(buildingId), presenter);
  return presenter.getPresentedResult();
}
{% endhighlight %}

This adheres to the Dependency Inversion principle and keeps Boundary implementations out of my infrastructure layers.

The only way how I'd be compliant with the original drawing and which actually takes away both of my concerns is if I see the ResponseModelConsumer as a Boundary. Then indeed a Boundary (whose responsibility is to consume ResponseModels) would have a dependency on another Boundary (which is actually a use-case) and the Presenter would be an implementation of a Boundary. But it just feels so strange to do so. An added benefit of the ResponseModelConsumer approach is that asynchronous handling becomes perfectly possible, without having that concern go beyond the infrastructure layer. One can perfectly create a reactive implementation or have it simply put the results on a queue.

Because of the abstract nature of the original design, it's hard to determine what the good approach is. I feel like I've gone as close as I possibly can to the original design while maintaining enough flexibility to defer certain choices to a later point in time. I guess the only person that's able to answer this is Uncle Bob, but I'd love to hear your responses and how you interpret the original design.
