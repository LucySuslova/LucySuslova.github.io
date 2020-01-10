---
layout: post
title:  "how I met espresso and kotlin "
subtitle: "intro"
date:   2020-01-10 09:40:39 +0300

---

I'm really excited when I get a new project and I can build test automation from scratch. That's great to choose language, frameworks,
tools and approaches to use according to project needs. 
Sometimes it can be rather difficult because you get used to, for example, ```Java + Appium``` for mobile automation. 
And it's okay, ```Java + Appium``` works fine for any kind of app test automation.

But what if you need to automate only Android native app? Is it worthwhile to implement tests based on [Appium][appium] 
or is it better to consider other options?

And what are those options? 

#### The problem

So here's some input data I had when I started to think about the tech stack for the next project.

* Android native app only, no iOS app in plans
* App is written on [Kotlin][kotlin]
* App has specific features like barcode scanning
* App is quite small (it has not so many features)
* I have access to app's code 
* I was limited to use [Azure Pipelines][pipelines] (hosted macOS) instead of Jenkins or cloud solutions

#### Solution

In this case I wanted to find a lightweight solution that can be easily deployed on hosted macOS. 

As far as app is in Kotlin it was reasonable to use Kotlin for test automation as well. 
Test code base is stored with app code in one repo so I didn't want to have 'java code' there.
Moreover, I always wanted to try Kotlin :)

Let's move to the framework. Appium setup on Azure Pipelines hosted OS is time and resource consuming. 
And there was no need to support multiple platforms. Solution is to use [Espresso][espresso] - Android lightweight native framework.
It works almost out-of-box, you don't need any complicated configurations. 

I was totally excited.. at first. 
In fact Espresso covers almost all needs and requirements for UI test automation. And Kotlin is much more comfortable than Java for me now.


#### Conclusion

Of course, I encountered some limitations of Espresso and big 'pros' why to continue using it. 
I will describe several use cases in the following posts.

But now I want you to note that Espresso is the UI testing framework. Not end-to-end. 
Always choose carefully tools for your TAF based on the project requirements. 


[appium]: http://appium.io/
[espresso]: https://developer.android.com/training/testing/espressov
[kotlin]: https://github.com/JetBrains/kotlin
[pipelines]: https://azure.microsoft.com/en-us/services/devops/pipelines/
