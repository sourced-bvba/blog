---
layout: post
category : article
title: "Spring Data Projections"
comments: true
tags : [technical]
---

Recently I've learned a nice feature in Spring Data: the ability to make projections based on interfaces.

Say you have a JPA entity like this:

{% highlight kotlin %}
@Entity data class Dish(@Id val id: String, val name: String, val price: BigDecimal, @OneToMany val ingredients : List<Ingredient>)
{% endhighlight %}

With Spring Data you can easily write a repository that can handle the CRUD operations for a `Dish`. However, sometimes you want to change
the format you get from the database. For example, you want a projection where you only get the name and the price from the database, strongly typed.
With Spring Data, you can do this. You just need to make an interface like this:

 {% highlight kotlin %}
interface DishWithPrice {
    @Value("{target.name + ' (' + target.price + ')'}"))
    fun getDishWithPriceName();
}    
{% endhighlight %}

Getting the projection is as simple as adding a method to the repository.

 {% highlight kotlin %}
interface DishRepository : JpaRepository<String, Dish> {
    fun findAllProjectedByDishWithPrice() : List<DishWithPrice>
}    
{% endhighlight %}

You don't need to make an implementation of that projection interface, Spring Data makes one on the fly. 

Additionally, Spring Data can handle fetching data intelligently in some cases, optimizing the amount of data that is retrieved from the database. In the first projection, the entitymanager will get the all the properties of DishPrice object, including the `id` field, even though it's not used in the projection. This is because the `DishWithPrice` is an "open projection". Spring Data also has a concept of a "closed projection", which means if a projection uses only the exact same getters as the entity, Spring Data will only fetch those fields. For example, if our interface looked like this:

 {% highlight kotlin %}
interface DishWithPrice {
    fun getName() : String
    fun getPrice() : BigDecimal
}    
{% endhighlight %}

Then Spring Data would only fetch the `name` and `price` fields from the database.

There are however a couple of caveats here. The first is that if you include a field in your projection that returns a `Collection` or a `Map` (or one of its subclasses), the projection is considered open, even though the getter is named the same, in our case `getIngredients`. 

The second is that you need to be careful not to add methods to the projection that are not accessors or getters on the original entity. 

{% highlight kotlin %}
interface DishWithPrice {
    fun getName() : String
    fun getPrice() : BigDecimal
    fun getDishWithPriceName() = getName() + " (" + getPrice() + ")"
}    
{% endhighlight %}

This will fail, because there is no `getDishWithPriceName` on `Dish`. Even more, every method that is not an accessor of the original entity will fail. Luckily, you can easily work around this limitation with Kotlin extension methods:

{% highlight kotlin %}
interface DishWithPrice {
    fun getName() : String
    fun getPrice() : BigDecimal
}    

fun DishWithPrice.getDishWithPriceName() = getName() + " (" + getPrice() + ")"
{% endhighlight %}

As the extension method is resolved statically, Spring Data doesn't have any issues with it and in this case, it's a closed projection, so you'll get the added benefit of optimized data fetching. The last example is by the way the closed projection alternative to my first example. With Java, this would be a tad bit harder to pull off, but possible nonetheless.

Spring Data projections are great, but keep the limitations of the system in mind. If you have large aggregates being handled by Spring Data that contain collections, it may not be the easiest solution to solve this problem. Using custom JPQL calling a constructor might be a better bet here, or doing the aggregation of the projection yourself using multiple calls (which may be necessary anyway to avoid a cartesian product problem).

Always check your SQL queries that are executed against a database. If you're fetching all the data from the database anyway, you could might as well avoid Spring Data projections alltogether and use decoupled datastructures to hide the data from the consumer.


