---
layout: post
category : article
title: "Decoupling security in a clean architecture"
comments: true
tags : [technical]
---

Security is a cross-cutting concern in most applications, and there's a plethora of options for you to choose from. However, most options tend to be quite invasive from an clean architectural point of view. 

For example, if you're using Spring Security, you can use the `@Secured` annotations in order to add security restrictions to certain methods. That said, if you want to add security to your application layer, using those annotations may not always be desireable, because of the dependency that is created when using those annotations. Remember, technical framework dependencies in your domain and application layers are bad, mkay? Yes, even Spring.

Annotations like `@Secured` also pose another issue. They are to generic and do not talk about the language. What's more readable, `@Secured("INVOICE_VIEW")` or `@InvoiceViewRestriction`. Adding extra annotations to frameworks aren't always that straightforward. And what if we want to change from using Spring Security to something else, like Apache Shiro? Or if we want use an abstraction like pac4j?

So how would you solve this? Well, when you're using Spring in your main partition, aspects are the solution to this problem. I'm guessing there should be a Java EE alternative to this as well (probably using interceptors?), but as I'm not a Java EE expert, I leave it to others to find a solution there.

So we start off from the business perspective, by creating an annotation. You can choose to either put this into your application or your domain layer. I tend to put this into my domain layer, as the annotations are part of the domain language and used in the implementation of the application.

``` kotlin
annotation class InvoiceViewRestriction
```

Now that we have this annotation, we can use it in the implementation of the use cases. By the way, there is also another reason why I'm putting my annotation in my domain layer: it forces you to put the annotation on the use case implementation instead of the interface. Why? Well, I'm kind of breaking my own rules there: it has a purely technical reason. Method annotations are not inherited and therefor annotation on interfaces cannot be used to match AspectJ pointcuts. Remember I said sometimes you need to make compromises in clean architecture? Well, this for me is one of those compromises. I can work around them by not using AspectJ, but it would just take too long. So I annotate my use case implementation:

``` kotlin
class ViewInvoiceImpl : ViewInvoice {
    @InvoiceViewRestriction
    fun viewInvoice(request: Request) : Response {
        ...
    }
}
```

Now that we have the annotation and its usage in place, we can create the aspect that handles the enforcement. This is something you do in an infrastructure level and where you make the technical framework choices.

``` kotlin
@Aspect
class InvoiceViewRestrictionAspect {
    @Around("@annotation(InvoiceViewRestriction)")
    fun handle(pjp: ProceedingJoinPoint) : Any {
        if(hasInvoiceViewingRights(getCurrentUser()) {
            return pjp.proceed()
        } else {
            throw AccessDeniedException("Access is denied")
        }
    }

    ... // implementation hasInvoiceViewingRights
}
```

The checking of the rights can then be framework specific. For example, with Spring Security, you'll check the current user in the `SecurityContextHolder` and check whether he has a certain authority. The contents of the `SecurityContextHolder` would be populated by an infrastructure layer on the other side (web), for example through the standard servlet filters offered by Spring Security.

But you can also choose to create a very naive, bespoke implementation at first to prove a concept before committing to a certain framework. From a business standpoint, nothing changes, you make the decision in the infrastructure layers. The application and the domain layers tell you what you want to do, how you technically do it is defined in the infrastructure layers. And you can test against behavior: given there is a user without permission X, if I call the use case, I get an access denied message. 

Without aspects, you can achieve the same thing using domain services. You define a domain service interface like this:

``` kotlin
interface AccessManager {
    fun <R> withPermission(permission: Permission, block: () -> R) : R {
        if(currentUserHasAccessTo(permission)) {
            return block.invoke()
        } else {
            throw AccessDeniedDomainException("Access is denied")
        }
    }

    fun currentUserHasAccessTo(permission: Permission) : Boolean
}
```

And you can have a couple of permissions in an enum

``` kotlin
enum class Permission {
    VIEW_INVOICE
}
```

Instead of using a domain annotation in your use cases, you'll now use a domain service. 

``` kotlin
class ViewInvoiceImpl(accessManager: AccessManager) : ViewInvoice {
    fun viewInvoice(request: Request) : Response {
        accessManager.withPermission(Permission.VIEW_INVOICE) {
            ...
        }
    }
}
```

In the infrastructure layer, now instead of implementing an aspect to handle the annotation, you implement the domain service:

``` kotlin
class AccessManagerImpl : AccessManager {
    fun currentUserHasAccessTo(permission: Permission) : Boolean {
        ...
    }
}
```

Here you'll use the same concepts as you would implementing the AspectJ advice, checking whether the current user has a certain permission. The `AccessManager` here is quite generic, using an enum, but you could make it as domain-specific as you want, i.e. `accessManager.withViewInvoicePermission { ... }`, the choice is up to you. You can also choose to expand this concept to add ACL-like properties to your access manager by create an abstraction for an ACL secured resource and passing that abstraction to the access manager to check whether a certain user has access to a specific domain object instance. 

When you're trying to keep your architecture as clean as possible, you'll striving towards putting all technological dependencies (and thus framework
decision) to the outside of your system, i.e. your infrastructure layers. In many cases, this means writing abstractions in your domain and application layers to describe what you want from a behavioral standpoint and leaving it up to the infrastructure layers to decide on how that behavior should be implemented. In addition, by creating those abstractions, you can focus on your security requirements from a business perspective instead of being constrained by the technical limitations of a certain framework. In other words, you're decoupled from the technical details.

With this approach you can declaratively describe what you want with regards to security and delay the implementation to a later point in time, for example by first implementing a naive, permissive security. You can then later, at any time, choose to replace that implementation with a more complete version using whatever framework you want, without having to worry about your business code being affected by that change. 

*Edit*: Thanks to the input from Darijan Jankovic, the non-annotation example has been optimized to use more a more functional style using higher order functions. Thanks Darijan!
