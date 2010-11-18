---
layout: post
title: Running MIT Scheme in Emacs on OS X
permalink: /2010/11/18/mit-scheme-in-emacs-on-os-x/
---

1. Download and install [MIT Scheme](http://www.gnu.org/software/mit-scheme/).

2. Create a symbolic link to the Scheme binary, something like this:

        cd /usr/local/bin
        sudo ln -s /Applications/mit-scheme.app/Contents/Resources/mit-scheme scheme

3. Set the `MITSCHEME_LIBRARY_PATH` environment variable in Emacs. I'm using the [Emacs Starter Kit](https://github.com/technomancy/emacs-starter-kit) so I do this in `~/.emacs.d/paulhorsfall.el`, you'll need to put it somewhere appropriate for your set-up. You might also want to set this in your shell if you'd like to run scheme there.

        ;; MIT Scheme
        (setenv "MITSCHEME_LIBRARY_PATH"  "/Applications/mit-scheme.app/Contents/Resources")

With that, you can now do `M-x run-scheme` to start a scheme interpreter. You can switch a buffer to Scheme mode using `M-x scheme-mode`, and from there you can do `C-x C-e` to send the expression to the left of the point to the Scheme interpreter for evaluation.