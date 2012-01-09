---
layout: post
title: "strip trailing whitespaces automatically in TextMate2"
date: 2012-01-09 12:45
comments: true
categories: TextMate2
---

I've been using TextMate2 the moment its alpha was out, and it has already proved itself good enough for my day-to-day coding tasks.

But there's a problem: I want my codes to be absolutely clean, and trailing whitespaces in lines end really bug me.

Though there's a command in Text Bundle which allows you to manually do the task, I would love TextMate to remove them automatically for me.
By default it doesn't have such functionality, but fortunately after some research through the Internet, I found a solution here:

[http://reinteractive.net/posts/4-stripping-whitespace-out-of-textmate-2](http://reinteractive.net/posts/4-stripping-whitespace-out-of-textmate-2)

In a nutshell, you need to create a custom bundle with a custom command looks like this:

{% img /images/textmate2-strip-whitespaces.png %}

It utilizes following script, which is shamelessly copied from Text Bundle in TextMate:

{% gist 1581221 %}

And that's it. Each time you pressed CMD + S, all trailing whitespaces in lines end are automatically removed before the document is saved. Cheers!
 