---
layout: post
title: "Writing Better BDD Stories"
comments: true
permalink: writing-better-bdd-stories
---

[BDD](http://en.wikipedia.org/wiki/Behavior-driven_development) (behavior driven development) empowers collaboration between developers, testers, analysts, project managers and stakeholders. As a result, these parties are able to correctly define features' requirements, evaluate their business benefit and set acceptance criteria for quality. 


Story in BDD can describe such aspects in a semi-formal language which is well-understood by all parties.


BDD practitioners may tend to underestimate the importance of writing high-quality stories. They rush too soon to explore BDD frameworks in the market, while they miss the essences of this methodology. Others are not even familiar with the abilities of BDD framework and tools, and what they have to offer for expressing better stories. It may eventually harm the implementation of methodology and may result, for example, in improper definition of requirements. This is why I think it's important to put extra effort into this process and produce better stories.


Below are some general tips, concepts and personal insights which I would like to share. They may be helpful in your next iteration of story writing or refactoring.

__Note:__ Since most [BDD frameworks](http://java.dzone.com/articles/brief-comparison-bdd) today support [Gherkin](http://docs.behat.org/en/latest/guides/1.gherkin.html) syntax (Given, When, Then) to express stories, we'll be adapting it too in this article. Though, many sections are also applicable for other story syntax.

![People talking BDD?](/assets/bdd-discuss.jpeg)

## Don't tie stories to technical details
Stories should not get into technical details such as technologies, algorithms, programming languages and architectures used in software.

For example: in web-based stories, you should definitely not use CSS selectors or HTML tags in steps. In stories with database operations, you should not explicitly describe the SQL queries or tables involved.
You should hide these technical details in steps' implementation. Scenarios should be described in high-level and understood by non-technical personnel. 

A very technical story:

	Given the user is on main page
	When he clicks on button with CSS selector "div ol>li p" 
	Then latest 20 entries from table "tbl.products" should be selected
	
A non-technical and collaborative story:

	Given the user is on main page
	When he asks for latest products
	Then latest 20 products should be displayed

## Avoid dependency between scenarios
Scenarios should be self-contained. Dependency between sequential scenarios may lead to readability and maintenance issues. 

__Remember!__ stories are read and reviewed by other team members or stakeholders, thus they are subject to changes in the future. It may not be clear to other people that scenarios are dependent.

You can test your stories' dependency by changing the order of scenarios and verify they still function as before. 

## Use tabular representation for complex inputs
Scenarios may include multiple parameters or complex data which may not fit into a single step properly. Expressing such parameters in a tabular format makes your scenarios much readable and isolates the scenario's steps from their actual input.

Before:

	Given the user is in products section
	When he selects the products: 1GB RAM (quantity=1), HDMI Cable (quantity=3), Rechargeable Batteries (quantity=2)
	And he adds them to shopping cart
	Then shopping cart should be updated

After:

	Given the user is in products section
	When he selects the following products:
	|Product|Quantity| 
	|1GB RAM|1|
	|HDMI Cable|3| 
	|Rechargeable Batteries|2|
	And he adds them to shopping cart
	Then shopping cart should be updated

## Use meta parameters/tags
When the number of stories increases over time, there is a need to manage all this amount of information and textual content. Using meta parameters or tags (annotated by __@__ character), you can organize stories based on different criteria and label them under categories, both at story and scenario level.

	Scenario: Login to system with permissions for users management
	@category permission 
	@type sanity  
	Given I am on login page
	When I login with administrator credentials
	Then "User Management" section should be accessible

Most BDD frameworks can filter stories or scenarios based on meta parameters, allowing you to skip irrelevant scenarios in current context.

## Don't overuse or misuse GivenStories
GivenStories are reusable stories (more like a set of steps) used as prerequisites for more specific stories. Steps defined in GivenStories are called before their associated story or scenario is. 

GivenStories can be useful when you have complex scenarios which include a set of precondition steps. Such steps can be isolated into a new story, and then can be invoked from other stories or scenarios using the GivenStories keyword. It greatly improves the maintainability of stories and empowers the reuse of common precondition steps (DRY).

However, many story writers tend to overuse this feature. Ending up with numerous GivenStories definitions in a story file may affect the readability of the scenarios and make them hard to follow.

	!-- A GivenStory --!
	precondition1:
	Given ...
	And ...
	And ...
	
	!-- A precondition to entire story --!
	GivenStories: precondition1
	
	Scenario: Example of scenario with precondition as GivenStories 
	!-- A precondition to scenario --!   
	GivenStories: precondition2,setupEnvironment
	Given ...
	
	Scenario: Another example of scenario with precondition as GivenStories
	!-- A precondition to scenario --!       
	GivenStories: precondition3,setupEnvironment
	Given ...

Moreover, many BDD frameworks don't support GivenStories in story syntax (Cucumber has even [deprecated](http://blog.josephwilk.net/ruby/cucumber-waves-goodbye-to-givenscenario.html) it).

__Note:__ GivenStories should not be used as a setup for each scenario in story. You should use [Backgrounds](http://docs.behat.org/en/latest/guides/1.gherkin.html#backgrounds) for this purpose.

## Use "And"s as steps
Avoid declaring multiple "And"s which belong to the same step. Instead, split them into several steps and use them to compose the requested scenario.
For example, let's break down this scenario:

Before:

	Scenario: Remove a product from cart
	Given the user is on main page and he is logged in and shopping cart is not empty
	......
	
After:

	Scenario: Remove a product from cart
	Given the user is on main page
	And he is logged in
	And shopping cart is not empty
	.....
	
## Break down complex steps implementation
 Don't let steps in scenarios spread over and cover too much functionality. Make sure to break down steps with complex implementation into several smaller steps. You should define steps which fulfill a very specific functionality in system. Such steps can be reusable in other scenarios and are less hard to maintain.  
 As a rule of thumb, step implementation should not include more than few lines of codes.

## Less "How", more "What" (Imperative vs Declarative)
A story should not emphasis "How" events occur or outcomes are produced. Instead, it should describe "What" does this event do or this outcome produces. 
Let's take a look at following scenario in two different representations:

Before:

	Scenario: Successful user login
	Given the user is in the login page
	When he fills username text field with "sammy"
	And he fills password text field with "mypass"
	And he fills passphrase text field with "passphrase1" 
	And he clicks on "Login" button
	Then a new message should appear with text "Login was successful"
	And a new link should appear with text "Welcome Sammy!"

After:

	Scenario: Successful user login
	Given the user is in the login page
	And he has valid login information
	When he logs in
	Then he should be successfully authorized in system

The new scenario describes what event is being performed (login), while the original scenario is composed of [UI steps](http://aslakhellesoy.com/post/11055981222/the-training-wheels-came-off) describing how login is performed. In the modified scenario, we clearly described what is the desired outcome, while in original story we described how this outcome is verified in details. 

__Remember!__ clients and stake holders think and talk in higher abstractions, so does your scenarios should be. 

In other terminology, the original and new scenarios are imperative and declarative scenarios, accordingly. The imperative scenario is long, very detailed and closely tied to UI (which may require modification of scenario if the UI changes). 
The declarative scenario is robust to changes, less "noisy", more collaborative and achieves same goal in fewer lines.

## Combine steps
Following the previous two tips, we somehow may have a conflict. We want to keep the step implementation as specific and low-level as possible (code maintenance, re-usability), but in the same time we want to compose declarative scenarios and describe them in high-level steps (readability, collaboration).

We can solve this conflict by grouping small steps into one composite step.
For example, we can create a composite step for defining the login procedure, based on much smaller steps.

	@When("user logs in")
	@Composite(steps = {
	When he fills username text field with "sammy"
	When he fills password text field with "mypass"
	When he fills passphrase text field with "passphrase1" 
	When he clicks on "Login" button
	}

Composite steps do not include any implementation, as they only define which smaller steps they contain. Above example is used to define composite steps in [JBehave BDD framework](http://jbehave.org). Other frameworks may require different syntax for achieving the same.

Notice how by combining small steps into one composite step, we insure that stories are collaborative and maintainable at the same time.

## Settle on the language
Prior to writing stories, you should define various consistencies, standards and ground rules related to the business language you are going to "speak". Defining language's terms, context, users' roles and stake holders __beforehand__ will insure an ubiquitous language and collaborative stories which can be well-understood among both technical and non-technical members in your domain.

Such process should be conducted with other team members (from different roles), and may require several revisions until the language is well-defined and agreed-on.
	
