---
layout: post
category : article
title: "A more OO-style approach to refactoring with Kotlin"
comments: true
tags : [technical]
---

Refactoring is one of the mainstays of most developers in order to get their code more maintainable and readable. It's part of the quintessential red-green-refactor cycle and an invaluable tool in order to remove and/or avoid technical debt into the system. 

However, refactoring sometimes tends to result in small procedural private methods in a class. Take the following example:

``` kotlin
@Named
@Transactional
class AddTableToFloorImpl(private val floorQueryRepository: FloorQueryRepository) : AddTableToFloor {
    override fun perform(request: AddTableToFloor.Request) : AddTableToFloor.Response {
        val floor = floorQueryRepository.getFloorById(request.floorId)
        val table = Table.createNew(request.name).save()
        floor.addTable(table)
        return AddTableToFloor.Response(table.id)
    }
}
```

This code does a bit too much and should be refactored. A indicator of the need to refactor is when you have multiple dots on the same line in a method that has more than a single line of code. So we end up with something like this after refactoring.

``` kotlin
@Named
@Transactional
class AddTableToFloorImpl(private val floorQueryRepository: FloorQueryRepository) : AddTableToFloor {
    override fun perform(request: AddTableToFloor.Request) : AddTableToFloor.Response {
        val floor = getFloor(request)
        val table = createNewTable(request)
        floor.addTable(table)
        return createResponse(table)
    }

    private fun createResponse(table: Table) = AddTableToFloor.Response(table.id)

    private fun createNewTable(request: AddTableToFloor.Request) =
            Table.createNew(request.name).save()

    private fun getFloor(request: AddTableToFloor.Request) =
            floorQueryRepository.getFloorById(request.floorId)
}
```

This code is already a lot more readable now, the intent of the implementation of the use case is easily understood. But we've introduced a very procedural way of refactoring. Granted, they're all private methods, but still, not that clean. 

Another indicator when you can refactor is when you have methods that have a single parameter. In many cases this means you should move that method to the parameter's class. However in this case, this is not possible. If you adhere to clean architecture principles, adding the methods on `Table` and `AddTableToFloor.Request` is not possible: `Table` (a domain class) has no knowledge of  `AddTableToFloor.Response`, and `AddTableToFloor.Request` (an application API use case class) has no knowledge of `Table` or `FloorQueryRepository`. If you where to move this code, you'd have to put a dependency on the application API in the domain, which would violate the outside-in dependency direction. So that's a no-go.

Luckily with Kotlin, you now have extension methods. These allow developers to add methods to classes inside a specific scope. So now we can refactor to this inside the application API implementation (which knows both the application API use cases and the domain):

``` kotlin
@Named
@Transactional
class AddTableToFloorImpl(private val floorQueryRepository: FloorQueryRepository) : AddTableToFloor {
    override fun perform(request: AddTableToFloor.Request) : AddTableToFloor.Response {
        val floor = request.getFloor()
        val table = request.createNewTable()
        floor.addTable(table)
        return table.createResponse()
    }

    private fun Table.createResponse() = AddTableToFloor.Response(id)

    private fun AddTableToFloor.Request.createNewTable() =
            Table.createNew(name).save()

    private fun AddTableToFloor.Request.getFloor() =
            floorQueryRepository.getFloorById(floorId)
}
```

By just moving a bit of code around, we now have a completely OO based refactoring. We've added 2 methods to `AddTableToFloor.Request` and a method to `Table` within the scope of the use case implementation. These instance methods on these classes don't exist outside of this implementation. 

To me, this feels like a much cleaner approach to refactoring, adding behavior on objects within a specific scope and using the dependencies which are only available within that scope. Kotlin to the OO rescue!