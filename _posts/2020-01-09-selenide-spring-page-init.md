---
layout: post
title:  "springboot + selenide for ui test automation "
subtitle: "use case"
date:   2020-01-09 09:40:39 +0300

---

One of my recent projects used [Selenide][selenide] and then [SpringBoot][springboot] in the tech stack for UI test automation. 

And I encountered one problem there. 

#### The Problem

For that test automation project I implemented ```Page Object``` and ```Page Component``` patterns. 
It gives better code structure, better readability and, of course, better code maintainability. 
When SpringBoot was integrated I wanted to inject 'page component' into 'page object' using Spring DI mechanism.

And what happened? Unfortunately, I was not able to do so. 

#### Input

I had a helper method ```at()``` that creates a page object instance and makes the test script more readable:

{% highlight java %}

public static <T> T at(Class<T> pageClass) {
        return Selenide.page(pageClass);
    }
    
{% endhighlight %}

When you develop a test script it looks like following:

{% highlight java %}

@Test
public void userCanRegisterTest(User user){

        at(HomePage.class)
                .acceptCookies()
                .registerUser(user);
                
        //rest of the test  
         
}

{% endhighlight %}

So you know exactly "where you are" - what page object you are currently working with. I find it pretty convenient.

And here's the code that did not work for me:

{% highlight java %}

@Component
public class HomePage {

    @Autowired
    private Header header;
    
    @Autowired
    private Footer footer;
    
    ...
    
}

{% highlight java %}

Note that I didn't create ```Header``` or ```Footer``` with ```new``` anywhere in the code 
and ```Header``` and ```Footer``` classes were annotated with ```@Component```.

So these autowired fields are never resolved and are ```null```. This is actually quite obvious but if you are new to Spring/Selenide it might be a little bit disappointing.
The problem is how Selenide instantiates page objects. It uses ```newInstance()``` to create page object. And Spring does not work this way.

#### Solution

Actually Spring has a mechanism how to handle autowiring in objects which are created with ```new``` or ```newInstance()``` etc.
You can check several approaches described in [this issue][issue] on StackOverflow.

But frankly speaking I think this is not a good idea. It's better to rework your design not to overcomplicate your TAF. 

Still, it's test automation.. keep it simple.



[selenide]: https://github.com/selenide/selenide
[springboot]: https://spring.io/projects/spring-boot
[issue]: https://stackoverflow.com/questions/52355132/spring-autowired-on-a-class-new-instance