---
layout: post
title:  "Service proxy"
date:   2021-04-15 22:00:00 +0200
categories: architecture communication 
---



{% highlight java %}
@ExposedService
public interface Hello {
    Response<HelloResponse> sayHello(HelloRequest request);
    Response<HelloResponse> respondToHello(HelloRequest request);
}
{% endhighlight %}


{% highlight java %}
public class TestClient {
    public String hello(String name) {
        var service = ServiceProxy.create(Hello.class);
        var request = HelloRequest.newBuilder().setName(name).build();
        var result = service.sayHello(request)
            .then((r) -> service.respondToHello(r.result.getText()));
    }
}
{% endhighlight %}

{% highlight java %}
public class HelloServiceImpl implements Hello {
    public String hello(String name) {
        return String.format(name);
    }
}
{% endhighlight %}
