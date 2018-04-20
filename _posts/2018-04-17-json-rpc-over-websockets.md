---
layout: post
category : article
title: "JSON-RPC over WebSockets? Why not?"
comments: true
tags : [technical]
---

A lot of people have a personal pet project that has a simple domain and which allows them do play around with new technologies and try out some new (and sometimes crazy ideas). In my case, I have a REST API designed for POS systems for restaurants, as I have some experience with building software like this, and I know the domain quite well.

Since one of my previous projects involved JSON-RPC, I thought to myself, how can I make this a bit more sexy and challenging? Well, what I came up with was building a JSON-RPC system using an asynchronous communication like WebSockets. Aside from the fact that this may be totally useless to most people, I was up for the challenge to try and build a system.

In hindsight, it wasn't really that big of a challenge. There are quite a few libraries out there that do the heavy lifting and in the end, it took me about a couple of hours to get something working. I used jsonrpc4j for the JSON-RPC part and Spring Boot to handle WebSocket interaction.

So first I started by enabling WebSocket support in my Spring Boot app:

{% highlight kotlin %}
@Configuration
@EnableWebSocketMessageBroker
class WebSocketConfig : WebSocketMessageBrokerConfigurer {
    override fun registerStompEndpoints(registry: StompEndpointRegistry) {
        registry.addEndpoint("/websocket")
                .withSockJS()
    }
}
{% endhighlight %}

Then we create a JSON-RPC service. 

{% highlight kotlin %}
@JsonRpcService("/product")
interface ProductJsonRpcService {
    fun findProducts(nameContains: String): List<ProductJson>
}

@Component
@JsonRpcService("/product")
class ProductJsonRpcServiceImpl(private val findProducts: FindProducts) : ProductJsonRpcService {

    override fun findProducts(nameContains: String): List<ProductJson> {
        return findProducts.perform(FindProducts.Request(nameContains)).toJsonList()
    }
}
{% endhighlight %}

For the sake of brevity, we assume here that we have a simple usecase `FindProducts` that provides the backend functionality. In this example, let's assume this will always return a product `steak`.

Now that we have the JSON-RPC service, we can expose it with a websocket.

{% highlight kotlin %}
@Controller
class WebsocketJsonRpcController(productJsonRpcService: ProductJsonRpcService,
                                 val objectMapper: ObjectMapper) {

    val jsonRpcServer = JsonRpcBasicServer(productJsonRpcService)

    @MessageMapping("/json-rpc-request/product")
    @SendToUser("/json-rpc-reply/product")
    fun handleProductServiceCall(@Payload value: String, requestHeaders: SimpMessageHeaderAccessor) : JsonNode  {
        val outputStream = ByteArrayOutputStream()
        jsonRpcServer.handleRequest(value.byteInputStream(), outputStream)
        return objectMapper.readTree(outputStream.toString("UTF-8"))
    }

}
{% endhighlight %}

If a web socket is opened on `http://localhost:8080/websocket/json-rpc-request/product` and send a valid JSON-RPC payload, the correct method will be called on the service and the message will be send to a user reply channel. The reason for this is that this way, only the user that has sent the request will receive the reply. 

And that's it. But how would you use this in practice?

Well, I first needed to build some kind of client that was able to call these kinds of services. What I came up with was this.

{% highlight kotlin %}
abstract class JsonRpcClientService(val session: StompSession, val objectMapper: ObjectMapper, replyChannel: String) : StompFrameHandler {
    val latches = mutableMapOf<String, CountDownLatch>()
    val returnValues = mutableMapOf<String, JsonNode>()

    init {
        session.subscribe(replyChannel, this)
    }

    override fun handleFrame(headers: StompHeaders?, payload: Any?) {
        val reply = payload as JsonNode
        val id = reply.get("id").asText()
        returnValues.put(id, reply)
        latches[id]!!.countDown()
    }

    override fun getPayloadType(headers: StompHeaders?) = JsonNode::class.java

    abstract fun getRequestChannel() : String

    internal inline fun <reified T : Any> invokeAndReturnList(methodName: String, params: List<Any>): List<T> {
        val id = UUID.randomUUID().toString()
        val latch = createLatchForId(id)
        val payload = JsonRpcRequest(id, methodName, params)
        session.send(createHeaders(id), objectMapper.writeValueAsString(payload))
        latch.await()
        val response = returnValues[id]!!
        cleanupResponses(id)
        val resultNode = response.get("result")
        val type = objectMapper.typeFactory.constructCollectionType(List::class.java, T::class.java)
        return objectMapper.readValue<List<T>>(objectMapper.treeAsTokens(resultNode), type)
    }

    internal inline fun <reified T : Any> invokeAndReturnSingle(methodName: String, params: List<Any>) : T  {
        val id = UUID.randomUUID().toString()
        val latch = createLatchForId(id)
        val payload = JsonRpcRequest(id, methodName, params)
        session.send(createHeaders(id), objectMapper.writeValueAsString(payload))
        latch.await()
        val response = returnValues[id]!!
        cleanupResponses(id)
        val resultNode = response.get("result")
        return objectMapper.readValue<T>(objectMapper.treeAsTokens(resultNode), T::class.java)
    }

    private fun cleanupResponses(id: String) {
        returnValues.remove(id)
        latches.remove(id)
    }

    private fun createLatchForId(id: String): CountDownLatch {
        val latch = CountDownLatch(1)
        latches.put(id, latch)
        return latch
    }

    private fun createHeaders(id: String): StompHeaders {
        val stompHeaders = StompHeaders()
        stompHeaders.destination = getRequestChannel()
        stompHeaders.id = id
        customizeHeaders(stompHeaders)
        return stompHeaders
    }

    open fun customizeHeaders(stompHeaders: StompHeaders) {
    }

    data class JsonRpcRequest(val id: String, val method: String, val params: List<Any>)

}
{% endhighlight %}

This is a basic abstract class for JSON-RPC clients over WebSockets. To use it to call our service, we have the following subclass.

{% highlight kotlin %}
class ProductServiceClient(session: StompSession, objectMapper: ObjectMapper) : JsonRpcClientService(session, objectMapper, "/user/json-rpc-reply/product") {
    override fun getRequestChannel() = "/json-rpc-request/product"

    fun find(nameContains: String) : List<ProductJson>? {
        val methodName = "findProducts"
        val param = nameContains
        return invokeAndReturnList(methodName, listOf(param))
    }
}
{% endhighlight %}

This will send a message on `/json-rpc-request/product`, while listening for the reply on `/user/json-rpc-reply/product`. 

This code requires you to build a `StompSession`. If you want to know how to create a one, you can do something like this.

{% highlight kotlin %}
private fun createSession(objectMapper: ObjectMapper): StompSession {
    val transports = listOf(WebSocketTransport(StandardWebSocketClient()), RestTemplateXhrTransport())
    val sockJsClient = SockJsClient(transports)
    val stompClient = WebSocketStompClient(sockJsClient)
    val mappingJackson2MessageConverter = MappingJackson2MessageConverter()
    mappingJackson2MessageConverter.objectMapper = objectMapper
    stompClient.setMessageConverter(mappingJackson2MessageConverter)
    val framehandler = SessionHandler()
    val handshakeHeaders = WebSocketHttpHeaders()
    handshakeHeaders.set("Company", "1")
    return stompClient.connect("http://localhost:8088/websocket", handshakeHeaders, framehandler).get(1, TimeUnit.SECONDS)
}

class SessionHandler() : StompSessionHandlerAdapter() {
    override fun handleFrame(headers: StompHeaders?, payload: Any?) {
    }
}
{% endhighlight %}

Mind you, this is all a very naive implementation of the functionality needed to make this work. There are quite a few corner cases that haven't been covered (error handling, security, concurrent usage), but I hope you see the possibilities in combining 2 technologies to create interesting synergies.