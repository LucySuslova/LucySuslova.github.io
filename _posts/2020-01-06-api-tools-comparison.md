---
layout: post
title:  "api test automation tools"
subtitle: "brief comparison"
date:   2020-01-06 09:40:39 +0300

---

If we are talking about API test automation in Java world there is one question - what tool to use? Maybe the most popular solution is [REST Assured][ra].
Indeed, RA is great and powerful, and it makes exactly what we need - sends requests an gets responses. Also you may heard about [Feign][feign].

Actually I find Feign to be very interesting because its creators were inspired by [Retrofit][retrofit], [JAXRS-2.0][jaxrs], and [WebSocket][ws] to make API clients easier.

#### Tools

When I need to create API TAF from scratch I analyse web services under test and general requirements. 
Also it's very important to estimate API flows complexity. If you need to make, for instance, 
direct calls to several endpoints there is no sense to build your own clients and write some extra code. 
REST Assured can handle it out-of-box. 
But sometimes you need to think about other solutions...

Basically, in most cases I have a choice between REST Assured, Retrofit and Feign. And here's the comparison.


### Retrofit

**Pros**

- Full access to the HTTP response
- Can retrieve POJO Instead of JSON
- Can make synchronous or asynchronous HTTP requests
- Uses annotations to describe the HTTP request
- Url manipulations allowed

{% highlight java %}
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId);
{% endhighlight %}


- Methods can be declared to send form-encoded and multipart data


{% highlight java %}
@Multipart
@PUT("user/photo")
Call<User> updateUser(@Part("photo") RequestBody photo, @Part("description") RequestBody description);
{% endhighlight %}


- Interceptors can be used to retrieve tokens etc. and add them to all (or by condition) requests
- Converters can be added to support different data types
- Pure type-safe HTTP client, not a framework. Can be extended with any runners/reports/config libs/assertion libs etc.
- Easy mocking

**Cons**

- Logging (need to create interceptor)
- Needs wrappers
- More coding


### Feign

**Pros**

- Allows you to write your own code on top of http libraries
- Connects the code to http APIs with minimal overhead and code via customizable decoders and error handling
- Inspired by Retrofit, gives flexibility
- Out of box logging
- Customization
- Built-in retry mechanism (Feign, by default, will automatically retry IOExceptions, regardless of HTTP method)

**Cons**

- Feign clients can be used to consume text-based HTTP APIs only, which means that they cannot handle binary data, 
e.g. file uploads or downloads.


### REST Assured*

**Pros**

- Easy setup
- Almost all needed features out of box
- Can be extended with assertion libs ([Harmcrest][hamcrest])

**Cons**

- HTTP Client

Apache 4.X, but the code depends on deprecated APIs. There are some concerns with this design. More details in this [issue][issue]

- Validate All Payload values in one step

* Not built-in. You need to use external libraries such as [Harmcrest][hamcrest]
* No easy way to do a “deep equals” comparison for nested objects
* No way to “ignore” fields - for e.g. id / date / time values which are dynamic

- No built-in data-type, conditional-logic and RegEx validations
- Impossible to validate schema of all elements in a JSON array in one step
- No native support for expressing JSON or XML in test-scripts

{% highlight json %}
"{ \"name\": \"Billie\" }"
{% endhighlight %}


{% highlight xml %}
"<cat name=\"Billie\"></cat>"
{% endhighlight %}

- Extracting multiple data-elements for reuse in subsequent HTTP calls

Convoluted.
The Fluent Interface which is supposed to be the main highlight of REST Assured actually [gets in the way here][extract-multiple-values]

- Can update a given JSON or XML using a path expression

[No][update-json]. Especially for data-driven tests, updating nested JSON is near impossible

- No SOAP support
- HTTPS/SSL without certificates

Although there is “relaxed” HTTPS, a certificate is needed in [some cases][ssl-config]

- Built-in support for switching environment config

Config is somewhat [convoluted][ra-config] in REST Assured. You have to use something like a DI framework (or roll your own) to read properties files

- Parallel test Execution

The creator has confirmed that “REST Assured is not 100% safe to use in multi-threaded scenarios”

- Report does not include HTTP request and response logs in-line
- Test Doubles or Mocks built-in

No. You have to use 3rd party frameworks such as [Wiremock][wiremock] etc

- [No][ra-ws] websocket support
- Async Support

No, you have to use Java code or a library like [Awaitility][awaitility]

###### *some RA points found in web and aggregated here


### Instead of conclusion

Frameworks and tools for API test automation are not limited to these three mentioned in the article. 
And you should choose carefully depending on the needs and requirements of your project. 

In short, general recommendations may be as following:

use **REST Assured** if you need to build reliable and maintainable test automation

use **Retrofit** if you need more flexibility and control

use **Feign** if you like experiments :)


[ra]: http://rest-assured.io/
[feign]: https://github.com/OpenFeign/feign
[retrofit]: https://github.com/square/retrofit
[jaxrs]: https://github.com/jax-rs
[ws]: https://www.oracle.com/technical-resources/articles/java/jsr356.html
[hamcrest]: http://hamcrest.org/JavaHamcrest/
[issue]: https://github.com/rest-assured/rest-assured/issues/497
[extract-multiple-values]: https://github.com/rest-assured/rest-assured/wiki/usage#extracting-values-from-the-response-after-validation
[update-json]: https://groups.google.com/forum/#!topic/rest-assured/uNGxBLB5uvI
[ssl-config]: https://www.javadoc.io/doc/com.jayway.restassured/rest-assured/latest/com/jayway/restassured/config/SSLConfig.html
[ra-config]: https://github.com/rest-assured/rest-assured/issues/239
[ra-ws]: https://github.com/rest-assured/rest-assured/issues/850
[wiremock]: http://wiremock.org/
[awaitility]: https://github.com/awaitility/awaitility