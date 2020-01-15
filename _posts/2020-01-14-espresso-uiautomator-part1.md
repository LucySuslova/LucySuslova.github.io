---
layout: post
title:  "espresso "
subtitle: "+ uiautomator. part 1"
date:   2020-01-14 09:40:39 +0300

---

I want to share some tips I discovered for myself while working on Android native app test automation with Espresso and Kotlin. 

Espresso can cover a lot of UI testing needs inside the app. It gives you full access to the app and you can start with any state you need, 
you can manage shared preferences or you can launch specific intent or whatever you need. 

But when it comes to, for instance, system components, notification shade or interaction with hardware buttons you need an extra help. 
This help comes from [UiAutomator][ui_a].

[UiAutomator][ui_a] is a part of Android SDK. It does not require any complicated installations, just include dependency in the ```build.gradle``` file. That's it.
 
UiAutomator can be used as a separate solution for Android test automation. 
A lot of [comparisons][a_ui_frameworks] of UI testing frameworks for Android apps can be found in the Internet. 

But, as for me, it's better to mix Espresso and UiAutomator together. You'll be able to automate any (even end-to-end) test flow using this combination.

UiAutomator exposes such [APIs][api]:

* ```UiCollection```: Enumerates a container's UI elements for the purpose of counting, or targeting sub-elements by their visible text or content-description property
* ```UiObject```: Represents a UI element that is visible on the device
* ```UiScrollable```: Provides support for searching for items in a scrollable UI container
* ```UiSelector```: Represents a query for one or more target UI elements on a device
* ```Configurator```: Allows you to set key parameters for running UI Automator tests

It has a set of helpful waits e.g. ```device.wait(Until.gone(By.res("android:id/aerr_mute")), findElementTimeout)```, 
conditions and verification. And even more:

you can execute shell script simply with

{% highlight kotlin %}
    val device = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation())
    device.executeShellCommand("adb shell settings put global airplane_mode_radios cell,nfc,wimax,bluetooth")
{% endhighlight %}

perform navigation using ```device.pressBack()```, open quick settings shade ```device.openQuickSettings()``` and lots of other features.

And, of course, an example. Here's a test class:

{% highlight kotlin %}
//imports

@RunWith(AndroidJUnit4::class)
class UploadNotificationTest : BaseTest() {

    private lateinit var uiDevice: UiDevice
    
    @get:Rule
    val falconSpoonRule = FalconSpoonRule()
    
    @Before
    fun setup() {
    //create UiDevice instance
    uiDevice = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation())
    
    //start main activity
    launchMainActivity()
    }
    
    @Test
    fun uploadNotificationTest() {
    //open menu with Espresso
        onView(withContentDescription("Open navigation drawer")).perform(click())
    //go to specific option with Espresso   
        navigateToOption(2)
    //click button with Espresso    
        onView(withText(R.string.upload_btn)).perform(click())
    //open notification shade with UiAutomator    
        uiDevice.openNotification()
    //verify notification with UiAutomator      
        verifyNotification(uiDevice)
    //close notification shade with UiAutomator      
        uiDevice.pressBack()
    }
    
    private fun navigateToOption(position: Int) {
        onView(withId(R.id.design_navigation_view))
            .perform(
                RecyclerViewActions.actionOnItemAtPosition<RecyclerView.ViewHolder>(
                    position,
                    RecyclerViewChildActions.actionOnChild(
                        click(),
                        R.id.design_menu_item_text
                    )
                )
            )
    }
    
    private fun verifyNotification(uiDevice: UiDevice) {
        uiDevice.findObject(UiSelector().text("TITLE")).exists()
        uiDevice.findObject(UiSelector().text("Uploading progress...")).exists()
    }
    
}
{% endhighlight %}

And inside ```BaseTest``` class:

{% highlight kotlin %}
//imports

open class BaseTest {

    fun launchMainActivity() {
        ActivityScenario.launch(MainActivity::class.java)
        //additional logic if needed
    }
    
    //rest of BaseTest class functions
}
{% endhighlight %}

As you see it is very easy to mix the both frameworks in one test script.

Another use case for UiAutomator can be enabling of 'offline mode' on the device. Actually, 
it's pretty the same as interacting with notification shade but here we toggle Airplane Mode:

{% highlight kotlin %}
    fun goOffline(uiDevice: UiDevice) {
        uiDevice.openNotification()
        uiDevice.findObject(UiSelector().description("Airplane mode")).click()
        uiDevice.pressBack()
    }
{% endhighlight %}

The same can be implemented with Espresso as well, you can check [here][espresso_offline_mode] or via such wrappers as [TestButler][testbutler] and even via adb command executed in runtime.


#### Summary

One framework is not always enough to achieve some goals and meet the requirements. 
And it's great that Android native tools allow us to be very flexible and choose what to use. Or even to use all at once.



[ui_a]: https://developer.android.com/training/testing/ui-automator
[a_ui_frameworks]: https://proandroiddev.com/android-ui-testing-frameworks-b0b52187ceb
[api]: https://developer.android.com/training/testing/ui-automator#ui-automator-apis
[espresso_offline_mode]: https://letsno.wordpress.com/2016/03/29/automating-offline-scenarios-using-android-espresso/
[testbutler]: https://github.com/linkedin/test-butler