---
id: 348
title: Writing BDD tests with Cucumber JVM
date: 2015-12-04T00:00:05+00:00
author: lakshmihm
layout: post
guid: http://www.javaadvent.com/?p=348
permalink: /2015/12/writing-bdd-tests-with-cucumber-jvm.html
dsq_thread_id:
  - 4962579373
categories:
  - 2015
  - Uncategorized
---
<a href="https://cucumber.io/docs/reference/jvm">Cucumber JVM</a> as an excellent tool to write your BDD tests.In this article I would like to give an introduction to BDD with Cucumber JVM.

Let's get started...

<strong>What is BDD? </strong>

<a href="http://www.javaadvent.com/content/uploads/2015/12/problems.png"><img class="alignnone wp-image-350" src="http://www.javaadvent.com/content/uploads/2015/12/problems-300x220.png" alt="problems" width="421" height="309" /></a>

In a nutshell, BDD tries to solve the problem of "understanding requirements with examples"
<img class="alignnone wp-image-349" src="http://www.javaadvent.com/content/uploads/2015/12/bdd-300x186.png" alt="bdd" width="447" height="277" />

<strong>BDD tools</strong>
There are lot of tools available for BDD and interestingly you can find quite a few vegetable names in the list ;) Cucumber,Spinach, Lettuce, JBehave, Twist etc. Out of these Cucumber is simple and easy to use.

<strong>Cucumber-JVM</strong>
Cucumber is written in Ruby and <strong>Cucumber JVM</strong> is an implementation of cucumber for the popular JVM languages like Java, Scala, Groovy, Clojure etc

<strong>Cucumber Stack</strong>
<a href="http://www.javaadvent.com/content/uploads/2015/12/stack.png"><img class="alignnone size-medium wp-image-353" src="http://www.javaadvent.com/content/uploads/2015/12/stack-191x300.png" alt="stack" width="191" height="300" /></a>
We write features and scenarios in a "Ubiquitous" Language and then implement them with the step definitions and support code.

<strong>Feature file and Gherkin</strong>
You first begin by writing a .feature file.A feature file conventionally starts with the <strong>Feature</strong> keyword followed by <strong>Scenario</strong>. Each scenario consists of multiple steps. Cucumber uses Gherkin for this. Gherkin is a Business Readable, Domain Specific Language that lets you describe software’s behavior without detailing how that behavior is implemented.
Example:
<pre>Feature: Placing bets	 	 
 Scenario: Place a bet with cash balance	 	 
 Given I have an account with cash balance of 100	 	 
 When I place a bet of 10 on "SB_PRE_MATCH"	 	 
 Then the bet should be placed successfully	 	 
 And the remaining balance in my account should be 90</pre>
As you can see the feature file is more like a spoken language with gherkin <strong>keywords</strong> like Feature, Scenario, Given,When, Then,And ,But, #(for comments).

<strong>Step Definitions</strong>
Once you have finalized the feature file with different scenarios, the next stage is to give life to the scenarios by writing your step definitions. Cucumber uses regular expression to map the steps with the actual step definitions. Step definitions can be written in the JVM language of your choice. The keywords are ignored while mapping the step definitions.
So in reference to the above example feature we will have to write step definition for all the four steps. Use the IDE plugin to generate the stub for you.
<pre>import cucumber.api.java.en.And;	 	 
import cucumber.api.java.en.Given;	 	 
import cucumber.api.java.en.Then;	 	 
import cucumber.api.java.en.When;	 	 
public class PlaceBetStepDefs {	 	 
 @Given("^I have an account with cash balance of (\\d+) $")	 	 
 public void accountWithBalance(int balance) throws Throwable {	 	 
 // Write code here that turns the phrase above into concrete actions	 	 
 //throw new PendingException();	 	 
 }	 	 
 @When("^I place a bet of (\\d+) on \"(.*?)\"$")	 	 
 public void placeBet(int stake, String product) throws Throwable {	 	 
 // Write code here that turns the phrase above into concrete actions	 	 
 // throw new PendingException();	 	 
 }	 	 
 @Then("^the bet should be placed successfully$")	 	 
 public void theBetShouldBePlacedSuccessfully() throws Throwable {	 	 
 // Write code here that turns the phrase above into concrete actions	 	 
 //throw new PendingException();	 	 
 }	 	 
 @And("^the remaining balance in my account should be (\\d+)$")	 	 
 public void assertRemainingBalance(int remaining) throws Throwable {	 	 
 // Write code here that turns the phrase above into concrete actions	 	 
 //throw new PendingException();	 	 
 }	 	 
}</pre>
<strong>Support Code</strong>
The next step is to back your step definitions with support code. You can for example do a REST call to execute the step or do a database call or use a web driver like selenium . It is entirely up to the implementation. Once you get the response you can assert it with the results you are expecting or map it to your domain objects.
For example you can you selenium web driver to simulate logging into a site
<pre>protected WebDriver driver;	 	 
@Before("@startbrowser")	 	 
public void setup() {	 	 
 System.setProperty("webdriver.chrome.driver", "C:\\devel\\projects\\cucumberworkshop\\chromedriver.exe");	 	
 driver = new ChromeDriver();	 	 
}	 	 
@Given("^I open google$")	 	 
public void I_open_google() throws Throwable {	 	 
 driver.manage().timeouts().implicitlyWait(5, TimeUnit.SECONDS);	 	 
 driver.get("https://www.google.co.uk");	 	 
}</pre>
<strong>Expressive Scenarios</strong>
Cucumber provides more options to organize your scenarios better.
<ul>
	<li><a href="https://github.com/cucumber/cucumber/wiki/Background">Background</a>- use this to define steps which are common to all scenarios</li>
	<li><a href="http://cucumber.github.io/api/cucumber/jvm/javadoc/cucumber/api/DataTable.html">Data Tables</a>- You can write the input data in table format</li>
	<li><a href="https://github.com/cucumber/cucumber/wiki/Scenario-Outlines">Scenario Outline</a>-placeholder for your scenario which can be executed for a set of data called Example.</li>
	<li><a href="https://github.com/cucumber/cucumber/wiki/Tags">Tags and Sub Folders</a> to organize your features-Tags are more like sticky notes for documentation.</li>
</ul>
<strong>Dependency Injection</strong>
More often than not you might have to pass the information created in one step to another. For example you create a domain object in your first step and then you need to use it in your second step. The clean way to achieve this is through Dependency Injection . Cucumber provides modules for the main DI containers like Spring, Guice, Pico etc.

<strong>Executing Cucumber</strong>
It is very easy to run Cucumber on IntelliJ<a href="https://www.jetbrains.com/idea/help/enabling-cucumber-support-in-project.html"> IDE</a> . It can be also integrated with your build system. You can also control the tests you want to run with different options.

<strong>Reporting Options</strong>
There are lot of plugins available for reporting . For example you could use the <a href="https://github.com/damianszczepanik/cucumber-reporting">Master Thought plugin</a> for the reports.

<strong>References</strong>
<a href="https://pragprog.com/book/srjcuc/the-cucumber-for-java-book">The Cucumber for Java book</a>- This is an excellent book and this is all you need to get you started
<a href="https://cucumber.io/docs/reference/jvm#java">Documentation</a>
<a href="https://github.com/cucumber/cucumber-jvm">GitHub link</a>
That's all folks. Hope you liked it. Have a good Christmas! Enjoy.