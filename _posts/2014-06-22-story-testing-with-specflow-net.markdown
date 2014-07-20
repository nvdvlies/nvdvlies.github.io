---
layout: post
title:  "Story Testing with SpecFlow.NET"
date:   2014-06-22 12:00:00
---

User stories are a great way to collaborate between developers, business analysts, quality assurance and other stakeholders, because everyone is able to read and/or write one.
[SpecFlow.NET](http://www.specflow.org/) is a tool that allows you to integrate those stories easily into your test framework.
A user describes scenarios on how to test a feature in plain text using the [Gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin) language. A scenario in Gherkin consists of a Given (the precondition), When (the action), and Then (the observable result) structure.

<!--more-->

### An example

{% highlight gherkin %}
Feature: Sum two numbers
     In order to avoid silly mistakes
     As a math idiot
     I want to be told the sum of two numbers

Scenario: Sum two numbers successfully
     Given I am on the calculator page
     And I have entered "50" into the first calculator field
     And I have entered "70" into the second calculator field
     When I press add
     Then the result should be "120" on the screen
{% endhighlight %}

To test this scenario in Visual Studio 2012, besides SpecFlow, I'm using NUnit as the underlying test framework and Selenium WebDriver in order to automate the web browser. 

### Visual Studio IDE integration
1. Download and install [NUnit Test Adapter](http://visualstudiogallery.msdn.microsoft.com/6ab922d0-21c0-4f06-ab5f-4ecd1fe7175d).
2. Download and install [Specflow IDE](http://visualstudiogallery.msdn.microsoft.com/9915524d-7fb0-43c3-bb3c-a8a14fbd40ee).

### Create a new project
1. Open Visual Studio 2012
2. In the menu-bar choose File | New Project.
3. Choose a project of type "Class Library" and provide a project name.

### Add NuGet packages
1. Right-click on the project in the Solution Explorer 
2. Choose "Manage NuGet packages".
3. Search and install the packages defined below:
     - SpecFlow.NUnit
     - Selenium Webdriver
     - Selenium WebDriver Support Classes
     - Selenium.Webdriver.ChromeDriver

### Add a new SpecFlow feature
1. Right-click on the project in the Solution Explorer
2. In the menu choose Add | New Item...
3. Choose "SpecFlow Feature File"

Replace the default content of the newly created feature with the user story shown earlier in this post. The default content should be almost identical, but mine has some minor differences regarding parameters.

### Create a test class
1. Right-click on the project in the Solution Explorer
2. In the menu choose Add | Class... (Yes, this could be a normal class)
3. Name the new class "Sum_two_numbers".
4. Add the code listed below:

{% highlight C# %}
using TechTalk.SpecFlow;
using NUnit.Framework;

namespace Calculator.Tests {
    [Binding]
    public class Sum_two_numbers {
        private CalculatorPage _calculatorPage;

        [Given(@"I am on the calculator page")]
        public void GivenUserIsOnTheCalculatorPage() {
            _calculatorPage = new CalculatorPage(Environment.WebDriver);
        }

        [Given(@"I have entered ""(.*)"" into the first calculator field")]
        public void GivenUserEntersValueInFirstNumberField(string value) {
            _calculatorPage.firstNumberField.SendKeys(value);
        }

        [Given(@"I have entered ""(.*)"" into the second calculator field")]
        public void GivenUserEntersValueInSecondNumberField(string value) {
            _calculatorPage.secondNumberField.SendKeys(value);
        }

        [When(@"I press add")]
        public void WhenUserPressesAddButton() {
            _calculatorPage.sumButton.Click();
        }

        [Then(@"the result should be ""(.*)"" on the screen")]
        public void ThenCalculatorResultFieldContains(string value) {
            Assert.AreEqual(value, _calculatorPage.resultField.Text);
        }
    }
}
{% endhighlight %}

Notice the class contains a method for every line in the scenario with an attribute to link the method to the correct scenario line. You can use a regular expression to pass in parameters from the scenario to the method. 

### Create a Page Object helper class
With Selenium you can simulate a user browsing your website. It lets you navigate to a certain page and perform actions like typing text into an inputfield and click buttons. I would advise to create a separate class which deals with the UI elements, so it can be reused in different tests. 

1. Right-click on the project in the Solution Explorer
2. In the menu choose Add | Class...
3. Name the new class "CalculatorPage".
4. Add the code listed below:

{% highlight C# %}
using OpenQA.Selenium;
using OpenQA.Selenium.Support.PageObjects;

namespace Calculator.Tests {
    public class CalculatorPage {
        public CalculatorPage(IWebDriver driver) {
            driver.Navigate().GoToUrl("http://localhost:5600/calculator/");
            PageFactory.InitElements(driver, this);
        }

        [FindsBy(How = How.Id, Using = "firstNumber")]
        public IWebElement firstNumberField { get; set; }

        [FindsBy(How = How.Id, Using = "secondNumber")]
        public IWebElement secondNumberField { get; set; }
       
        [FindsBy(How = How.Id, Using = "sum")]
        public IWebElement sumButton { get; set; }

        [FindsBy(How = How.Id, Using = "result")]
        public IWebElement resultField { get; set; }
    }
}
{% endhighlight %}

With the FindsBy attribute used above you can easily resolve an element on the page. We can now perform actions on these elements within our tests.

### Create a WebDriver helper class
The class will provide global access to the Selenium WebDriver.

1. Right-click on the project in the Solution Explorer
2. In the menu choose Add | Class...
3. Name the new class "Environment".
4. Add the code listed below:

{% highlight C# %}
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using TechTalk.SpecFlow;

namespace Calculator.Tests {
    [Binding]
    public class Environment {
        private static ChromeDriver webDriver;

        /// <summary>
        /// Selenium webdriver singleton
        /// </summary>
        public static IWebDriver WebDriver {
            get {
                return webDriver ?? (webDriver = new ChromeDriver());
            }
        }

        /// <summary>
        /// Automation logic that has to run after the entire test suite is completed.
        /// </summary>
        [AfterTestRun]
        public static void AfterTestRun() {
            WebDriver.Close();

            WebDriver.Quit();
            webDriver = null;
        }
    }
}
{% endhighlight %}

### Run test
1. In the menu-bar choose Test | Windows | Test Explorer
2. Click "run all" in the Test Explorer to compile and execute the scenario(s).

![](/assets/posts/2014/06/run_tests.png)

A scenario will turn green when the test passes and red when it doesn't.
