---
layout: post
title:  "kotlin "
subtitle: "developing test scripts. test robots"
date:   2020-01-15 09:40:39 +0300

---

I was working with Java so it wasn't a problem to switch to Kotlin. Indeed, Kotlin is cool, 
its features help a lot to reduce code lines, it provides null-safety mechanism, data classes and other cool features. 

My plan is to highlight some language features and give a short example of usage. Of course, 
it is not a complete overview of all existing features, just those of them that I used when developing test scripts with espresso for Android native app.

So let's start.

#### robot pattern + kotlin (objects, ```apply()```)

You may heard about Page Object pattern that is widely used in test automation. For Espresso tests there is [```Robot```][robot] pattern - 
each 'robot' contains functions that correspond to specific user actions on a specific (one) screen. Robots can be implemented in 
couple of ways especially with the power of Kotlin.

Robot:

{% highlight kotlin %}
//imports

object EnvironmentsRobot {

    fun chooseStage = apply {
        onView(withId(R.id.stageEnv)).perform(click())
    }

    fun chooseTest= apply {
        onView(withId(R.id.testEnv)).perform(click())
    }

    fun chooseProd = apply {
        onView(withId(R.id.prodEnv)).perform(click())
    }
    
    fun close = apply {
        onView(withId(R.id.close)).perform(click())
    }
    
}

{% endhighlight %}

And in your test you can use it as follows:

{% highlight kotlin %}
    @Test
    fun switchEnvironmentTest() {
        EnvironmentsRobot
            .chooseTest()
            .close()
            .chooseStage()
            .close()
            .chooseProd()
            .close()
    }          
{% endhighlight %}

We make use of Kotlin objects and scope function ```apply()``` to enable method chaining. You can read more about scope functions [here][scope_fn].

But we can go further to make test code more readable and compact. Let's modify our 'robot'. Again we are using ```apply()``` 
function and it returns the object itself. Also note that our 'robot' is not an object anymore. It is a simple class now:

{% highlight kotlin %}
//imports

fun switchEnvironmentRobot(func: EnvironmentsRobot.() -> Unit) = EnvironmentsRobot()
        .apply { func() }

class EnvironmentsRobot {

    fun chooseStage() = onView(withId(R.id.stageEnv)).perform(click())

    fun chooseTest() = onView(withId(R.id.testEnv)).perform(click())

    fun chooseProd() = onView(withId(R.id.prodEnv)).perform(click())

    fun close() = onView(withId(R.id.close)).perform(click())

}
{% endhighlight %}

And here's an updated test:

{% highlight kotlin %}
    @Test
    fun switchEnvironmentsTest() {
        switchEnvironmentRobot {
            chooseStage()
            close()
            chooseTest()
            close()
            chooseProd()
            close()
        }
    }
{% endhighlight %}

You can create as many robots as needed and use them in one test. The test will look like this:

{% highlight kotlin %}

    @Test
    fun switchEnvironmentsTest() {
    
        switchEnvironmentRobot {
            //actions
        }
        
        loginRobot {
            //actions
        }    
        
        syncRobot {
            //actions
        }      
        
    }

{% endhighlight %}

Simple and clear. 


[robot]: https://academy.realm.io/posts/kau-jake-wharton-testing-robots/
[scope_fn]: https://kotlinlang.org/docs/reference/scope-functions.html