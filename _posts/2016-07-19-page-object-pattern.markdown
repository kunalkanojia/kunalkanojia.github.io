---
layout: page
title:  "Page object pattern"
quote: Within your web app's UI there are areas that your tests interact with. A Page Object simply models these as objects within the test code. This reduces the amount of duplicated code and means that if the UI changes, the fix need only be applied in one place.
comments: true
---

We all love tests and with TDD comes a lot of it.
 
Integration tests have also become part of development cycle with adoption of BDD and also due the simplicity with which they can be implemented.

For example with scala and play we write a lot of tests like this  - 

{% highlight scala %}
class IntegrationSpec extends PlaySpec with ... {
  "User" should {
    "be able to sign-up in the application" in {
       //Act
       browser.goTo("/signUp")
       browser.find("input#firstName").text("first")
       browser.find("input#lastName").text("last")
       browser.find("input#email").text("email@domain.com")
       browser.find("input#password").text("password")
       browser.find("#signup_form").submit
       //Assert
       browser.pageSource must contain("Welcome, first last!")
    }
  }
}
{% endhighlight %}

But I really find the above code difficult to read. Worse is when you start testing long interactions and its difficult to distinguish between the Arrange, Act and Assert part of the test.

And then you have browser interactions like below. Looking at which you are sure its something you should not touch.

{% highlight scala %}
//This is real, I didn't make it up.
browser.$("#report_summary > .row > div + div + div + div .data-display").getText mustEqual 100
{% endhighlight %}

At my current firm we follow TDD so we had lots of such integrations tests with WebDriver API's. It was increasingly becoming difficult to maintain and modify as the UI would change frequently.

We did a lot of things to improve it. Added common helper like login and signup methods to the browser. But then it was the helper class that become too long and difficult to read.

Then we decided to start using the page object pattern and the above test code was changed to this - 

{% highlight scala %}
class IntegrationSpec extends PlaySpec with ... {
  "User" should {
    "be able to sign-up in the application" in {
      // Act
      signUpPage.open()
      signUpPage.signUp(
        firstName = "first",
        lastName = "last",
        companyEmail = "email@example.com",
        password = "pass"
      )
      //Assert
      browser.pageSource must contain("Welcome, first last!")
    }
  }
}
{% endhighlight %}


{% highlight scala %}
//Sign-up page object
class SignUpPage(implicit val browser: TestBrowser) extends Page {
  def open(): Unit = browser.goTo(UserController.signUp().url)
  def signUp(
    firstName: String,
    lastName: String,
    email: String,
    password: String,
  ): DashboardPage = {
    browser.$("input#firstName").text(firstName)
    browser.$("input#lastName").text(lastName)
    browser.$("input#email").text(companyEmail)
    browser.$("#signup_form").submit
    
    new DashboardPage
  }
}
{% endhighlight %}

As you can see from above, we stopped manipulating the HTML in the tests. We started invoking application API's and moved the HTML logic to the page object.

The page object wraps an HTML page, or fragment, with an application-specific API, allowing you to manipulate page elements without digging around in the HTML.

Page objects are a good example of encapsulation - they hide the details of the UI structure from the tests.

Some useful rules to follow when using this pattern: 

  -  Page objects should provide an interface that's easy to program to and hides the underlying HTML.
  -  To access a text field you should have accessor methods that take and return a string, check boxes should use booleans, and buttons should be represented by action oriented method names.
  -  Page objects should'nt be built for each page but rather for significant sections for the page. Ex. HeaderPage, FooterPage. ie. Build page for the structure that makes sense for user using the application.
  -  Try to follow single responsibility principle and don't make assertions in the Page objects. Example - Instead of `signUpPage.assertTitle("Signup to this funky app")` in the page object. Our test should assert `signUpPage.getTitle mustEqual "Signup to this funky app"`
  - Different results for the same action are modelled as different methods. Example - Signup Page can have methods like `signUp(...)` which returns dashboard and `signUpExpectingFailure(...)` which takes user back to sign-up page.

  

If unlike me you are using ScalaTest instead of Specs2. Then you can use the Page trait provided by ScalaTest and write cool stuff like below - 

{% highlight scala %}
class HomePage extends Page {
  val url = "localhost:9000/index.html"
}

val homePage = new HomePage
go to homePage
{% endhighlight %} 


## Conclusion

In conclusion, using page object pattern makes our tests more robust to changes in the UI.

And when your company employs a new UX guy to revamp your UI you know exactly which part of integration tests will break.


## References

[https://github.com/SeleniumHQ/selenium/wiki/PageObjects](https://github.com/SeleniumHQ/selenium/wiki/PageObjects)

[http://www.scalatest.org/user_guide/using_selenium#pageObjects](http://www.scalatest.org/user_guide/using_selenium#pageObjects)


