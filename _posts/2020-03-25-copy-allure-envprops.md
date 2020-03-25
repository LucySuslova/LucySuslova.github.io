---
layout: post
title: "copy allure 2 environment.properties "
subtitle: "via gradle task"
date: 2020-03-25 09:40:59 +0300

---
It is a common case to include some environment data into test report when using any reporting tool.
For instance, [Selenide (starting from v.5.5.1)][selenide_emulated_mobile] provides us out-of-box feature to run test 
scripts against emulated mobile Chrome browser. It is possible just adding one ```System property```:

```java -Dchromeoptions.mobileEmulation="deviceName=iPhone X"```

So now we want to display the environment in the test report whether it's desktop or mobile. If it's mobile, we want 
to have device name to be reflected in the report for future analysis.

[Allure 2][allure] reports support Environment widget where env data can be displayed. It is required to place environment.properties file 
with all the data we want to expose into allure-results folder before generating test report. 

#### How it can be done?

Actually it is a quite simple task. 

Let's create ```environment.properties``` file inside ```src/test/resources``` folder. 
The content of this file will be following:

{% highlight java %}
Run_On:${env}
{% endhighlight %}

This means that we have a placeholder ```${env}``` that should be replaced with the actual value when copying ```environment.properties```
file.

To perform copy operation let's create a gradle task. It is required to specify the source folder, what file and where to copy. 
Also, we need to replace the placeholder with actual device value. It can be done in 'expand' section:

{% highlight groovy %}
task copyEnvProps(type: Copy) {
    from 'build/resources/test'
    include 'environment.properties'
    into 'build/allure-results'
        expand([
                env: System.getProperty("chromeoptions.mobileEmulation") ?: "desktop"
        ])
}
{% endhighlight %}

Finally, we need to execute ```copyEnvProps``` task. As we know we need to do it after test run but before generating the report:

{% highlight groovy %}
tasks.withType(Test)*.finalizedBy 'copyEnvProps'
{% endhighlight %}

As a result we have an updated Allure report:

![Allure environment section]({{ site.baseurl }}/img/allure_env.png)

Full ```build.gradle``` example is [here][gist].


[gist]: https://gist.github.com/LucySuslova/83622c79eb677ba3cd31b310d7679fc1
[allure]: https://docs.qameta.io/allure/#_environment
[selenide_emulated_mobile]: https://selenide.org/2019/11/29/selenide-5.5.1/