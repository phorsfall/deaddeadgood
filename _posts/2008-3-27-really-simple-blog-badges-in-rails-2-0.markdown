--- 
layout: post
title: Really Simple Blog Badges in Rails 2.0
---
<p>When I wanted a quick (<a href="http://ajaxian.com/archives/is-easy-implementation-the-same-as-good-code">and arguably dirty</a>) way to show my current reading list from deadtre.es on this blog, I figured that it was time to play with <a href="http://blog.codefront.net/2007/10/10/new-on-edge-rails-json-serialization-of-activerecord-objects-reaches-maturity/">the JSON support which was baked in Rails 2.0</a>. Wanting to keep things super simple, all I wanted to do was:</p>

<ul><li>Create a simple JSON feed I can include via a <code>&lt;script></code> tag. (I'll write a simple Javascript callback which will insert my reading list into each page. If I decide to make this easier for users I'll put together the usual configuration page where people can play with the look and feel of their badge and get some code to copy and paste.)</li>
<li>Keep things RESTful. Each user already has a URL for their bookmarks so I really wanted to avoid to avoid creating new controllers or actions.</li></ul>

<p>Cutting to the chase, this pretty much turns out to be a one-liner in Rails. Here's an example:</p>

<pre><code class="ruby">
class BookmarksController &lt; ApplicationController
  session :off, :only => :index, :if => Proc.new { |request| request.format.json? }
  
  def index
    # fetch bookmarks ...
    respond_to do |format|
      format.json do     
        excluded_book_attrs = [:aws_domain, :id, :created_at, :updated_at]        
        excluded_bookmark_attrs = [:book_id, :id, :updated_at, :user_id]
        render :json => @bookmarks.to_json(:except => excluded_bookmark_attrs, :include => { :book => { :except => excluded_book_attrs }}), :callback => 'show_deadtrees_books'
      end
    end
  end
end
</code></pre>

<p>It doesn't get much simpler than that! There are a couple of things worth pointing out:</p>

<ul><li><code>render :json</code> takes care of setting the content type to application/json in the reponse header. Additionally, it wraps the JSON in a call to the method specified by the <code>:callback</code> option. (I should allow users to override this via the querystring to avoid name conflicts in their Javascript or to remove the callback altogether to support use outside of a <code>&lt;script></code> tag.)</li>

<li>You can include Active Record associations when calling <code>to_json</code> using <code>:include</code></li>

<li>You can also control which attributes are serialized (including those on associations) using the <code>:only</code> and <code>:except</code> options.</li>

<li>I've disabled sessions for JSON requests to the index action since they're not required and carry a reasonable overhead.</li></ul>

<p>With that in place, all you need to do is implement your Javascript callback (<a href="/javascripts/custom.js">borrow mine</a> if you like) and your blog badge is done!</p>

