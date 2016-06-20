---
layout: post
title: "WebElement Waits"
comments: true
---

Selenium Webdriver 2.0 API comes with a set of utility classes for applying different wait conditions when locating DOM elements or when waiting for an event to happen in DOM.
API provides WebDriverWait and ExpectedCondition classes with a built-in busy-wait-polling machnism, which can be customized via different parameters such as wait timeout, interval between polls and ignored exceptions. It also provides ExpectedConditions helper class which includes many ready-made wait conditions used commonly in WebDriver tests. For example:
However, I had a scenario where I wanted to apply an expected condition on a WebElement instance, not on a WebDriver instance. Since WebElement is also a sub interface of SearchContext, it seemed abvious that API will support WebElement waits. But sadly, current implementation doesn't provide such support.

Since WebDriverWait is based on a generic wait object FluentWait, implementing a WebElementWait should be straight forward.

WebElementWait subclasses FluentWait<T> and defines WebElement class as its input type:

{% highlight java %}
public class WebElementWait extends FluentWait<WebElement> {

    public WebElementWait(WebElement element, long timeOutInSeconds) {
        super(element);
        withTimeout(timeOutInSeconds, TimeUnit.SECONDS);
    }

}
{% endhighlight %}

Once WebElementWait is ready, we can implement WebElement-specific ExpectedConditions.

First, we define a new interface for WebElement ExpectedConditions

{% highlight java %}
public interface WebElementExpectedCondition<T> extends Function<WebElement, T> {}
{% endhighlight %}

Finally, we can implement our desired ExpectedConditions based on new interface. For example:

{% highlight java %}
/**
 * An expectation for checking that an element is present on the DOM of a
 * page. This does not necessarily mean that the element is visible.
 *
 * @param locator used to find the element
 * @return the WebElement once it is located
 */
public static WebElementExpectedCondition<WebElement> presenceOfElementLocated(final By locator) {
    
    return new WebElementExpectedCondition<WebElement>() {
        
        @Override
        public WebElement apply(WebElement element) {
            try {
                WebElement innerElement = element.findElement(locator);
                return innerElement;
            } catch (StaleElementReferenceException e) {
                return null;
            }
        }

        @Override
        public String toString() {
            return "presence of element located by: " + locator;
        }
    };
}
{% endhighlight %}
