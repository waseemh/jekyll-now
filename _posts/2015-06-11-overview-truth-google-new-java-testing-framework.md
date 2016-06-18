---
layout: post
title: "An overview of 'Truth', Google's new Java testing framework"
comments: true
permalink: overview-truth-google-new-testing-framework
---
Although still considered a very new project, it was interesting to see what does [Truth](https://google.github.io/truth/), a new testing framework by Google, pioneer in the world of test automation.

'Truth' is a Java testing framework with an aim to make test assertions and failures more readable.
It is not a fully-featured testing framework, but it can be integrated with any type of test and with any testing framework.

Truth comes with two major features: __Fluent Assertions API__ and __Failure Strategies__

## Fluent Assertions API

Truth's assertion API allows you to write test assertions in a fluent style.

The way assertions are composed allow other users to easily understand what the test is trying to assert.
As a demonstration, let's compare good ol' JUnit assertions with Truth assertions:

{% highlight java %}
//Basic assertion:
assertTrue(user.isConnected()); //JUnit 
ASSERT.that(user.isConnected()).isTrue(); //Truth

//Collection assertion:
assertTrue(collectionA.contains(q)); //JUnit 
ASSERT.that(collectionA).contains(q); //Truth

//Map assertion:
assertEquals("John Doe", userMap.get(id)); //JUnit 
assertThat(userMap).containsEntry(id, "John Doe");//Truth
{% endhighlight %}

Additionally, API provides advanced assertions which are not available in JUnit:
{% highlight java %}
//Composite propositions:
ASSERT.that(collectionA).containsExactly(a, b, c, d).inOrder();

//Iterative propositions
Set<String> fruits = asList("Apple","Orange","Pomegranate");
ASSERT.in(fruits).thatEach(STRING).endsWith("e");
{% endhighlight %}

As you can see, the fluent API allows you to compose complex, yet readable propositions.

For a [full list](http://google.github.io/truth/usage/#built-in-propositions) of built-in propositions.

When compared with JUnit+Hamcrest assertions, it appears that Truth's fluent API is slightly better:

{% highlight java %}
Assert.assertThat(0, is(lessThan(1))); //JUnit+Hamcrest
ASSERT.that(0).isLessThan(1); //Truth
{% endhighlight %}

But when compared with other advanced assertion Java open source libraries such as [AssertJ](http://joel-costigliola.github.io/assertj/) or [FEST](https://code.google.com/p/fest/), Truth's fluent API doesn't provide any significant enhancements.

## Failure Strategies

Truth has expanded test assertion beyond its traditional definition. An assertion in Truth is a __'test verb'__ which asserts on a __'subject'__ (a subject is the object under test, i.e: a collection, an Integer, a User object).
Other test verbs provided by Truth are __'Assumption'__ and __'Expectation'__. 

Each of these verbs defines how a test will behave upon failure, AKA __'Failure Strategy'__:

  - __Assertion failure strategy__: the normal behavior you would expect from an assertion - fail the test and report error.
  - __Assumption failure strategy__: skip test upon failure without reporting an error.
  - __Expectation failure strategy__: continue to run the test upon failure, but report an error when test is completed.

You can also implement your own failure strategy and use it in a test verb (more on that later).

In order to switch to a different failure strategy (ASSERT is default), you can use ASSUME or EXPECT (as JUnit rule) when writing assertions. For example:

{% highlight java %}
//Assumption:
ASSUME.that(4).isLessThan(10);

//Expectation (must be first declared as a JUnit Rule using Expect.create()):
@Rule public final Expect EXPECT = Expect.create();
EXPECT.that("apple").contains("r");
EXPECT.that("apple").contains("c");
EXPECT.that("apple").contains("f");
{% endhighlight %}

## Customizable Failure Messages

If an assertion error message isn't clear enough or too general, you can override it using withFailureMessage(message). Method invocation should be appended to the assertion with an appropriate error message. For example, if you want to assert that a list is empty but still display a specific message in case of error:

{% highlight java %}
List<User> userList = userRepository.getUsersBelowAge(age);
ASSERT.withFailureMessage("Repository should not contain any registered users with age below " + age).that(userList).isEmpty();
{% endhighlight %}

## Extensible Assertions

Truth is a very extensible framework as it provides an abstraction of test assertion.
Assertions are very modular in Truth -  you can define your own domain-specific assertion API by implementing custom test verbs, subjects and failure strategies. 


Creating a custom test verb by extending TestVerb class:
{% highlight java %}
public class MyVerb extends TestVerb {
    //create test verb as a static variable with 'throw assertion error' failure strategy
    public static final MyVerb MY_VERB_ASSERT = new MyVerb(Truth.THROW_ASSERTION_ERROR);

    public MyVerb(FailureStrategy failureStrategy) {
        super(failureStrategy);
    }
}
{% endhighlight %}

Creating a custom subject by extending Subject class:

{% highlight java %}
public class UserSubject extends Subject<UserSubject, User> {

    public UserSubject(FailureStrategy failureStrategy, User subject) {
        super(failureStrategy, subject);
    }

    @Override public void isEqualTo(User other) {
        if (getSubject().getName() != other.getName()) {
            fail("is equal to", getSubject(), other);
        }
    }
}
{% endhighlight %}

Use custom verb and subject in a test:

{% highlight java %}
@Test public void userTypeProposition() {
    MY_VERB_ASSERT.that(new User("John","Doe")).isEqualTo(new User("Jane","Doe"));
}
{% endhighlight %}

## Summary

Truth has a solid foundation given its highly extensible design and abstract definitions of an assertion (verb, subject, propositions, failure strategy).
But other open source testing frameworks provide similar fluent assertion APIs, so Truth doesn't bring any added value in this area.
However since it's still an alpha project, major improvements and new features might appear on upcoming releases.
