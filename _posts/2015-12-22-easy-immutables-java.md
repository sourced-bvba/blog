---
layout: post
category : article
title: "Easy immutable objects with Java"
comments: true
tags : [java]
---

The concept of immutability in Java does not need any introduction. Anyone who has written highly multi-threaded systems has probably worked with immutable objects. Immutability greatly simplifies code and avoids encountering a whole range of issues. You’re probably already using a lot of immutable classes without realizing it, like String or the date classes in JDK 8 and JodaTime. However, implementing immutability within application domains has always been something left to the developers of that domain. Using things like copy constructors, final fields and defensive copying it was the developer’s responsibility to make sure his object was immutable and wasn’t exposing any mutable state.

That was until the advent of the [Immutables](http://immutables.org) library.

To start using the library, you just need to add the dependency to your project:

``` groovy
provided 'org.immutables:value:2.1.2'
```

``` xml
<dependency>
  <groupId>org.immutables</group>
  <artifactId>value</artifactId>
  <version>2.1.2</version>
  <scope>provided</scope>
</dependency>
```

The library uses annotations and the APT mechanism in Java to generate code based on simple interfaces or abstract classes defining how an immutable object should look like. A simple example of this would be:

``` java
@Value.Immutable
public interface Person {
  String getFirstName();
  String getLastName();
}
```

You can also opt to use abstract classes.

``` java
@Value.Immutable
public abstract class Person {
  public abstract String getFirstName();
  public abstract String getLastName();
}
```

In both cases, when compiling, the APT mechanism will automatically generate an immutable version of this interface and a builder to create immutable objects.

``` java
Person person = ImmutablePerson.builder().firstName("John").lastName("Doe").build();
```

That’s all what is needed to create immutable versions of your domain. If you want a mutable version, you can opt to create your own class, implementing the interface yourself. However, the library, despite its name, also offers a way to create mutable versions of the interface in the exact same way you create immutable versions.

``` java
@Value.Modifiable
public interface Person {
  String getFirstName();
  String getLastName();
}
```

This will create a ModifiablePerson which behaves the same way as a regular POJO, but for which a builder will also be automatically created. Use cases for this are when you’re using frameworks which cannot handle immutable objects well, like Hibernate. This will enable you to quickly switch between mutable and immutable versions of the same domain and limit the scope in which the mutable versions is exposed (preferable only in the repository layer).

One of the more interesting features of the library is how optional attributes are handled. By default, every field is mandatory unless you annotate it indicating that it is optional. The builder will throw an exception if you’re trying to build an immutable object with missing mandatory fields. However, you can also choose to use the Optional classes in Guava or JDK 8. This makes optionality an explicit part of your domain. For example, if you wanted to make lastName optional, you can do it like this:

``` java
@Value.Immutable
public interface Person {
  String getFirstName();
  Optional<String> getLastName();
}
```

This will have no impact on how the builder is created, it will transparently allow you to use the field as a String (and as an Optional as well).

The final juicy bit is how it handles JSON out of the box. The library has support for Jackson and GSON and allows you to use immutable objects in your REST APIs. In the case of Jackson, this is done by merely adding a module to the ObjectMapper.

Introducing immutability in your domain has never been easier. I encourage you to start looking at this library and see whether your domain will benefit from immutability.
