---
layout: post
title:  "testcontainers "
subtitle: "selenide. junit5"
date:   2020-02-20 09:40:59 +0300

---

This is a short example how to launch tests in dockerized browsers using testcontainers lib, [JUnit5][junit5] and [Selenide][selenide]. 

Prerequisites: docker installed and running.

### What is testcontainers?

According to the official website [Testcontainers][testcontainers] is a Java library that supports JUnit tests, 
providing lightweight, throwaway instances of common databases, Selenium web browsers, or anything else that can run 
in a Docker container. 

### Testcontainers, JUnit5, Selenide

Testcontainers support JUnit5 but still has dependencies on JUnit4. JUnit5 integration is achieved with the help of extension model.
Integration with Selenide is quite simple. We just need to tell Selenide to use the driver created by testcontainers.

First of all it is required to annotate the class with ```@Testcontainers``` annotation:

{% highlight java %}
//imports...

@Testcontainers
public class SetupBase {
//...
}
{% endhighlight %}

And then to create the container itself. 

Extra configurations like saving video records of all tests or just failed ones 
are also available. In this code example container will be restarting before each test method. 
Also it is possible to create a shared container that will be reused by all methods of a test class. Note to place ```@Testcontainers``` 
annotation on the class where you are creating ```@Container```.

{% highlight java %}
    @Container
    private final BrowserWebDriverContainer browserInContainer =
            new BrowserWebDriverContainer()
                    .withRecordingMode(BrowserWebDriverContainer.VncRecordingMode.RECORD_ALL, new File("build"))
                    .withCapabilities(provideCaps(System.getProperty("selenide.browser")));
{% endhighlight %}

To get some flexibility what browser to use I have ```provideCaps(String browser)``` method that creates the set of 
capabilities to be used on browser creation based on passed parameter.

At this point we have container created and started. Now we need to tell Selenide what driver to use:

{% highlight java %}
    @BeforeEach
    public void setup() {
        RemoteWebDriver driver = browserInContainer.getWebDriver();
        WebDriverRunner.setWebDriver(driver);
        SelenideLogger.addListener("AllureSelenide", new AllureSelenide().savePageSource(false).screenshots(true));
        open();
    }
{% endhighlight %}

That's all. Write your tests as always and they will be executed in docker container. 
Full code example can be found [here][repo].


[repo]: https://github.com/LucySuslova/selenide-testcontainers
[testcontainers]: https://www.testcontainers.org/
[selenide]: https://github.com/selenide/selenide
[junit5]: https://junit.org/junit5/