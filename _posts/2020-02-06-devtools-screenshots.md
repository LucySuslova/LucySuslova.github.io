---
layout: post
title:  "element screenshot "
subtitle: "selenium 4. chrome dev tools"
date:   2020-02-06 09:40:59 +0300

---

An example how to take element screenshot using Selenium 4 devtools API.

### Take element screenshot

First step is to get ```DevTools``` and create session:

{% highlight java %}
    private DevTools chromeDevTools;
    private final String gitHubUrl = "https://github.com/";
    private final String fileName = "github_screenshot.png";

    @BeforeEach
    public void setup() {
        open(gitHubUrl);
        ChromeDriver driver = (ChromeDriver) getWebDriver();
        chromeDevTools = driver.getDevTools();
        chromeDevTools.createSession();
    }
{% endhighlight %}

After that we can use ```chromeDevTools``` for taking screenshots. In order to do this we need to utilize 
```Page.captureScreenshot``` method:

{% highlight java %}
    @Test
    public void elementScreenshotTest() throws IOException {
        SelenideElement logo = $("header .octicon-mark-github");
        Viewport viewport = new Viewport(logo.getLocation().getX(), logo.getLocation().getY(),
                logo.getSize().getWidth(), logo.getSize().getHeight(), 1);

        String image = chromeDevTools.send(Page.captureScreenshot(Optional.of(PNG), Optional.empty(),
                Optional.of(viewport), Optional.empty()));
        writeToFile(image, fileName);

        Path content = Paths.get(fileName);
        assertEquals(content.getFileName().toString(), fileName);

        try (InputStream is = Files.newInputStream(content)) {
            Allure.addAttachment("Screenshot_Devtools", is);
        }
    }
{% endhighlight %}

The screenshot will be saved with 'github_screenshot.png' name to project's root folder and attached to Allure test report.

After all required actions and verifications let's clean up and delete our screenshot:

{% highlight java %}
    @AfterEach
    public void cleanup() throws IOException {
        Path path = Paths.get(fileName);
        Files.deleteIfExists(path);
    }
{% endhighlight %}

Voilà. In the same manner it is possible to take screenshots of full page with the help of devtools in [Selenium 4][selenium4].

Full code example is [here][repo].


[selenium4]: https://github.com/SeleniumHQ/selenium/projects/2
[repo]: https://github.com/LucySuslova/chrome-devtools-selenide