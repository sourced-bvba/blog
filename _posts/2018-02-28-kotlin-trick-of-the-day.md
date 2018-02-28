---
layout: post
category : article
title: "Kotlin TOTD: externally immutable, internally mutable collections"
comments: true
tags : [technical]
---

Consider the following constraints:

You have a Kotlin class `MyClass` that has a` List<String>` property, called `items`. You want to :
- be able to initialize the List property with a `List`, i.e. use a `List<String>` as the constructor property type
- be able to change the collection internally, for example through instance methods (for example `addItem(item: String)`)
- be able to access the property, but returning once again a List, so `MyClass(...).items` should return a `List` 
- avoid using `var`
- do all this as transparently as possible for the consumer of your Kotlin class without introducing
  any new fields with a name different than items, so no underscores in your public API.

In short, make a class that makes this work:

{% highlight kotlin %}
fun main(args: Array<String>) {
    val myClass = MyClass(listOf("one"))
    myClass.addItem("two")
    println(myClass.items)
}
{% endhighlight %}

`MyClass` should accept a `List`, not a `MutableList`. `myClass.items` should also return a `List` (so that `add` is not possible). 

Try it and see what you come up with.

Found it?

Well, it may look easy at first, but Kotlin throws you a curve ball because of the difference between a List and a MutableList. The cleanest solution I could find to this problem is this:

{% highlight kotlin %}
class MyClass(items: List<String>) {
    private val _items : MutableList = items.toMutableList()
    val items: List<String> = _items

    fun addItem(item: String) {
        _items.add(item)
    }
}
{% endhighlight %}

If someone has a shorter, cleaner way of achieving this, I'm all ears.


