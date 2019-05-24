---
layout: post
category : article
title: "Checking your clean architecture"
comments: true
tags : [java, kotlin, architecture]
---

A new post was long overdue, but I've been too busy working with amazing people at Atomist, making a product that changes how companies look at software delivery. But that's not what this article will be about. I'm going back to one of the topics that has dominated 2018 for me: Clean Architecture.

Recently I looked at a presentation that was given at Spring I/O by Tom Hombergs, which I think was a great presentation on how to implement a clean architecture using Spring. He also has a book that accompanies his presentations (https://leanpub.com/get-your-hands-dirty-on-clean-architecture/), which is quite as good as well, even though I don't always agree with the patterns he's using. 

But one of the slides caught my attention, which was the one that showcased a testing mechanism:

```java
@Test
void validateRegistrationContextArchitecture() {
    HexagonalArchitecture
        .boundedContext("io.reflectoring.copyeditor.registration")
        .withDomain("domain")
        .withAdapters("adapter")
            .incoming("in.web")
            .outgoing("out.persistence")
            .and()
        .withApplicationLayer("application")
            .services("book")
            .services("invitation")
            .incomingPorts("port.in")
            .outgoingPorts("port.out")
            .and()
        .withConfiguration("configuration")
        .check(allClasses());
}
```

If you start searching for this, I hope you have more luck that I had, because I wasn't able to find the code that achieves this, but this encouraged me to create my own which is what this article is about. You see, when you're using a multi-module monorepo for applications or microservices that adheres to clean architecture principles, you can enforce a lot of the rules with the dependencies between the modules. The incoming adapter module (what I call consuming infrastructure) should only have a dependency on the application api module (and perhaps a shared vocabulary). That's it. However, if you have a single repository, people are able to access every part of your application, even though you wouldn't want them to make shortcuts.

Using ArchUnit, I can make rules that check certain invariants when you use a clean architecture. A modules should:

- only access classes from modules that it should be able to access when adhering to clean architecture principles
- only extend from classes from modules that it should be able to access when adhering to clean architecture principles
- only have annotations for modules that it should be able to access when adhering to clean architecture principles

In code, this looks like this:

```kotlin
fun getRestrictiveRules(sourcePackage: String, vararg allowedPackages: String): List<ArchRule> {
    val list = mutableListOf<ArchRule>()
    list.add(ArchRuleDefinition.classes()
            .that().resideInAPackage(sourcePackage)
            .should().onlyAccessClassesThat().resideInAnyPackage(*allowedPackages))
    list.add(ArchRuleDefinition.classes()
            .that().resideInAPackage(sourcePackage)
            .should().onlyDependOnClassesThat().resideInAnyPackage(*allowedPackages))
    list.add(ArchRuleDefinition.classes()
            .that().resideInAPackage(sourcePackage)
            .should(onlyBeAnnotatedWithClassesInPackage(*allowedPackages)))
    return list
}

fun onlyBeAnnotatedWithClassesInPackage(vararg allowedPackages: String): ArchCondition<JavaClass> {
    return OnlyAnnotatedWithClassesInPackage(*allowedPackages)
}

private class OnlyAnnotatedWithClassesInPackage internal constructor(private vararg val allowedPackages: String) : ArchCondition<JavaClass>("only be annotated with classes in packages [${allowedPackages.joinToString(", ")}]") {
    override fun check(item: JavaClass, events: ConditionEvents?) {
        val annotations = item.annotations
        annotations.forEach {
            val match = PackageMatchers.of(*allowedPackages).apply(it.rawType.`package`.name)
            if(!match) {
                events!!.violating.add(SimpleConditionEvent(item, false, "${item.name} has annotation that is not in allowed package: ${it.rawType.name}"))
            }
        }
        item.fields.forEach { field ->
            val fieldAnnotations = field.annotations
            fieldAnnotations.forEach {
                val match = PackageMatchers.of(*allowedPackages).apply(it.rawType.`package`.name)
                if(!match) {
                    events!!.violating.add(SimpleConditionEvent(item, false, "${item.name} has field (${field.name}) with annotation that is not in allowed package: ${it.rawType.name}"))
                }
            }
        }
        item.methods.forEach { method ->
            val fieldAnnotations = method.annotations
            fieldAnnotations.forEach {
                val match = PackageMatchers.of(*allowedPackages).apply(it.rawType.`package`.name)
                if(!match) {
                    events!!.violating.add(SimpleConditionEvent(item, false, "${item.name} has method (${method.name}) with annotation that is not in allowed package: ${it.rawType.name}"))
                }
            }
        }
    }
}
```

Next I created a DSL with Kotlin that describes a clean architecture:

```kotlin
val architecture = cleanArchitecture {
    boundedContext("be.sourcedbvba.restbucks.order") {
        application {
            boundary {
                subPackage = "api.."
            }
            interactor {
                subPackage = "impl.."
            }
        }
        domain {
            model {
                subPackage = "domain.model.."
            }
            services {
                subPackage = "domain.services.."
            }
        }
        infrastructure {
            consuming {
                subPackage = "infra.web.."
            }
            implementing {
                subPackage = "infra.persistence.."
            }
        }
        shared {
            vocabulary {
                subPackage = "shared.vocabulary.."
            }
        }
        mainPartition {
            subPackage = "main.."
        }
    }
}
```

Now I can create an extension function that returns the rules for the definition that I just created. 

```kotlin
fun CleanArchitectureDefinition.rules(): List<ArchRule> {
    val list = mutableListOf<ArchRule>()
    this.boundedContexts.forEach { bc ->
        applicationApiRules(bc).forEach { list.add(it) }
        applicationImplRules(bc).forEach { list.add(it) }
        domainRules(bc).forEach { list.add(it) }
        consumingInfraRules(bc).forEach { list.add(it) }
        implementingInfraRules(bc).forEach {list.add(it) }
        sharedVocabularyRules(bc).forEach { list.add(it) }
        mainPartitionRules(bc).forEach { list.add(it) }
    }
    return list
}
```

The consuming infrastructure rules for example look like this:

```kotlin
val list = mutableListOf<ArchRule>()
    definition.consumingInfrastructurePackages.forEach {
        val allowedPackages = arrayOf(
                *definition.consumingInfrastructurePackages,
                *definition.applicationBoundaryPackages,
                *definition.sharedVocabularyPackages)
        list.addAll(getRestrictiveRules(it, *allowedPackages))

    }
    return list.toTypedArray()
```

In other words, the packages that I defined as a part of the consuming infrastructure should only be able to access:
- all the packages in the consuming infrastructure (that should even be more restrictive if you have multiple consuming infrastructures)
- all the packages in the application boundary (the use case interfaces)
- all the packages in the shared vocabulary

However, I soon bumped into a limitation of such a restrictive policy, because off course my consuming infrastructure is using external libraries (Spring Webflux classes, Reactor types, ...) and this off course now fails my test. So I introduced the concept of whitelisting: every piece in the architecture is able to define a whitelist of packages that it allows in its piece of the puzzle, and also a global whitelist, because now also things like usage of `java.util` classes now breaks the test.

So now you get the definition like this:

```
val architecture = cleanArchitecture {
    boundedContext("be.sourcedbvba.restbucks.order") {
        whiteList = listOf(
                "java.lang..",
                "java.util..",
                "java.math..",
                "kotlin..",
                "org.jetbrains.annotations.."
        )

        ...
        infrastructure {
            consuming {
                subPackage = "infra.web.."
                whiteList = listOf(
                        "org.springframework.web..",
                        "org.springframework.http..",
                        "reactor.core.."
                )
            }
            ...
        }
    }
}
```

And the rules get updated like this:

```kotlin
val list = mutableListOf<ArchRule>()
    definition.consumingInfrastructurePackages.forEach {
        val allowedPackages = arrayOf(
                *definition.whiteListPackages,
                *definition.consumingInfrastructurePackages,
                *definition.consumingInfrastructureWhitelist,
                *definition.applicationBoundaryPackages,
                *definition.sharedVocabularyPackages)
        list.addAll(getRestrictiveRules(it, *allowedPackages))

    }
    return list.toTypedArray()
```

Running all the tests takes less than a second, but this now provides a safety net. If someone adds a dependency to a new layer and starts using its classes, this will break this test. Well, you might say, then you just add that package to the whitelist. True, but to me that would be a trigger for an ADR (Architectural Decision Record). No one should change the architecture validation test without an ADR.

While I know people can game the system by putting classes in specific packages, but even then it would become hard. If you want to put `@RestController` on a use case implementation, you won't be able to unless you put it in the consuming infrastructure packages. But then you suddenly wouldn't be able to access the domain services (because that package only has access to the application layer). I like tests like this, because they force people to adhere to architectural decisions, even if you're using a single module project. Not having to worry that someone is exposing domain objects directly to the outside world through a web interface is priceless.

The code with the complete example you can find here: https://github.com/lievendoclo/clean-restbucks/tree/restrictive/main-partition/src/test/kotlin/be/sourcedbvba/restbucks/order

Feel free to reach out and give comments.