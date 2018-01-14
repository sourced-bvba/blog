---
layout: post
category : article
title: "Creating a Kotlin DSL for validation"
comments: true
tags : [technical]
---

As a fan of Clean Architecture, I try to stay as close to its principles as possible. For example, the domain and application layers should have as little dependencies as possible and defer their technical choices to the infrastructure layers. In some cases, this may prove to be a harder exercise than you might imagine, but not an impossible one. 

Take validation for example. We current have the Bean Validation 2.0 specification in Java at the moment, implemented by Hibernate Validator, the de facto choice for validation in most frameworks. However, this specification relies mainly on annotations which need to be put on the datastructures they validate. In the case of Clean Architecture, you'll probably want to validate the input of your use cases. But that would mean a new dependency, the API for the Bean Validation spec, for the application API layer. Meh.

If you want to hide implementation details from a part of your application you'll probably create an abstraction, so that's what I started to build, in order to decouple my application API validation from any technical choice. And since I love using Kotlin, I thought a DSL might be a fun approach. Basically, I wanted my application API to define a validation specification which could then be converted into a real validation implementation in one of the infrastructure layers. I wanted to end up with something like this

{% highlight kotlin %}
val spec = validationSpec {
    constraints<Foo> {
        field(Foo::bar) {
            notBlank()
            size(min = 5)
        }
    }
}
{% endhighlight %}

Here I'm basically creating a validation specification for the `Foo` class, defining that the `bar` field (a String) just be at least 5 characters long and not only contain whitespace characters.

I wanted a couple of other hard constraints (after yet again some very useful input from [@khofmans](http://www.twitter.com/khofmans)):
- No hardcoded field names for compile-time safety
- Type checking on the fields, the constraints for `Foo` should only contain fields of `Foo`
- Type checking on the rule: a `email()` constraint is not applicable to an numeric field, so you shouldn't be allowed to that constraint to such a field

If you know how Hibernate Validator works, you also know that you can already programmatically configure the validator. For example, you can do the example with the DSL like this:

{% highlight kotlin %}
val constraintMapping = DefaultConstraintMapping()
constraintMapping.type(Foo::class.java)
        .property("bar", ElementType.FIELD)
        .constraint(NotBlankDef())
        .constraint(SizeDef().min(5))
val config = Validation.byProvider(HibernateValidator::class.java).configure()
config.addMapping(constraintMapping)
val validator = config.buildValidatorFactory().validator
{% endhighlight %}

After this, you can use the `validator` to validate your data structures. However, there are a couple of issues here. For starters, the `name` property is hardcoded and you can't check whether the validated class actually has a `name` field. Secondly, there's nothing that prohibits me from defining a `EmailDef()` constraints on an numeric field with this API. I think we can do a better job.

First, I define the data structures that make up the validation specification:

{% highlight kotlin %}
data class ValidationSpec(val constraints: MutableList<Constraints<out Any>> = mutableListOf())

data class Constraints<T : Any>(val klass: KClass<T>, val fieldConstraints: MutableList<FieldConstraint<T, out Any>> = mutableListOf())

data class FieldConstraint<T : Any, P: Any>(val property: KProperty1<T, P>,
                                            val constraintRules: MutableList<ConstraintRule<in P>> = mutableListOf()) {}

interface ConstraintRule<T : Any>
{% endhighlight %}

A `ValidationSpec` holds one or more `Constraints` for the different types, which in their turn define the various `ConstraintRule`'s for the `FieldConstraint`'s. A `ConstraintRule` is type-bound, in order to fulfill one of my constraints. So to implement the constraints needed in the DSL example, we need to provide a couple of implementations of `ConstraintRule`.

{% highlight kotlin %}
data class StringSize(val min: Int, val max: Int) : ConstraintRule<String>
class StringNotBlank : ConstraintRule<String>
{% endhighlight %}

Now Building a DSL for a `ValidationSpec` is fairly easy:

{% highlight kotlin %}
fun validationSpec(block: ValidationSpec.() -> Unit) : ValidationSpec {
    val validationSpec = ValidationSpec()
    block(validationSpec)
    return validationSpec
}

inline fun <reified T : Any> ValidationSpec.constraints(block: Constraints<T>.() -> Unit) {
    val constraints = Constraints(T::class)
    this.constraints.add(constraints)
    block(constraints)
}

fun <T : Any, P : Any> Constraints<T>.field(property: KProperty1<T, P>, block: FieldConstraint<T, P>.() -> Unit) {
    val fieldConstraint = FieldConstraint(property)
    fieldConstraints.add(fieldConstraint)
    block(fieldConstraint)
}

fun FieldConstraint<out Any, String>.notBlank() {
    constraintRules.add(StringNotBlank())
}

fun FieldConstraint<out Any, String>.size(min: Int = 0, max: Int = Int.MAX_VALUE) {
    constraintRules.add(StringSize(min, max))
}
{% endhighlight %}

One of the fun parts of Kotlin is reified generics. This means that in some cases, you can get the type of a generic type parameter. For the `constraints`, this means we can do `T::class`, something that is impossible in Java. Otherwise we would have been forced to pass the type as a parameter, while now this can be inferred, making `constraints<Foo> {...}` possible, instead of resorting to `constraints(Foo::class) {...}` in the DSL.

So now that we have a implementation agnostic DSL that we can use in our application API, we need a translation mechanism that we can use in our infrastructure layer to translate the specification to a real validation implementation.

{% highlight kotlin %}
class HibernateValidatorSpecFactory(val spec: ValidationSpec) {
    private val constraints: MutableMap<KClass<out ConstraintRule<out Any>>, ConstraintRuleTranslator<out ConstraintRule<out Any>>> = mutableMapOf()

    fun <C : ConstraintRule<out Any>> registerCustomConstraint(constraintRuleClass: KClass<out C>, translator: ConstraintRuleTranslator<out C>) {
        constraints.put(constraintRuleClass, translator)
    }

    class ConstraintRuleTranslator<C : ConstraintRule<out Any>>(private val block : (C) -> ConstraintDef<*, *>) {
        fun translate(c: C) : ConstraintDef<*, *> = block.invoke(c)
    }

    private fun toConstraintMapping(spec: ValidationSpec): ConstraintMapping {
        return with(spec) {
            val constraintMapping = DefaultConstraintMapping()
            constraints.forEach {
                val typeMapping = constraintMapping.type(it.klass.java)
                it.fieldConstraints.forEach {
                    val propertyMapping = typeMapping.property(it.property.name, ElementType.FIELD)
                    it.constraintRules.forEach { rule: ConstraintRule<out Any> ->
                        val c = propertyMapping.constraint(toConstraintDef(rule))
                    }
                }
            }
            constraintMapping
        }
    }

    private fun <R: ConstraintRule<out Any>> toConstraintDef(rule: R): ConstraintDef<*, *> {
        if(constraints.containsKey(rule::class)) {
            val translator = constraints[rule::class] as ConstraintRuleTranslator<R>
            return translator.translate(rule)
        } else {
            if (rule is StringNotBlank) {
                return ConstraintRuleTranslator<StringNotBlank>({ NotBlankDef() }).translate(rule)
            } else if (rule is StringSize) {
                return ConstraintRuleTranslator<StringSize>({ SizeDef().min(it.min).max(it.max) }).translate(rule)
            } else {
                throw IllegalStateException()
            }
        }
    }

    fun createValidator(): Validator {
        val constraintMapping = toConstraintMapping(spec)
        val config = Validation.byProvider(HibernateValidator::class.java).configure()
        config.addMapping(constraintMapping)
        val factory = config.buildValidatorFactory()
        return factory.validator
    }
}
{% endhighlight %}

The translator could use some serious refactoring love, With this translator, we can now define a specification, make a `Validator` out of it and eventually validate an object.

{% highlight kotlin %}
// simple data class
data class DslTest(val sField: String, val iField: Int)

// transform spec into hibernate validator and check a data class instance
fun main(args: Array<String>) {
    val spec = validationSpec {
        constraints<DslTest> {
            field(DslTest::sField) {
                notBlank()
            }
        }
    }
    val validator = HibernateValidatorSpecFactory(spec).createValidator()
    val dslTest = DslTest("", 3)
    val violations = validator.validate(dslTest)
    if(violations.isNotEmpty()) {
        throw ConstraintViolationException(violations)
    }
}
{% endhighlight %}

This mechanism is also extensible. For example, I can define new rules and extend the DSL in my own code.

{% highlight kotlin %}
data class IntegerMinimumValue(val value: Int) : ConstraintRule<Int>
fun FieldConstraint<out Any, Int>.min(value: Int) {
    constraintRules.add(IntegerMinimumValue(value))
}
{% endhighlight %}

Now say I want to validate using the following spec (`baz` is an Int field), which contains a `min` extensions to the rules.

{% highlight kotlin %}
val spec = validationSpec {
    constraints<Foo> {
        field(Foo::bar) {
            notBlank()
            size(min = 5)
        }
        field(Foo::baz) {
            min(3)
        }
    }
}
{% endhighlight %}

In order to use the extension, you'll have to register the new `ConstraintRule` in the translator.

{% highlight kotlin %}
val hibernateValidatorSpecFactory = HibernateValidatorSpecFactory(spec)
hibernateValidatorSpecFactory.registerCustomConstraint(
        IntegerMinimumValue::class, HibernateValidatorSpecFactory.ConstraintRuleTranslator({ MinDef().value(it.value)})
)
{% endhighlight %}

Here I'm using a built-in `ConstraintDef` from Hibernate Validator, but you can build your own if you want which means creating a new validation annotation, a validator for that annotation and a new `ConstraintDef` implementation.

While this was an interesting experiment, the code here is nowhere near production ready. But it clearly shows the viability of the underlying ideas. Writing Kotlin DSLs to create datastructures is incredibly powerful and allowed for strongly typed data structures and the enforcement of rules regarding allowed types. If I have the time, I'll post a working example on Github, providing support for most of the constraints in the standard Bean Validator 2.0 spec.




