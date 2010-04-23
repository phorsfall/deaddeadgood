--- 
layout: post
title: Installing _why's Bloopsaphone on OS X
---
<p>Inspired by a recent rush of nostalgia, I finally got around to playing with <a href="http://viewsourcecode.org/why/">_why's</a> <a href="http://github.com/mental/bloopsaphone">Bloopsaphone</a> recently. Give or take the odd crash, it works pretty well with whichever version of <a href="http://www.portaudio.com/">PortAudio</a> <a href="http://www.macports.org/">MacPorts</a> gave me.</p>

<pre><code>sudo port install portaudio
sudo gem install bloopsaphone -- --with-opt-lib=/opt/local/lib --with-opt-include=/opt/local/include
</code></pre>

<p>There are a couple of Bloopsaphone tunes by <a href="http://freqvibez.net/">freQvibez</a> which are worth looking at if you want to hear the kind of thing you can create with this.</p>

<ul>
<li>cheeky drat (<a href="http://freqvibez.net/music/cheeky_drat.mp3">mp3</a>|<a href="http://github.com/localhost/bloopsaphone/blob/master/ext/ruby/tune_cheeky_drat.rb">source</a>)</li>
<li>warp-o-mat (<a href="http://freqvibez.net/music/warp-o-mat.mp3">mp3</a>|<a href="http://github.com/localhost/bloopsaphone/blob/master/ext/ruby/tune_warp.rb">source</a>)</li>
</ul>
