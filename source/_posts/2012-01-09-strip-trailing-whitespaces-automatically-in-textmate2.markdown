---
layout: post
title: "strip trailing whitespaces automatically in TextMate2"
date: 2012-01-09 12:45
comments: true
categories: TextMate2
---

I've been using TextMate2 the moment its alpha is out, and it has already proved itself good enough for my day-to-day coding tasks.

But there's a problem: I want my codes to be absolutely clean, and trailing whitespaces in lines end really bug me.

By default TextMate doesn't have such functionality, but fortunately after some research through the Internet, I found a solution here:

[http://reinteractive.net/posts/4-stripping-whitespace-out-of-textmate-2](http://reinteractive.net/posts/4-stripping-whitespace-out-of-textmate-2)

In a nutshell, you need to create a custom bundle with a custom command looks like this:

{% img /images/textmate2-strip-whitespaces.png %}

The following script is used to strip whitespaces, shamelessly copied from Text Bundle in TextMate:

{% gist 1581221 %}

And that's it. Each time you pressed CMD + S, all trailing whitespaces in lines end are automatically removed before the document is saved. Cheers!
