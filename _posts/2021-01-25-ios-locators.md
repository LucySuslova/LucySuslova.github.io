---
layout: post
title:  "appium "
subtitle: "ios specific locator strategies"
date:   2021-01-25 09:40:39 +0300

---
Nowadays a lot of tools, libraries, frameworks or solutions emerge to facilitate test automation needs. 
Moreover, some of them are designed for codeless test automation applying artificial intelligence and machine learning.
And that's great, it means that this area does not stand still and is actively evolving.

In this post I want to go back to basics. To element locator strategies.

Robust test automation requires reliable element locators. It is extremely important. We should agree that it's not a good 
idea to copy kind of absolute XPaths from a browser's inspector. So you need to analyze an app or a website, its structure and 
decide what strategy to choose in this or that case.

Selenium WebDriver offers following [element selection strategies][element selection strategies]:

```
* class name	
* css selector
* id
* name
* link text
* partial link text
* tag name
* xpath
```

They are pretty well-known for all automation QAs, it's impossible to create any test script without locating proper elements. 
Some of them are used frequently, others quite rarely. 

When someone starts mobile test automation I guess it's natural to use what you know better. And we keep using IDs and XPaths.
But if you spend a minute to read Appium docs you'll find out that it offers something more.

[Appium][Appium] offers various strategies that can be used by AQAs:

```
* accessibility id
* id
* xpath
* class name
* locators interpreted by the underlying automation frameworks
* -image
```

Let's stop on ```locators interpreted by the underlying automation frameworks```.

When you use Appium for mobile test automation you define a specific driver that will be used for iOS and Android. 
It can be ```XCUITest``` for iOS, ```UiAutomator2```, ```Espresso``` drivers for Android. So you can make use of these 
native automation frameworks.

### iOS examples 

Appium has dedicated ```MobileBy``` that helps us to find mobile elements. It is used like this:

{% highlight java %}

String confirmBtnSelector = "**/XCUIElementTypeAlert/**/XCUIElementTypeButton[1]";
driver.findElement(MobileBy.iOSClassChain(confirmBtnSelector));
{% endhighlight %}

#### ```-ios predicate string```

We can combine different operators and use recognized [attributes][attributes] to build matchers for the specific element.

```MobileBy.iOSNsPredicateString("type == 'XCUIElementTypeButton' && label BEGINSWITH 'URL'");```  
matches button with the label's text that begins with 'URL'.

```MobileBy.iOSNsPredicateString("type=='XCUIElementTypeButton' AND name CONTAINS[c] 'back'")```  
matches button with the name that contains 'back'. [c] means case-insensitive. Such locator will match BACK, back, Back, etc 
in the element's name.

```MobileBy.iOSNsPredicateString("type == 'XCUIElementTypeStaticText' AND value ENDSWITH[cd] 'adiós'")```  
matches StaticText with value that ends with 'adiós' case and diacritic insensitive.

```MobileBy.iOSNsPredicateString("type == 'XCUIElementTypeButton' AND (name == 'hamburger' OR name == 'dashboardToolbar')")```  
matches button with name that is equal either to 'hamburger' or 'dashboardToolbar'.


#### ```-ios class chain```

Using ```-ios class chain``` you can perform **direct children** and **indirect descendant** [search][search].

```MobileBy.iOSClassChain("**/XCUIElementTypeLink[1]")```  
matches the first link.

```MobileBy.iOSClassChain("**/XCUIElementTypeLink[-1]")```  
matches the last link.

```MobileBy.iOSClassChain("**/XCUIElementTypeOther[`label == "picker"`]/**/XCUIElementTypeStaticText")```  
matches all descendant StaticTexts of the first ElementTypeOther with the label equal to 'picker'.
___
These locator strategies are not cross-platform. But in such way you can go a little deeper into understanding of a particular mobile platform. 

Starting from [version][version] 1.18.0 you can also find iOS specific locators inside Appium Inspector.

![Appium inspector]({{ site.baseurl }}/img/appium.png)




[Appium]: https://appiumpro.com/editions/60-how-to-pick-the-right-locator-strategy
[element selection strategies]: https://www.selenium.dev/documentation/en/getting_started_with_webdriver/locating_elements/#element-selection-strategies
[attributes]: https://github.com/facebookarchive/WebDriverAgent/wiki/Predicate-Queries-Construction-Rules#available-attributes
[search]: https://github.com/facebookarchive/WebDriverAgent/wiki/Class-Chain-Queries-Construction-Rules
[version]: https://github.com/appium/appium-desktop/releases/tag/v1.18.0