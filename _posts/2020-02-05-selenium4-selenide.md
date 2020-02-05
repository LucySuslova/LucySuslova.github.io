---
layout: post
title:  "selenium 4. chrome dev tools "
subtitle: "basic usage"
date:   2020-02-05 09:40:39 +0300

---

[Selenium 4][selenium4] is now in alpha (4.0.0-alpha-4) and we can try out its new features. They introduced native 
support for [Chrome DevTools][devtools]. This means that we can use Chrome Development properties such as Console, 
Network etc. from our test scripts. Cool, yeah?

Let's test it with [Selenide][selenide].

First of all we need to get DevTools and create session:

{% highlight java %}
    private DevTools chromeDevTools;
    private final String imdbUrl = "https://www.imdb.com/";

    @BeforeEach
    public void setup() {
        open(imdbUrl);
        ChromeDriver driver = (ChromeDriver) getWebDriver();
        chromeDevTools = driver.getDevTools();
        chromeDevTools.createSession();
    }
{% endhighlight %}

Then we can implement test scripts that are using devtools in various flows.

### Go offline

{% highlight java %}
    @Test
    public void goOfflineTest() {
        chromeDevTools.send(Network.enable(Optional.empty(), Optional.empty(), Optional.empty()));
        chromeDevTools.send(
                emulateNetworkConditions(true, 0, 0, 0, Optional.empty()));
        open(imdbUrl);
        $(".error-code").shouldHave(Condition.text("ERR_INTERNET_DISCONNECTED"));
    }
{% endhighlight %}

### Mobile device emulation

{% highlight java %}
    @Test
    public void webToMobileTest() {
        String mAgent = "Mozilla/5.0 (Linux; Android 8.0; Pixel 2 Build/OPD3.170816.012) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Mobile Safari/537.36";
        chromeDevTools.send(Network.enable(Optional.empty(), Optional.empty(), Optional.empty()));
        chromeDevTools.send(Network.setUserAgentOverride(mAgent, Optional.empty(), Optional.empty()));
        open(imdbUrl);
        assertEquals("https://m.imdb.com/", getWebDriver().getCurrentUrl(), "Opened imdb version is not mobile");
    }
{% endhighlight %}

### Adding custom headers to requests

{% highlight java %}
    @Test
    public void addCustomHeaderTest() {
        chromeDevTools.send(Network.enable(Optional.empty(), Optional.empty(), Optional.empty()));
        chromeDevTools.send(Network.setExtraHTTPHeaders(new Headers(ImmutableMap.of("testCustomHeader",
                "testCustomHeaderValue"))));
        chromeDevTools.addListener(Network.requestWillBeSent(), requestWillBeSent ->
                assertEquals(requestWillBeSent.getRequest().getHeaders().get("testCustomHeader"),
                        "testCustomHeaderValue"));
        open(imdbUrl);
    }
{% endhighlight %}

Each sent request will contain this ```testCustomHeader: testCustomHeaderValue``` key-value pair in headers.
_______

Please find full code examples in the [github repo][repo].


[selenium4]: https://github.com/SeleniumHQ/selenium/projects/2
[devtools]: https://chromedevtools.github.io/devtools-protocol/tot/
[selenide]: https://github.com/selenide/selenide
[repo]: https://github.com/LucySuslova/chrome-devtools-selenide