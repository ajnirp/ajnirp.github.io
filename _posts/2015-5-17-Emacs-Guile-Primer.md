---
layout: post
title: Quick Emacs/Guile primer
published: true
---

What's this?
------------

This post is a quick introduction on using Guile Scheme within Emacs, written mainly for future me. You're welcome, future Rohan!

Why?
----

I tend to jump between languages a lot and my general-purpose editor of choice is Sublime Text. Lately, though, I've been programming in Guile Scheme for my Summer of Code project, which means I've been playing around in Emacs a lot. It's fun and I really love the Emacs/Scheme workflow, but the only downside is that the shortcuts are a little different from those I'm used to in Sublime Text. Hence, this post. Although all this information is anyway available in the GNU Emacs guide and the [Geiser guide](http://www.nongnu.org/geiser/), I've culled the unimportant stuff and condensed the useful bits into this blog post.

Quick Note
----------

In the shortcuts that follow, `C` is `Ctrl`, `M` is `Alt`, `RET` is `Enter` and `S` is `Shift`. `<mouse-11>` is the left click mouse button, and `<mouse-3>` is the right click mouse button.

Starting out
------------

Get Emacs 24 with `apt-get install emacs24`.

Then, theme it to your liking. This means creating a file named `init.el` and placing it in the `~/.emacs.d` directory. Here are [my settings](http://paste.lisp.org/display/148290), with comments explaining the configuration.

Once you're done changing `init.el`, run `M-x eval-buffer` to tell Emacs about it - or just `M-x eval-region` to eval a particular sexp.

Next, install Geiser with `M-x package-install RET geiser RET`.

Basic Emacs stuff
-----------------

* `C-g` cancels the minibuffer command.
* `C-x C-s` saves your work.
* `C-x C-f` visits a file (which means it opens it if it exists and creates it if it doesn't).
* `C-x 0` closes the current buffer.
* `C-x 1` closes everything except the current buffer.
* `C-x-2` splits the current buffer horizontally and duplicates it in the newly created window.
* `C-x-3` does the same thing as above, except vertically.
* `M-x man RET pagename RET` opens up the man pages for `pagename`.
* `C-h i` opens the info pages. This includes the info pages for Emacs, Geiser, Guile and many other GNU programs and software.
* `C-h k` describes a keybinding. For example, try `C-h k C-h k`.
* `C-x k buffername RET` kills the buffer named 'buffername'. Merely typing `C-x k` allows you to scroll up and down and select a buffer to be killed. Or you can type in the first few letters of the buffername and hit TAB to autocomplete.
* `C-x b buffername RET` visits the buffer named 'buffername'. Its behaviour is similar to that of `C-x k`.
* `M-;` comments or uncomments a region, short for `M-x comment-region`.
* `C-x l` counts the number of lines in the page, and displays in brackets the number of lines before and after the cursor position.

Using Geiser
------------

Note: Geiser supports Racket and Chicken Scheme too, but those aren't relevant to us right now.

* `M-x run-guile` summons a Guile REPL.
* `C-c C-z` switches between the `.scm` file currently being edited and this REPL.
* `C-up` goes back in history at the Guile REPL.
* `C-down` goes forward in history at the Guile REPL.

Dealing with sexps
------------------

##Moving around

* `C-M-f` (f = forward) jumps to the end of the sexp in front of the current edit point.
* `C-M-b` (b = backward) jumps to the beginning of the sexp behind the current edit point.
* `C-M-S-f` selects the sexp in front of the current edit point (Makes sense: Shift is used for selection).
* `C-M-S-b`: same thing, but for the sexp behind the current edit point.
* `C-i` intelligently indents the selected region. If no region is selected, it indents only the current line.

##Guile and sexps

* `C-x C-e` evaluates the sexp behind the current edit point.
* `C-c C-b` evaluates the entire buffer.

##Commenting
* Putting "*;" before a sexp comments it out (as well as all contained sexps), while leaving the containing sexps intact.

Buffers
-------

* `M-x buffer-menu` brings up the list of active buffers. This is useful because sometimes Emacs decides to reuse a window carrying one of your useful buffers for something else, like showing error messages when Geiser evaluates an sexp, or showing minibuffer command completion candidates. Use `up` and `down` to move around the buffers list, and hit `RET` when you find the buffer you were looking for.
* If you just want to look at a buffer, but not interact with it, do `M-x view-buffer RET` and then navigate to the buffer you want.
* `C-x o` (o = other) switches between visible buffers.
* `C-<mouse-1>` brings up a graphical buffer menu.

Viewing RFCs
------------

* For reading RFCs, I use `irfc`. Get it by following the instructions [here](http://www.emacswiki.org/emacs/Irfc).
* `M-x irfc-visit RET 2131` downloads RFC 2131 (if it isn't already download) to the location specified in `init.el` and opens it in the current buffer.