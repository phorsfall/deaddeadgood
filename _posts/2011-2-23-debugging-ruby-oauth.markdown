---
layout: post
title: Debugging Net::HTTP and Ruby OAuth
permalink: /2011/2/23/debugging-net-http-and-ruby-oauth/
---
There's a handy method on [Net::HTTP](http://www.ruby-doc.org/stdlib/libdoc/net/http/rdoc/classes/Net/HTTP.html#M001371) which allows you to send detailed debugging information (including request and response headers/bodies) to a stream. The [OAuth gem](https://github.com/oauth/oauth-ruby) uses Net::HTTP and it's easy to get at. For example, you can write debug info to your web server's log like this:

{% highlight ruby %}
# Create a consumer
consumer = OAuth::Consumer.new("key", "secret", :site => "https://agree2")

# Enable debugging
consumer.http.set_debug_output($stderr)

# Do stuff
request_token = consumer.get_request_token(:oauth_callback => "http://example.com")
{% endhighlight %}
