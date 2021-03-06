---
layout: post
title:  "kind of fluent wait for robot framework "
subtitle: "to make sure element is not visible"
date:   2020-01-03 09:40:39 +0300
header-img:  "img/posts/01.jpg"

---

[Robot Framework][rf] provides native solution how to wait for element to be visible and vice versa. But using such keywords as
```Wait Until Element Is Not Visible``` or  ```Wait Until Element Is Visible``` with timeout often fails with stale element exception.
    And I try not to use these keywords in my test scripts to avoid possible undesirable flakiness.



Often enough I need to know when the loader element has already disappeared and I can continue to perform further actions and verifications.
    In such cases I use a kind of ```Wait Until Element Is Not Visible``` workaround.

{% highlight python %}
        *** Keyword ***
        Wait For Loader To Disappear
        :FOR    ${i}    IN RANGE    15
        \    Sleep  1
        \    ${present}=  Run Keyword And Return Status    Element Should Be Visible    ${loaderId}
        \    Exit For Loop If    ${present}==False
        \    log  Wait for loader to disappear ${i} sec     console=True
 {% endhighlight %}


This keyword can be modified a little bit to accept '15' or any other value - seconds to wait - as a parameter:

{% highlight python %}
        *** Keyword ***
        Wait For Loader To Disappear
        [Arguments]    ${seconds}
        :FOR    ${i}    IN RANGE    ${seconds}
        \    Sleep  1
        \    ${present}=  Run Keyword And Return Status    Element Should Be Visible    ${loaderId}
        \    Exit For Loop If    ${present}==False
        \    log  Wait for loader to disappear ${i} sec     console=True
 {% endhighlight %}
 
[rf]: https://robotframework.org/
