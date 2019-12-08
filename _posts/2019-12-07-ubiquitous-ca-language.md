---
layout: post
category : article
title: "Why we probably need a ubiquitous language for Clean Architecture"
comments: true
tags : [architecture]
---

Some time ago, I was asked by a friend to review a set of blog posts he wrote where he explained his experience with Clean Architecture and how he wanted to implement it using the technology he knew, .NET. While there were items where I didn't agree with his approach, reading his article made me realize that we need a better way to communicate when it comes to architecture.

This lack of communication is also something that Simon Brown, a well-known figure in the software architecture community, is saying: we need once again a way to communicate using somewhat standardized formats. His approach is the C4 methods, and while I do think that it has tremendous merit, it doesn't even have to be so complicated to be effective in smaller teams. I am not advocating the revival of the complete UML spec (which was the opposite movement of the pendulum), but we do need to start thinking about a simplified version.

If you look at the C4 model, for example, you see blocks and relationships between them. The relationships are mainly limited to a 'implements' and a 'uses' relation. The blocks remain relatively abstract, becoming more concrete once you drill down into the 2nd to 3rd level of the C4 model (the 4th level, in my opinion, is complete overkill, and if you start doing this manually, you should contemplate your life's choices). In that regard, we could simplify a couple of UML concepts.

But before we do that, we first need to agree on a language. There is nothing as frustrating as when people are talking about a repository, and you have no clue where you need to be in the architectural layers to find that thing. Some people talk about repositories when they talk about query services, others actually mean components that are way deeper in the layered model, almost the final step before accessing the database. In order to effectively communicate, we need a concept used in DDD: Ubiquitous Language. A set of language elements that are unambiguous in their meaning. Let's have a look at the drawing used by Robert C. Martin to depict the elements used in Clean Architecture:

![Clean Architecture Design](/img/CleanArchitectureDesign.png)

Starting from this diagram, where one could argue it is a very simplified UML diagram, we can distinguish some elements which we can extract to a language concept:

- **Entity**: A domain (aggregate) entity, containing the state and business behavior associated with that entity.
- **Entity gateway**: An interface that defines a way to store and retrieve entities from an external system. It is, in effect, a gateway for entities flowing from one system to another, for example, from your application to the database.
- **Implementing infrastructure**: This implements the `entity gateway's with the specific technologies that move the entities to and from an external system, like a database API. It can use an model internally to facilitate the usage of the technology, but that choice is completely transparent and should not leak to the outside.
- **Boundary**: An interface defining a business interaction with the system. A boundary has input (a **Request Model**) and output (a **Response Model**). This is what is exposed to the components that will actually expose behavior to the outside world.
- **Interactor**: An implementation of a `boundary`. It uses `entity gateway's to get entities and call the business behavior on those entities. It provides business level coordination of entities to fulfill a single business use case as defined by the `boundary`.
- **Consumer infrastructure**: While depicted here as **Controller**, this is too specific as many people see controllers as web-oriented components, whereas this part of the architecture is about exposing your system to the outside world. They call the `boundary` components in your system.
- **Presenter**: As the `consumer infrastructure` of your system calls the `boundary` components, you need something to decouple the `response model` from the outside world. It could be that you need to do some specific conversion due to the technology being used (XML or JSON, for example). The presenter's task is to take the `response model` and transform it into a **View Model**.

So now, we have a language that we can use to identify elements in our architecture unambiguously. The beautiful part is that UML now gives us some simple tools to accompany the language to transform the diagram to something most people can understand:

- We use full line arrows for `uses` and dotted arrows for `implements` relationships
- We use the UL terms as stereotypes in the schema

That's it. If we would take a basic interaction, say, getting a customer from a system, we could now easily utilize the simplified UML and the UL we agreed on to result in a diagram like this:

![Clean Architecture Example](/img/ca-example-uml.png)

Once you have this, you can have a lot of meaningful discussions around the code, like "Hmmm... I don't like the fact you have technology-specific code in the `GetCustomerImpl`, can't we defer that to the infrastructure layer?" or "Why is there JSON specific formatting in the `GetCustomerResponse`, that should be in the model that's delivered by the `JsonCustomerPresenter`, right?". I used colors to strengthen my point, where orange means infrastructure and, therefor, places where you can freely use technology-specific libraries or frameworks.

I'm not saying using this specific language would be the end-all for all discussions, but it does present a starting point, I think, to finally formalize a couple of critical components and concepts in software architecture. Next time someone starts about a `Service,` you should at least be able to link that component to one of the UL terms to determine its place in the architecture. And I'm sure that we'll probably be able to identify cross-functional components more early and violate a couple of rules (i.e., components that act as an interactor or boundary but also seem to have access to implementing infrastructure).

In the end, I realize that this is already reasonably detailed once you start going into these kinds of diagrams, which I think are the 3rd level in a C4 diagram. But I genuinely believe standardizing on something as crucial as technical architecture concepts makes sure at least that part is one that people don't need to spend time on getting it aligned within the team. Also, standardizing the technical term would make it easier to link those UL elements to language the business understands. And *anything* that helps us bring business and technology closer together seems like a good target.