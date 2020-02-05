---
layout: post
title:  "selenium 4. chrome dev tools  "
subtitle: "fake geolocation"
date:   2020-02-05 09:40:59 +0300

---

Sometimes our test scenarios require to fake geolocation. Now it's a piece of cake with [Selenium 4][selenium4].

Here's a simple test that opens Google Maps and changes geolocation: 

### Fake location

{% highlight java %}
    private DevTools chromeDevTools;
    private final String mapsUrl = "https://www.google.com/maps/";
    
    @BeforeEach
    public void setup() {
        open(mapsUrl);
        ChromeDriver driver = (ChromeDriver) getWebDriver();
        chromeDevTools = driver.getDevTools();
        chromeDevTools.createSession();
    }

    @Test
    public void fakeLocationTest() {
        Number latitude = 37.422290;
        Number longitude = -122.08405;

        $("h1").shouldBe(visible);

        chromeDevTools.send(
                Emulation.setGeolocationOverride(Optional.of(latitude), Optional.of(longitude), Optional.of(100)));

        sleep(2000);
        open(mapsUrl);
        sleep(2000);

        $("#widget-mylocation").click();
        waitForUrlChanged(4000);

        assertAll(() -> {
            assertTrue(getWebDriver().getCurrentUrl().contains(longitude.toString()),
                    "Opened url does not contain fake longitude");
            assertTrue(getWebDriver().getCurrentUrl().contains(latitude.toString()),
                    "Opened url does not contain fake latitude");
        });

    }
{% endhighlight %}

I noticed that it takes some time to apply geolocation changes. That's why there should be some delays like ```sleep(2000);```.

Full code example is [here][repo].


[selenium4]: https://github.com/SeleniumHQ/selenium/projects/2
[repo]: https://github.com/LucySuslova/chrome-devtools-selenide