---
layout: page
title:  "Integration testing with page object pattern"
quote: Reading big integration tests full of WebDriver API's is a nightmare.
comments: true
---

We all love tests and with TDD comes a lot of it. Integration tests have also become part of development cycle with adoption of BDD and also due the simplicity with which they can be implemented.

For example in play you will normally see a lot of integration tests like one below - 

{% highlight scala %}
class IntegrationSpec extends PlaySpec with OneServerPerTest with OneBrowserPerTest with HtmlUnitFactory {
  "Application" should {
    "work from within a browser" in {
      go to ("http://localhost:" + port)
      pageSource must include ("Create user and get started.")
    }
  }
}
{% endhighlight %}