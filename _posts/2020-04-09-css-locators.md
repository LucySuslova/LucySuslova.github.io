---
layout: post
title: "element.getText() omitting child nodes "
subtitle: "js. selenide"
date: 2020-04-09 09:40:59 +0300

---
#### The problem

Get text ('99.999,99 €') from the element that looks like following:

{% highlight html %}
<p data-test-id="test_element">
      99.999,99 €
      <sup class="ref">
        [1]
      </sup>
      <sup class="ref">
        [2]
      </sup>
</p>
{% endhighlight %}

Nothing special at the first sight.

To do that let's create css selector to find required element ```$$('p[data-test-id=test_element]')```. And let's see its text:

{% highlight javascript %}
$$('p[data-test-id=test_element]').textContent
{% endhighlight %}

Allegedly we'll get '99.999,99 €'. But the result is: 
```
"
      99.999,99 €
      
        [1]
      
        [2]
      "
```

Okay, we need to get rid of the child nodes' text. How to do that?

#### The solution

Simplest way is to use JavaScript. 

{% highlight javascript %}
$$('p[data-test-id=test_element]').childNodes[0].nodeValue
{% endhighlight %}

This command will return exactly what we need:
```
"
      99.999,99 €
      "
```

Now let's use this solution with [Selenide][selenide]:

{% highlight java %}
    private String getPrice() {
        return executeJavaScript("return arguments[0].childNodes[0].nodeValue;", $("p[data-test-id=test_element]"));
    }
{% endhighlight %}

Simple and useful :)


[selenide]: https://github.com/selenide/selenide