---
layout: post
title:  "spring @PostConstruct annotation with retrofit2 "
subtitle: "to build http api endpoints"
date:   2020-01-03 23:45:13 -0400

---

Sometimes you may want to use [Retrofit2][retrofit2] and [Spring framework][spring] in your test automation project when developing automated scripts to test web services.
It is up to you to decide if it is good or bad to use such 'NOT-lightweight' framework as Spring in you TAF but that's completely another topic.
But if Spring + Retrofit is your case let's take a look at Spring @PostConstruct annotation.

####Regular code structure with Retrofit
Here's an example of regular code structure when we're using Retrofit.

First of all we have an interface with API endpoints:

{% highlight java linenos %}
        public interface GitHubService {
        @GET("users/{user}/repos")
        Call<List<Repo>> listRepos(@Path("user") String user);
        }
{% endhighlight %}
 
And a class where you configure service builder:
 
{% highlight java linenos %}
 Retrofit retrofit = new Retrofit.Builder()
     .baseUrl("https://api.github.com/")
     .build();
 
 GitHubService service = retrofit.create(GitHubService.class);
{% endhighlight %}
 
But what if you have plenty of services and you want to generates an implementation of each service interface? 
Here we can make use of ```@PostConstruct```.

####@PostConstruct - why to use?

Now we can modify code structure a bit and as a result we have ```ServiceBuilder.class```, ```GitHubEndpoints``` 
and ```TwitterEndpoints``` interfaces with endpoints and implementation classes ```GitHubService``` and ```TwitterService```. Let's take a look at each of these classes.

In ```ServiceBuilder.class``` we have following ```build()``` method:

{% highlight java linenos %}
 @Component
 public class ServiceBuilder {

    public <T> T build(Class<T> clazz) {
        OkHttpClient.Builder okHttpClientBuilder = new OkHttpClient.Builder();
        okHttpClientBuilder
                .readTimeout(120, TimeUnit.SECONDS)
                .writeTimeout(120, TimeUnit.SECONDS)
                .connectTimeout(120, TimeUnit.SECONDS);
        return getRetrofit(okHttpClientBuilder).create(clazz);
    }
    
    private Retrofit getRetrofit(OkHttpClient.Builder okHttpClientBuilder) {
        HttpLoggingInterceptor loggingInterceptor = new HttpLoggingInterceptor();
        loggingInterceptor.level(HttpLoggingInterceptor.Level.BODY);
        return new Retrofit.Builder()
                .baseUrl("https://base.url.com")
                .client(okHttpClientBuilder
                        .addInterceptor(loggingInterceptor)
                        .followRedirects(false)
                        .build())
                .addConverterFactory(GsonConverterFactory.create(
                        new GsonBuilder()
                                .setLenient()
                                .create()))
                .build();
    }
    
    //other stuff 
 }
{% endhighlight %}
 

And in 'service' classes we call ```build()``` passing the required class with endpoints. For instance,

{% highlight java linenos %} 
 @Component
 public class GitHubService {

    @Autowired
    private ServiceBuilder serviceBuilder;
    
    private GitHubEndpoints gitHubEndpoints;
    
    @PostConstruct
    public void init() {
        gitHubEndpoints = serviceBuilder
                .build(GitHubEndpoints.class);
    }
    
    //api calls go here
    
  }
{% endhighlight %}
 
So in this case we have simple and well-structured code.
 
[retrofit2]: https://square.github.io/retrofit/
[spring]: https://spring.io/