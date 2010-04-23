--- 
layout: post
title: The State of Ruby's send Method
---
<p>Calling a method in Ruby can be thought of as sending a message to an object. This isn't just an abstract concept, it's actually implemented in the language by the <code class="ruby">send</code> method.</p>

<pre><code class="ruby">'foo'.length
# => 3
'foo'.send :length
# => 3
</code></pre>

<p>This is a really important method in Ruby (often used in <a href="http://en.wikipedia.org/wiki/Metaprogramming">meta-programming</a>), so important in fact, that just in case you accidentally overwrite <code class="ruby">send</code> in your own code, it is aliased as <code class="ruby">__send__</code> to make sure it can always be called.</p>

<p>One aspect of <code class="ruby">send</code> that often causes discussion<sup><a href="#fn1-47">1</a></sup> is the fact that it allows you to call private methods which would otherwise be inaccessible using the regular method calling syntax, for example:</p>

<pre><code class="ruby">class Foo
  private
  def bar
    'baz'
  end
end

foo = Foo.new

foo.bar
# => NoMethodError: private method `bar' called for #&lt;Foo:0x118fcbc>

foo.send :bar
# => "baz"
</code></pre>

<p>And for a while, Ruby 1.9 changed the implementation of <code class="ruby">send</code> to only allow public methods to be called, introducing a new <code class="ruby">send!</code> method<sup><a href="#fn2-47">2</a></sup> which continued to allow private methods to be called. However, this has now been removed from Ruby 1.9 in favour of keeping the functionality of <code class="ruby">send</code> as is. Instead, a new <code class="ruby">public_send</code> method has been introduced which as you might expect will only call public methods.</p>

<ol>
<li id="fn1-47">There are probably others threads, but <a href="http://blade.nagaokaut.ac.jp/cgi-bin/scat.rb/ruby/ruby-talk/263034">the send bang method was discussed on the Ruby Talk mailing list</a>.</li>
<li id="fn2-47">For forward compatability with Ruby 1.9 <a href="http://github.com/rails/rails/commit/b01a7c69fc8554c2e696609ea8ab76875d405165">send was aliased as send! in Rails</a> as well, although <a href="http://github.com/rails/rails/commit/a1eb4e11c2cccb91483fa15f1a1a0b2ae518d2cf">this has also since been removed</a>.</li>
</ol>
