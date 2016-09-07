---
layout: post
category : article
title: "JDBI, a nice Spring JDBC alternative"
comments: true
tags : [java]
---

Recently I was researching some more Java EE-like alternatives for Spring Boot, such as WildFly Swarm and Dropwizard. And while looking at Dropwizard, I noticed they were using a library for JDBC access I hadn't encountered before: JDBI. Normally, my first response to plain JDBC access is using the JdbcTemplate class Spring provides, but lately I've been having some small issues with it (for example it not being able to handle getting the generated keys for a batch insert in an easy way). I'm always interested in giving other solutions a try so I started a small PoC project with JDBI. And I was pleasantly suprised. <!--more-->

JDBI is an abstraction layer on top of JDBC much like JdbcTemplate. And it shares most if not all the functionality JdbcTemplate provides. What's interesting is what it provides in addition to that. I'll talk about a couple of them.

## Built in support for both named and indexed parameters in SQL

Most of you know you have `JdbcTemplate` and `NamedParameterJdbcTemplate`. The former supports indexed parameters (using `?`) while the latter supports named parameters (using `:paramName`). JDBI actually has built in support for both and does not need different implementations for both mechanisms. JDBI uses the concept of parameter binding while executing a query and you can bind either to an index or to a name. This makes the API very easy to learn.

## Fluent API functionality

JDBI has a very fluent API. For example, take this simple query with JdbcTemplate:

{% highlight java %}
Map<String, Object> params = new HashMap<>();
params.put("param", "bar");
return jdbcTemplate.queryForObject("SELECT bar FROM foo WHERE bar = :param", params, String.class);
{% endhighlight %}

With JDBI, this would result in this (jdbi is an instance of the JDBI class):

{% highlight java %}
Handle handle = jdbi.open();
String result =  handle
    .createQuery("SELECT bar FROM foo WHERE bar = :param")
    .bind("param", "bar")
    .first(String.class);
handle.close();
return result;
{% endhighlight %}

To be completely correct, these two methods aren't really functionally equivalent. The JdbcTemplate version will throw an exception if no result or multiple results are returned whereas the JBDI version will either return null or the first item in the same situation. So to be functionally equivalent, you'd have to add some logic if you want the same behavior, but you get the idea.

Also be aware you need to close your `Handle` instance after you're finished with it. If you don't want this, you'll need to use the callback behavior. This can be accomplished quite cleanly when using Java 8 thanks to closures:

{% highlight java %}
return dbi.withHandle(handle ->
                          handle.createQuery("SELECT bar FROM foo WHERE bar = :param")
                              .bind("param", "bar")
                              .first(String.class)
);
{% endhighlight %}

## Custom parameter binding

We've all seen this in JdbcTemplate:

{% highlight java %}
DateTime now = DateTime.now();
Map<String, Object> params = new HashMap<>();
params.put("param", new Timestamp(now.getDate().getTime()));
return jdbcTemplate.queryForObject("SELECT bar FROM foo WHERE bar = :param", params, String.class);
{% endhighlight %}

The parameter types are quite limited to those supported by default in plain JDBC, which means simple types, String and java.sql types. With JDBI, you can bind custom argument classes by implementing a custom `Argument` class. In the case above, this would look like this:

{% highlight java %}
public class LocalDateTimeArgument implements Argument {

    private final LocalDateTime localDateTime;

    public LocalDateTimeArgument(LocalDateTime localDateTime) {
        this.localDateTime = localDateTime;
    }

    @Override
    public void apply(int position, PreparedStatement statement, StatementContext ctx) throws SQLException {
        statement.setTimestamp(position, new Timestamp(localDateTime.toEpochSecond(ZoneOffset.UTC)));
    }
}
{% endhighlight %}

With JBDI you can then do this:

{% highlight java %}
Handle handle = jdbi.open();
DateTime now = DateTime.now();
return handle
    .createQuery("SELECT bar FROM foo WHERE bar = :param")
    .bind("param", new LocalDateTimeArgument(now))
    .first(String.class);
handle.close();
{% endhighlight %}

However, you can also register an `ArgumentFactory` that builds the needed Argument classes when needed, which enables you to just bind a `LocalDateTime` value directly.

## Custom DAOs

Those of us that are lucky enough to use Spring Data know that it supports a very nice feature: repositories. However, if you're using JDBC, you shit out of luck, because this feature doesn't work with plain JDBC.

JDBI however has a very similar feature. You can write interfaces and annotate the method just like Spring Data does. For example, you can create an interface like this:

{% highlight java %}
public interface FooRepository {
    @SqlQuery("SELECT bar FROM foo where bar = :param")
    public String getBar(@Bind("param") String bar);
}
{% endhighlight %}

You can then create a concrete instance of that interface with JDBI and use the methods as if they were implemented.

{% highlight java %}
FooRepository repo = jdbi.onDemand(FooRepository.class);
repo.getBar("bar");
{% endhighlight %}

When creating an instance `onDemand` you don't need to close the repository instance after you're done with it. This also means you can reuse it as a Spring Bean!

## Other things and conclusion

There is a myriad of features I haven't touched yet, such as object binding, easy batching, mixins and externalized named queries. These features combined with those I mentioned earlier make JDBI a compelling alternative to JdbcTemplate. Due to the fact it works nicely in combination with Spring's transaction system, there's little effort needed to start using it. Most objects in JDBI are threadsafe and reusable as singletons so they can be defined as Spring beans and injected where needed.

If you're looking into doing plain JDBC work and need something different than JDBC template, have a look at JDBI. It's definitely going in my toolbox.
