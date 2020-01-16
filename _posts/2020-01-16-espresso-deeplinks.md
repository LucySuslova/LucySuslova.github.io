---
layout: post
title:  "espresso "
subtitle: "navigation. deep links"
date:   2020-01-16 09:40:39 +0300

---

When you automate user actions inside the app it's clear that your test script gonna interact with multiple views/activities. 
There are some cases when you need to open a specific screen, for instance, 'Edit Item Screen', bypassing all previous steps, 
e.g. login, item search, etc.

There are a lot of solutions how to [test a single fragment][fragment] in isolation or validate and stub [intents][intent] 
sent out by the app under test. 

But, for example, launching a single fragment needs a container (e.g. activity). Moreover, if you need to test a separate 
screen and the app architecture is based on DI ([Dagger][dagger] or whatever) you need to mirror app code providing test implementations. 
And if you are not an Android developer or have limited access to app's code, it can be challenging. 

In this post I want to talk about deep links and how they can help with in-app navigation for test automation.

In Android, a deep link is a link that takes you directly to a specific destination within an app.[ref][deeplink]

Let's give it a try.

#### Input

We have an app with single MainActivity and multiple fragments. Navigation graph contains 'login' fragment, 
'user pets' fragment, 'search for a pet' fragment, and 'edit pet' fragment. 
And it is required to open the 'edit pet' fragment omitting the other three.

#### Solution

To launch the fragment we can have the following function where activity launches with custom intent:

{% highlight kotlin %}
    fun launchFragment(activityRule: ActivityTestRule<MainActivity>, destinationId: Int, arg: Bundle? = null) {
        val launchFragmentIntent = buildLaunchFragmentIntent(destinationId, arg)
        activityRule.launchActivity(launchFragmentIntent)
    }
{% endhighlight %}

Of course, we need to pass the ```id``` of the fragment we want to launch ```destinationId: Int```.

On Edit Pet screen we can see selected pet's data in separate fields that can be edited. ```launchFragment``` function 
accepts optional ```arg: Bundle?``` because we need to pass a pet that should be edited. 
In such case fields of edit screen will be populated with pet's data.


Function ```buildLaunchFragmentIntent``` is the place where we're building a deep link to the required fragment (```destinationId: Int```) directly:

{% highlight kotlin %}
    fun buildLaunchFragmentIntent(destinationId: Int, arg: Bundle?): Intent =
        NavDeepLinkBuilder(context)
            .setGraph(R.navigation.navigation_graph)
            .setComponentName(MainActivity::class.java)
            .setDestination(destinationId)
            .setArguments(arg)
            .createTaskStackBuilder().intents[0]
{% endhighlight %}

So just passing the fragment id to the ```launchFragment``` function will do the magic and launch the specified fragment.

One question still remains open - ```arg: Bundle?```. How to create our pet (Bundle) to pass it to the link builder and display on edit pet screen?

Our pet should have type, name, and age. These values are passes as parameters to ```createTestBundle``` function:

{% highlight kotlin %}
    fun createTestBundle(petType: String?, petName: String?, petAge: Int?): Bundle? {
        val bundle = Bundle()
        petType?.let { bundle.putString("petType", it) }
        petName?.let { bundle.putString("petName", it) }
        petAge?.let { bundle.putInt("petAge", it) }
        return bundle
    }
{% endhighlight %}

Finally we can use all these functions in the test class:

{% highlight kotlin %}
    //imports
    
    @RunWith(AndroidJUnit4::class)
    class EditPetTest : BaseTest() {
    
        companion object {
            val petToEdit = Utils.createTestBundle("cat", "Oreo", 2)
        }
    
        @get:Rule
        var activityRule = ActivityTestRule(MainActivity::class.java)
    
        @Before
        fun setup() {
            launchFragment(activityRule, R.id.editPetFragment, petToEdit)
        }
    
        @Test
        fun editPetTest() {
            
            //fragment is already launched here
         
        }
    
    }
{% endhighlight %}


[intent]: https://developer.android.com/training/testing/espresso/intents
[fragment]: https://medium.com/@aitorvs/isolate-your-fragments-just-for-testing-ea7d4fddcba2
[dagger]: https://dagger.dev/
[deeplink]: https://developer.android.com/guide/navigation/navigation-deep-link