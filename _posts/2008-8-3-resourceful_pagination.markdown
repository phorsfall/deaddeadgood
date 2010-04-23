--- 
layout: post
title: resourceful_pagination
---
<p>I've been doing quite a bit of work recently on an Active Resource project and needed to add pagination to one of our resources. I had a quick look around to see what other people were doing, and while it looks like <a href="http://github.com/mislav/will_paginate/tree/agnostic/">work is underway to add support for Active Resource</a> (and ORMs other than Active Record) to <a href="http://github.com/mislav/will_paginate/">will_paginate</a> I don't think it's quite ready just yet. So while we wait, may I present <a href="http://github.com/phorsfall/resourceful_pagination/">resourceful_pagination</a>.</p>

<p>If you want to dive straight in, take a look at the <a href="http://github.com/phorsfall/resourceful_pagination/tree/master/README.txt">readme</a> then install the gem with:<p>

<pre><code>sudo gem install phorsfall-resourceful_pagination --source http://gems.github.com</code></pre>

<p>With the gem installed you get a shiny new <code>ActiveResource::Base#paginate</code> method. Use it like this:</p>

<pre><code class="ruby">MyResource.paginate(:page => params[:page])</code></pre>

<p>This returns a <code>WillPaginate::Collection</code> which you can use with will_paginate's view helpers as usual.</p>

<p>Easy, right?, Well, sort of. The gem makes a few assumptions about your Active Resource class and your RESTful resource. If you want to use this you'll probably need to make a few changes. First you need to add two class methods to your Active Resources.</p>

<ol>
<li>A <code>count</code> method which returns the total number of items. This could call a custom count action on the app server.</li>
<li>A <code>per_page</code> method indicating how many records you require per page.</li>
</ol>

<p>Something like this would do the trick:</p>

<pre><code class="ruby">class MyResource < ActiveResource::Base
  @@per_page = 10
  cattr_reader :per_page

  def self.count
    get(:count)['count']
  end
end</code></pre>

<p>You also need the <code>index</code> action of the controller on the app server to handle the <code>:starting_at</code> and <code>:per_page</code> parameters. These map to <code>:offset</code> and <code>:limit</code> in SQL and can be passed straight through to Active Record.</p>

<pre><code class="ruby">MyResourcesController < ApplicationController
  def index
    MyResource.find :all, :limit => params[:per_page], :offset => params[:starting_at]
    # Render XML etc.
  end

  def count
    count = MyResource.count
    render :xml => { :count => count }
  end
end</code></pre>

<p>And that's about it. There's a little bit more to do if you have a nested resource, but there's a bit more on that in the readme.</p>

<p>Enjoy.</p>
