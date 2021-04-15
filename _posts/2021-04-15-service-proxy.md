---
layout: post
title:  "Service proxy"
date:   2021-04-15 22:00:00 +0200
categories: architecture communication 
---



{% highlight java %}
@ExposedService
public interface Hello {
    Response<HelloResponse> hello(HelloRequest request);
    Response<RespondResponse> respond(RespondRequest request);
}
{% endhighlight %}

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


To call this API using the service proxy you call it as shown below.
What is so great about this? As you can see there are no references to HTTP, gRPC or something similar. The reason for this is that when the technology changes, we do not want to modify our logic, to put it differently, we shouldn't need to know if this is going over HTTP or if it's gRPC or even an in-process call. This also makes it easier during development and test to get the application up and running as you can switch so that all calls are done in-process.

{% highlight java %}
public class TestClient {
    public String hello(String name) {
        var service = ServiceProxy.create(Hello.class);
        var request = HelloRequest.newBuilder().setName(name).build();
        var result = service.sayHello(request)
            .then((r) -> service.respondToHello(r.result.getText()))
            .onError((error) -> {
                System.out.println(String.format("error: {} with message {}",error.getStatusCode(), error.getError());
                return new Response<HelloResponse>(error);
            });
        return result.getText();
    }
}
{% endhighlight %}
