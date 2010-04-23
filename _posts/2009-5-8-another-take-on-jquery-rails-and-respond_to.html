--- 
layout: post
title: Another Take on jQuery, Rails and respond_to
---
<p>I been using <a href="http://jquery.com">jQuery</a> to do AJAX with Rails a lot recently, and I love it. When I was starting out however, I hit the same problem that seems to trip up everyone: AJAX requests made from jQuery don't trigger <code>format.js</code> (or <code>wants.js</code> if you're cool) within the <code>respond_to</code> block of Rails controller actions.</p>

<p><a href="http://ozmm.org/posts/jquery_and_respond_to.html">The most common solution</a> I've seen to this problem involves adding a one-liner to your Javascript to instruct jQuery to add a <code>text/javascript</code> <code>Accept</code> header to AJAX requests. Which works great in FireFox, but not in Safari.</p>

<p>The problem seems to be that in Safari, <code>text/javascript</code> is <em>appended</em> to the existing <code>Accept</code> header, so Rails sees something like <code>text/html, */*, text/javascript</code> which it still interprets as a request for HTML.</p>

<p>An alternate solution that I've been using is to add the following to <code>environment.rb</code>.</p>

<pre><code class="ruby">config.action_controller.use_accept_header = false</code></pre>

<p>This instructs Rails to use the <code>X-Requested-With</code> rather than the <code>Accept</code> header to determine the request format. Since jQuery sets this header in both browsers, Rails controllers will behave as expected. </p>

<p>One thing to keep in mind is that if your app has an XML or JSON API, you might need to think carefully about whether this is the best way to fix this problem. For a while <a href="http://github.com/rails/rails/commit/2f4aaed7b3feb3be787a316fab3144c06bb21a27">this became the default in Rails</a>, although <a href="http://github.com/rails/rails/commit/4ce9931f4f30045b2975328e7d42a02188e35079">that didn't last very long</a>.</p>
