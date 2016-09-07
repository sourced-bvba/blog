---
layout: post
title: "When reading Excel with POI, beware of floating points"
comments: true
categories: [java]
---

Recently, at my current project, we've been hunting down a bug in our software. See, our source data comes from Excel files and need to be imported into our system. Since it's a Java project, we're using POI.

Our problem began when we tried to read a certain cell that contained the value 929 as a numeric field and store it into an integer. So we did something like this.

{% highlight java %}
int value = new Double(cell.getNumericCellValue()).intValue();
{% endhighlight %} 

POI only has one way to read numeric cell values, and that's by using `double`. Guess what we got after reading the cell? 928. The reason? The numeric cell value, that shows in Excel as 929, is actually read as 928.99999999... You have to love floating points. You're probably wondering why we just didn't read the value as a `String` value and used `Integer.parseInt(...)`. Well, POI doesn't allow reading a cell as a `String` value if it's actually a numeric cell type.

The solution? 

{% highlight java %}
int value = new BigDecimal(d).setScale(0, RoundingMode.HALF_UP).intValue()
{% endhighlight %}  

The sad part is that this only happens for certain numeric values, if you're unlucky, the first code example will work (until it suddenly doesn't anymore). Again, that's floating point logic. This, by the way, is why I hate floating point and why I think they shouldn't be allowed in modern development. It's the principle of least surprise. Groovy, for examples, understands this and considers all decimal values in code as `BigDecimal` (and allows for easy math operations on those fields, unlike Java). You need to explicitly state your want to use a double.

So remember, when reading Excel numeric values, what you see isn't always what you get. That, and always use `BigDecimal`, even if the API forces you to use floating point fields. There's really no excuse for using `double` or `float`.
