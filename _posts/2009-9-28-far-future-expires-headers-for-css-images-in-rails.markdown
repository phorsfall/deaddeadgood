---
layout: post
title: Far Future Expires Headers for CSS Images in Rails
permalink: /2009/9/28/far-future-expires-headers-for-css-images-in-rails/
---
<p>Setting far future expires headers for your static assets noticeably improves the speed at which pages load and is easy to setup in Rails thanks to <a href="http://api.rubyonrails.org/classes/ActionView/Helpers/AssetTagHelper.html">asset timestamps</a>. All you need to do is:</p>

<ol>
<li>Use <code>image_tag</code>, <code>stylesheet_link_tag</code>, <code>javascript_include_tag</code> and friends in your views.</li>
<li>Update your web server config to set the correct headers.</li>
</ol>

<p>One problem that's not immediately obvious here is that while your web server is happily adding expires headers to the images you're using in CSS, those requests don't have the timestamp in the query string so the browser will continue to use the cached version after you have updated the file.</p>

<p>One solution is to instruct the web server to <a href="http://www.dcmanges.com/blog/asset-versioning-in-rails">only add the expires header to requests that do include the asset timestamp</a>. While this might be ok if you only have a few images in your CSS, if most of your images are specified this way then you're still not making the most of the browser's cache which was the whole point of the exercise in the first place.</p>

<p>An alternative solution is to use ERB templates to generate your stylesheets, and use the Rails' helpers to get asset timestamps in your CSS. It's dead easy, here's how:</p>

<h3>Create a Stylesheets Controller and a Matching Route</h3>

{% highlight ruby %}
class StylesheetsController < ApplicationController
  caches_page :application, :iphone
end
{% endhighlight %}

{% highlight ruby %}
map.connect "/stylesheets/:action.css", :controller => "stylesheets", :format => "css"
{% endhighlight %}

<p>This is just about as simple as you can imagine. We're using page caching to cache the stylesheet to disk, once rendered it will be served like any other static asset.</p>

<h3>Move Stylesheets to the Views Directory</h3>

<pre><code>mv public/stylesheets/application.css app/views/stylesheets/application.css.erb
mv public/stylesheets/iphone.css app/views/stylesheets/iphone.css.erb</code></pre>

<h3>Use the image_path Helper</h3>

<p>Wherever you reference an image in your stylesheet, use the <code>image_path</code> helper like so:</p>

<pre><code>body { background: url(&lt;%= image_path('fade.png') %&gt;); }</code></pre>

<p>When the template renders this will output something like:</p>

<pre><code>body { background: url(/images/fade.png?1253089219); }</code></pre>

<p>And that's all there is to it. After each deploy the cached stylesheet will disappear and the next time it's requested it will be regenerated with new timestamps, which in turn will cause the browser to fetch fresh copies of your images.</p>

<h3>A Note on Asset Hosts</h3>

<p>We're also using an asset host, for a couple of reasons:</p>

<ol>
<li>A separate host for static assets can itself improve page load speed.</li>
<li>We can specify far-future headers on anything served from the asset host, safe in the knowledge that the URL will have been generated using the Rails helpers and will therefore include the timestamp in the query string.</li>
</ol>

<p>This causes one additional complication with the approach described earlier however. Our asset host doesn't run Rails since it only serves static assets, but the first request for the stylesheet needs to go through Rails so the template can be rendered and written to disk.</p>

<p>Assuming you're using Capistrano, a simple solution is to add a task that fetches the stylesheets from the Rails host after each deploy. Adding this to <code>deploy.rb</code> will do the trick:</p>

{% highlight ruby %}
task :prime_cache, :roles => :app do
  run <<-CMD
    wget --spider http://#{domain}/stylesheets/application.css;
    wget --spider http://#{domain}/stylesheets/iphone.css
  CMD
end

after "deploy", :prime_cache
after "deploy:migrations", :prime_cache
{% endhighlight %}
