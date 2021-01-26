---
layout: post
title:  "appium "
subtitle: "deep links. real devices "
date:   2021-01-18 09:40:39 +0300

---

I've already [explained][explained] how deep links can help with in-app navigation when we have some test automation utilizing Espresso framework.
But the point is that such approach suites only for one platform - Android.

Mobile test automation can be done using different tools and frameworks. Tool selection should be based on the app itself, 
tech stack for app development, whether app is cross-platform and lots of other factors.
If we are talking about test automation in terms of cross-platform app it is fair to consider [appium][Appium] to be one of the 
best fits for this purpose.

So let's address one more example how to open deep links on real devices using Appium.  

#### Input

Deep links on Android can be easily opened just with

```
driver.execute(
      'mobile:deepLink',
      {
        url: "<deep-link-url>",
        package: "<package-name>"
      }
    );
```

For iOS, additional steps should be taken. We need to use Safari browser to open deep links.

As far as 'open deep link' implementation differs for iOS and Android let's consider a common solution.

#### Solution

I prefer writing code that is highly readable, so you don't need to spend a lot of time figuring out what's going on in 
the particular test or method. 

That's why I structured my implementation in this way, having common public method 
```public void openDeepLinkUrl(String url) {...}``` to call withing test script.

{% highlight java %}

    @Test
    public void openDeepLinkTest() {
        // do something before
        new DeepLinkUtils().openDeepLinkUrl("login/tom/mypassword");
        // assertions, etc.
    }
{% endhighlight %}

And we have platform specific implementations to open deep links.

For iOS:

{% highlight java %}

    private void openIosDeepLinkUrl(String url) {
        driver().terminateApp("io.cloudgrey.the-app");
        driver().activateApp("com.apple.mobilesafari");
        if (!$(urlButton).exists()) driver().getKeyboard();
        $(urlButton).shouldBe(Condition.visible).click();
        $(urlField).sendKeys(String.format("%s%s\uE007", PREFIX, url));
        $(openButton).shouldBe(Condition.visible).click();
    }
{% endhighlight %}

and for Android:

{% highlight java %}

    private void openAndroidDeepLinkUrl(String url) {
        driver().terminateApp("io.cloudgrey.the_app"); //optional
        HashMap<String, Object> args = new HashMap<>();
        args.put("url", PREFIX + url);
        args.put("package", "io.cloudgrey.the_app");
        driver().executeScript("mobile: deepLink", args);
    }
{% endhighlight %}


![Appium inspector]({{ site.baseurl }}/img/appium-deep-links.png)


Then we can have a cross-platform method:

{% highlight java %}

    public void openDeepLinkUrl(String url) {
        if (isAndroid()) openAndroidDeepLinkUrl(url);
        else openIosDeepLinkUrl(url);
    }
{% endhighlight %}

That's it. 

Full code example can be found [here][repo].



[appium]: http://appium.io/
[repo]: https://github.com/LucySuslova/appium-deeplinks
[explained]: https://lucysuslova.github.io/2020/01/16/espresso-deeplinks/