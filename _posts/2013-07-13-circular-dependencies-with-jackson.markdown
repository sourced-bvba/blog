---
layout: post
title: "Circular dependencies with Jackson"
date: 2013-07-13 14:14
comments: true
categories: [java]
---

Circular dependencies and JSON have always been a pain. But it's not just JSON, the problem also exists when you're trying to serialize a graph which contains circular dependencies (parent/child with bidirectional relationships).

Some time ago, we were considering exposing our JPA datamodel through REST services. Off course, a lot of JPA model classes contain bidirectional relationships, which was a real pain to get working. We ended up with a separate data model consisting of DTO's (yuck!) and a mapping between the two models. But after a while we had to abandon our REST quest due to the fact that the JPA data model was getting to complicated. So we let go of the loose coupling between the client and the server, which made the issue go away completely. REST services where built when the need for external communication arose, but for client-server communication a more direct dependency was used (CDI/EJB or Spring injection).

<!--more-->

Recently, I once again looked at Jackson. My reasons now where a bit different. Our data model has grown to a point where finding out what exactly is in a graph is getting problematic. A simple SQL query doesn't cut it anymore and we're forced to start debugging in order to see what an object actually contains. Knowing in advance how a complex JPA datamodel is populated through a JQPL query is a science on its own. 

So I thought, why not have the possibility to send the same JPQL query and have the result returned to us as JSON. The problem, I thought, would be those wretched circular dependencies. Luckily, the Jackson developers have since developed a solution to the problem: their JSON serializer now supports object references. And it's usable out-of-the-box for JPA datamodels.

Their JSON object reference requires an object to have a unique ID. Luckily, this is also the case for JPA entities. However, JSON id references need to be unique across the entire graph, whereas JPA id's only need to be unique within the same entity. In our case, it wasn't really an issue, as we use UUID's for JPA id fields, which are unique throughout the entire database.

So how do you serialize an object graph? Well, assume you have two entities with bidirectional relationships like this:

``` java
@Entity
public class ParentEntity {
    @Id
    private String id;
    private String description;
    @OneToMany(mappedBy = "parent")
    private List<ChildEntity> children;

     // getters and setters omitted for brevity
}

@Entity
public class ChildEntity {
    @Id
    private String id;
    private String description;
    @ManyToOne
    private Parent parent;

    // getters and setters omitted for brevity
}
```

Adding Jackson JSON identities is very simple:

``` java
@Entity
@JsonIdentityInfo(generator=ObjectIdGenerators.PropertyGenerator.class, property="id")
public class ParentEntity {
    ...
}

@Entity
@JsonIdentityInfo(generator=ObjectIdGenerators.PropertyGenerator.class, property="id")
public class ChildEntity {
    ...
}
```

And that's it! If you would now serialize a parent object with 2 children, you'll get something like this:

``` json
{
    "id": "parent-id1",
    "description": "parent",
    "children": [
        {
            "id": "child-id1",
            "description": "child1",
            "parent": "parent-id1"
        },
        {
            "id": "child-id2",
            "description": "child2",
            "parent": "parent-id1"
        }
    ]
}
```

