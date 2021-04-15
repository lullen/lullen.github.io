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

{% highlight java %}
public class HelloServiceImpl implements Hello {
    public Response<HelloResponse> hello(HelloRequest request) {
        var text = String.format("Hello there {}.", request.getName());
        var response = HelloResponse.newBuild().setText(text).build();
        return new Response<HelloResponse>(response);
    }
    
    public Response<RespondResponse> respond(RespondRequest request) {
        var text = String.format("{} Hello to you as well.",request.getGreeting());
        var response = RespondResponse.newBuild().setText(text).build();
        return new Response<RespondResponse>(response);
    }
}
{% endhighlight %}
