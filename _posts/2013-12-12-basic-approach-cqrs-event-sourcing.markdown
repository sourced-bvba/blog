---
layout: post
title: "A basic approach to implementing CQRS with event-sourcing"
date: 2013-12-12 22:00
comments: true
categories:
---

I've become a huge fan of CQRS-based codebases due to the fact that such codebases tend to scale really well and are more readable due to the clear boundaries and responsabilities of the components.

When you have a system that's quite heavy on concurrency (a lot of concurrent system consumers), CQRS will only get you so far. A lot of times such issues can only be solved by either throwing more hardware at the problem or rethinking the way your application handles data. I'll try to explain to you how you can use an event-based architecture to handle concurrent loads. Mind you, this is a very basic and naive implementation, but it should handle the most common concepts. The code is done in Groovy and Spring 3. It uses Guava's EventBus to simulate a message bus, one for commands and one for events.<!--more-->

Thinking in commands and events is actually very natural. Commands tell you what a system is capable of doing, events tell the you what the system has done.

The example handles 2 commands: saving documents and deleting documents. The queryservice allows you to search for all documents and get a single document by name. We're using TDD, so I wrote a spock test first that validates the behavior I want.

``` groovy
@ContextConfiguration(classes = TestConfiguration)
class CqrsDemo extends Specification {

    @Autowired
    private CommandService commandService;

    @Autowired
    private QueryService queryService;

    def "this should work"() {
        when:
        commandService.executeCommand(new Command("saveDocument", [documentName: "Hello.pdf", documentData: null]))
        then:
        queryService.getDocument("Hello.pdf") != null
        queryService.getDocument("Hello.pdf").data == null
        queryService.allDocuments.size() == 1

        when:
        commandService.executeCommand(new Command("saveDocument", [documentName: "Hello.pdf", documentData: "Hello".bytes]))
        then:
        queryService.getDocument("Hello.pdf").data == "Hello".bytes
        queryService.allDocuments.size() == 1

        when:
        commandService.executeCommand(new Command("removeDocument", [documentName: "Hello.pdf"]))
        then:
        queryService.getDocument("Hello.pdf") == null
        queryService.allDocuments.size() == 0
    }
}
```

Really basic. We'll save a new document, try to update that document and then remove that document. 

The Spring 3 context just contains both EventBus instances (synchronous in this article).

``` groovy
@Configuration
@ComponentScan
class TestConfiguration {
    @Bean @Qualifier("events")
    EventBus eventBus() {
         new EventBus()
    }

    @Bean @Qualifier("commands")
    EventBus commandBus() {
        new EventBus()
    }
}
```

Basically with CQRS, you have commands (write operations) and queries (read operations). We'll start off creating the command service.

``` groovy
@Service
class CommandService {
    EventBus commandBus

    @Autowired
    CommandService(@Qualifier("commands") EventBus commandBus, @CommandExecutor List<Object> commandExecutors) {
        this.commandBus = commandBus
        for (Object executor : commandExecutors) {
            commandBus.register(executor)
        }
    }

    public void executeCommand(Command command) {
        commandBus.post(command)
    }
}
```

The command service is responsible for executing commands and registering all the command executors found in the context in the command bus.

We know we have 2 commands, so we'll create 2 command executors. For the sake of simplicity, a command is a basic datastructure consisting of a type and a map of parameters. Events are modelled the same way (type and parameters). Both are immutable by nature.

``` groovy
@groovy.transform.Immutable
class Command {
    String commandType
    Map<String, Object> parameters = [:]
}

@groovy.transform.Immutable
class Event implements Serializable {
    String eventType
    Map<String, Object> parameters = [:]
}
```

CommandExecutors receive commands from the command bus and handle the commands that they support. Based on those commands, events are raised. In this example the command executor directly raises the events, but in real systems this will probably be the underlying components (repositories and such). I made a basic superclass to support command selection and event dispatching and an event publisher to be able to monitor which events are posted on the event bus.

``` groovy
abstract class CommandSubscriber {
    @Autowired @Delegate
    EventPublisher eventPublisher;

    @Subscribe
    void execute(Command command) {
        if(supportedCommandTypes().contains(command.commandType)) {
            onCommand(command)
        }
    }

    abstract void onCommand(Command command);

    abstract String[] supportedCommandTypes();
}

@Component
class EventPublisher {
    @Autowired @Qualifier("events")
    private EventBus eventBus

    def postEvent(Event event) {
        println "Event ${eventType} is raised!"
        eventBus.post(event)
    }
}
```

Implementing the command executors for both the save and remove commands is then quite trivial.

``` groovy
@CommandExecutor
@Component
class RemoveDocumentCommandExecutor extends CommandSubscriber {

    @Override
    void onCommand(Command command) {
        def event = new Event(eventType: "removedDocument", parameters: command.parameters)
        postEvent(event)
    }

    @Override
    String[] supportedCommandTypes() {
        ["removeDocument"]
    }
}
```

The save command executor also support raising 2 events: one when a new document is created and another when an existing document is updated. The lookup of the document is done by the queryservice (which we will explain in a bit).

``` groovy
@CommandExecutor
@Component
class SaveDocumentCommandExecutor extends CommandSubscriber {

    @Autowired
    private QueryService queryService;

    @Override
    void onCommand(Command command) {
        def event
        if(queryService.getDocument(command.parameters.documentName)) {
            event = new Event(eventType: "updatedDocument", parameters: command.parameters)
        } else {
            event = new Event(eventType: "savedDocument", parameters: command.parameters)
        }
        postEvent(event)
    }

    @Override
    String[] supportedCommandTypes() {
        ["saveDocument"]
    }
}
```

The CommandExecutor is a qualifier annotation so I can easily inject the list of command executors in my service.

Now for the query part. We now know that the application throws 3 events. We'll make component that consumes those events and builds up an view model based on those events. After that, the query service that use that component to expose that view with the needed methods.

``` groovy
@Component
class DocumentView {

    @Autowired
    DocumentView(@Qualifier("events") EventBus eventBus) {
        eventBus.register(this)
    }

    def documents = [] as List<Map>

    @Subscribe
    void onEvent(Event e) {
        switch(e.eventType) {
            case "savedDocument":
                documents << [name: e.parameters.documentName, data: e.parameters.documentData]
                break
            case "removedDocument":
                documents.removeAll { it.name == e.parameters.documentName }
                break
            case "updatedDocument":
                def document = documents.find { it.name == e.parameters.documentName}
                document.data = e.parameters.documentData
                break
        }
    }
}
```

Just like the command executors were registered on the command bus, this component will register itself on the event bus so it received all the events posted on that bus. Based on those events, it will create a view representing the state of the documents at that time.

Last but not least, the query service uses the view to provide the needed data retrieval methods.

``` groovy
@Service
class QueryService {
    @Autowired
    DocumentView view;

    def getDocument(String name) {
        view.documents.find { it.name == name }
    }

    def getAllDocuments() {
        view.documents
    }
}
```

And that's it. If you now run the test, it should succeed.

The decoupling of the component is what makes event-sourcing so powerful. I could just as easily create another event consumer creating an entirely different data model. I could create a component that writes the data to a persistent storage or send mails based on events.

You could also make the event busses asynchronous, although you have to alter your test to wait for events to be consumed (unit testing asynchonous communication is not easy).

If you want a real implementation of event-sourcing and CQRS, you should take a look at the [Axon framework](http://www.axonframework.org). The concepts are similar but way more complete that what I just showed you.
