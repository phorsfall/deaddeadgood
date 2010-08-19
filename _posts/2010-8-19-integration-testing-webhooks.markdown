---
layout: post
title: Integration Testing Webhooks
permalink: /2010/8/19/integration-testing-webhooks/
---
### The Problem
Testing aspects of your web app that use an API that employs webhooks/callbacks can be awkward, as typically your development environment is not accessible by the API provider. You can expose your local server to the internet by creating an SSH tunnel, but when it comes to testing it's less than ideal if you have to manually start the tunnel, and perhaps perform some config with the API provider before you can run the tests. A lot of the knowledge about the environment required to run the test is kept outside of the test itself, therefore it is not as automated as it could be. If you bother to write the test, you probably won't run it.

### A Solution
The solution to this problem (at least for one project I'm working on) is a few Cucumber steps and a simple Ruby wrapper around the wonderful [localtunnel](http://github.com/progrium/localtunnel) gem, which makes it simple to create throw-away SSH tunnels which are perfect for testing. Localtunnel does _all_ of the heavy lifting here, the Cucumber steps simply start a tunnel once per test run, and in this example login to a Google Checkout sandbox account to configure the callback URL based on the host which is dynamically assigned to you each time you run localtunnel. (You don't _have_ to use Cucumber for this, you could easily adapt this to straight up webrat/capybara or whatever it is you're using.)

First, here's a wrapper around the localtunnel binary that makes it simple to start a new tunnel and fetch the hostname which is forwarded to your machine. (It uses `PTY`, so I imagine it won't work on Windows.) In a Rails app, you could drop this in `lib/local_tunnel.rb`.

{% highlight ruby %}
require 'pty'
require 'singleton'

class LocalTunnel
  include Singleton
  attr_accessor :url, :pid

  def initialize
    @started = false
  end

  def start(port = 8080)
    unless @started
      r, w, pid = PTY.spawn("localtunnel #{port}")
      self.pid = pid

      output = ""
      r.each do |line|
        output << line
        break if line =~ /accessible from (.*) \.\.\./
      end
      self.url = $1

      if url
        @started = true
      else
        raise "Could not start SSH tunnel:\n\n#{output}\n"
      end
    end
  end

  def started?
    @started
  end
end
{% endhighlight %}

With that, writing a Cucumber step to setup the tunnel is trivial:

{% highlight ruby %}
Given /^my test environment is available via an SSH tunnel$/ do
  LocalTunnel.instance.start(9887)
end
{% endhighlight %}

And here's an example of some steps to configure Google Checkout. Clearly this is somewhat app specific, but you get the idea. (I'm setting a flag here so that this is only performed by the first feature that actually needs it. If there's a cleaner way to do this please let me know.)

{% highlight ruby %}
Given /^the Google Checkout Sandbox is configured to callback to my test environment$/ do
  unless google_checkout_sandbox_configured?
    visit "https://sandbox.google.com/checkout/sell/"
    And %q{I login to Google as "me@gmail.com" with password "mypassword"}
    click_link "Settings"
    click_link "Integration"
    fill_in "apiURL", :with => "#{LocalTunnel.instance.url}/google_checkout"
    click_button "Save"
    click_link "Sign out"
    self.google_checkout_sandbox_configured = true
  end
end

When /^I login to Google as "([^\"]*)" with password "([^\"]*)"$/ do |login, password|
  within_frame "login" do
    fill_in 'Email', :with => login
    fill_in 'Passwd', :with => password
    click_button "Sign in and continue"
  end
end

# A flag used to track wether we've configured the Google Sandbox to use the SSL
# tunnel URL across scenarios.
module GoogleCheckoutHelpers
  def google_checkout_sandbox_configured=(value)
    self.class.class_eval { @google_checkout_sandbox_configured = value }
  end

  def google_checkout_sandbox_configured?
    self.class.class_eval { @google_checkout_sandbox_configured }
  end
end

World(GoogleCheckoutHelpers)
{% endhighlight %}

With that in place, you might end up with a scenario that looks a little like this:

    Feature: Paying with Google Checkout

      Background:
        Given my test environment is available via an SSH tunnel
        And the Google Checkout Sandbox is configured to callback to my test environment

      Scenario: Paying with Google Checkout
        Given I have some products in my cart
        When I press "Google Checkout"
        And I login to Google as "customer" with password "password"
        And I press "bottomBuyButton"
        Then I should see "Your order has been sent."
        Given I am logged in as an admin
        And I am on the admin orders page
        And I click view on the 1st order
        And I wait until I see "Current status: Paid"
        Then I should see "Current status: Paid"

This pretty much takes care of the problem as I originally outlined it. The only slight snag is that the first time you use localtunnel you need to pass it the path to your public SSH key. If a developer who hasn't used localtunnel before tries to run our feature, the SSH tunnel won't be created and the feature will fail. To help them along, they'll see an instruction to run something like `localtunnel -k /Users/jeff/.ssh/id_rsa.pub` included in the test's output.