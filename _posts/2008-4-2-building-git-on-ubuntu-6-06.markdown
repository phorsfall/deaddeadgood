--- 
layout: post
title: Building Git on Ubuntu 6.06
---
<p>I'm spending some time updating my Rails projects to use <a href="http://www.capify.org/">Capistrano 2</a> and <a href="http://git.or.cz/">Git</a>. Here are the steps I took to build <a href="http://git.or.cz/">Git</a> on my Ubuntu 6.06 slice.</p>

<pre>
<code>
sudo apt-get install curl libcurl3-gnutls-dev libexpat1-dev
wget http://kernel.org/pub/software/scm/git/git-1.5.4.5.tar.gz
tar xvf git-1.5.4.5.tar.gz
cd git-1.5.4.5.tar.gz
NO_TCLTK=yes make prefix=/usr/local all
NO_TCLTK=yes sudo make prefix=/usr/local install
</code>
</pre>
