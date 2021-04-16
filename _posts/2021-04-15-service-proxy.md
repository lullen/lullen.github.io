---
layout: post
title:  "Service proxy"
date:   2021-04-15 11:00:00 +0200
categories: architecture communication 
---

# Synchronous communication
When creating a system, you always need to talk between different components. There are two different types of communication, asynchronous and synchronous and in most systems you will need to use both. In this post I will focus on synchronous communication.

# When to use
Synchronous communication is used when you want to have a direct response on a request. You want to have a direct response when you are calling downwards in your architecture. This is because it makes sense to have a tighter coupling when calling downwards. Think of an web application where the controller is using async communication as in using a message bus to talk with the data access layer. While possible it doesn't make sense as it's in the same unit of work. When calling sideways you should not use sync communication as you are then coupling two different unit of work together. In the picture the solid line is synchronous and the dotted line is asynchronous.

![Sync and async communication](/assets/2021-04/sync-async-communication.jpg)

# Transport agnostic communication
When we communicate between different components we do not care codewise what medium for transportation we're using. If it's in-proc, HTTP or gRPC doesn't matter, our code should look the same. All we care about is that the transportation is fast and reliable. When the technology changes, as it always does, we do not want to change the whole application to support the new technology. We want to change one place where the call is happening and be fine with it. This also gives us the ability to run the application in different ways during development and testing where it could be benificial to keep the number of services running as low as possible to speed up the inner-loop and to save your precious RAM.


# Service proxy
To support this you need a service proxy that does the communication for you whether it's over the wire or in-process. When you then have a single point where all the communication goes through, you have a perfect place where you can add logging, timing, exception handling and everthing else that you expect to have when communicating with other components.

# Implementation
I have started to build a [small library](https://github.com/lullen/service-proxy) that supports this idea, while not completed it is functional enough to show how it could look like.


First we define an interface and tag it with the ExposedService attribute. The methods should only have one parameter and it should be an object. The return value should be of type `Response<T>`. What the `Response<T>` does is to give you a way handle exceptions and errors in a uniform way and gives you the opportunity to join calls. Both the parameter and return value must be ProtoBuf objects, this is a small code-smell that could be fixed in a future version. At the moment I have focused on getting the service proxy to work with [Dapr](https://dapr.io) and in-process but could easily be extended to support HTTP or gRPC without Dapr.

{% highlight java %}
@ExposedService
public interface Hello {
    Response<HelloResponse> hello(HelloRequest request);
    Response<RespondResponse> respond(RespondRequest request);
}
{% endhighlight %}

Then we should add an implementation to the interface, exactly as you would do with any other interface. 

{% highlight java %}
public class HelloServiceImpl implements Hello {
    public Response<HelloResponse> hello(HelloRequest request) {
        var text = String.format("Hello there {}.", request.getName());
        var response = HelloResponse.newBuilder().setText(text).build();
        return new Response<HelloResponse>(response);
    }
    
    public Response<RespondResponse> respond(RespondRequest request) {
        var text = String.format("{} Hello to you as well.",request.getGreeting());
        var response = RespondResponse.newBuilder().setText(text).build();
        return new Response<RespondResponse>(response);
    }
}
{% endhighlight %}

To configure the server, this is what is currently needed.

{% highlight java %}
public static void main(String[] args) throws IOException, InterruptedException {
    var injector = Guice.createInjector(new DaprModule(), new ProxyModule());
    ServiceProxy.init(ProxyType.Dapr, injector);

    final var service = injector.getInstance(DaprServer.class);
    var port = 5000;
    service.start(port);
    service.registerServices(List.of(Hello.class));
    service.awaitTermination();
}

// Guice module
public class DaprModule extends AbstractModule {
    @Override
    protected void configure() {
        bind(Hello.class).to(HelloServiceImpl.class);
        bind(DaprServer.class);
    }
}
{% endhighlight %}


To configure the client to use the proxy, this is what is currently needed.

{% highlight java %}
public static void main(String[] args) throws Exception {
    var injector = Guice.createInjector(new ProxyModule());
    ServiceProxy.init(ProxyType.Dapr, injector);

    var text = new TestClient().hello("Ludwig");
    System.out.println(text);
}
{% endhighlight %}

Now that it is implemented we can finally call it using the code below and as you can see there are no references to HTTP, gRPC or something similar. If something returns an error or exception the next then() won't be called and instead the onError method will be called with a status code and an error message. You do not need to have the onError method to catch the exception as it's done lower in code and you never need to surround the service calls with a try/catch. 

{% highlight java %}
public class TestClient {
    public String hello(String name) {
        var service = ServiceProxy.create(Hello.class);
        var request = HelloRequest.newBuilder().setName(name).build();
        var result = service.sayHello(request)
            .then((r) -> {
                var request = RespondRequest.newBuilder().setGreeting(r.result.getText()).build();
                service.respondToHello(request);
            })
            .onError((error) -> {
                System.out.println(String.format("error: {} with message {}",error.getStatusCode(), error.getError());
                return new Response<HelloResponse>(error);
            });
        return result.getText();
    }
}
{% endhighlight %}

# In-proccess

If you instead want to communicate in-process all you need to do is to change the configuration. The application that you start should reference the client and the server and it's main method should look something like.

{% highlight java %}
public static void main(String[] args) throws Exception {
    var injector = Guice.createInjector(new ProxyModule(), new DaprModule());

    // Setup service proxy to in-proc mode
    ServiceProxy.init(ProxyType.InProc, injector);

    // Initialize the service loader so the proxy knows how to instantiate the services.  
    ServiceLoader.init(injector);
    ServiceLoader.registerServices(List.of(Hello.class));

    var text = new TestClient().hello("Ludwig");
    System.out.println(text);
}
{% endhighlight %}

# Future enhancements
In the future I want to make it easier to configure the services. Today you need to specify which interfaces you want to host, that should of course be done automatically. I want to be able to switch between different DI containers more easily and lastly I want support more transportation mediums. 