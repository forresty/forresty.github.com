---
layout: post
title: "Ayi Loves You"
date: 2012-12-11 17:26
comments: true
categories:
---

Recently I read a lot of open source code, some of them are really nice, some of them suck.

Since I have my TextMate 2 set up so trailing whitespace and `\t` in Ruby files are highlighted, every now and then when I stumble upon those patterns, I find myself fixing them locally, and depending on my mood, I might send them a pull request.

Though I have a keyboard shortcut set up in my TextMate so it's just a trivial task for each file, but it's really annoying to see them coming up again and again, right?

When yesterday my [pull request to capistrano](https://github.com/capistrano/capistrano/pull/319) was rejected and was told by this nice guy called Roy Liu, that rather than fixing them file by file, it's preferred to do it in batch.

So today I literally went through the entire project, read through the code, trying to understand them while fixing the styles. Oh man it hurts my eyes.

Obviously some tasks are better done using automation. Thus in a hurry I wrote this little [ayi gem](https://github.com/forresty/ayi).

Basically it clean up codes for you, and I intend to make it do this one thing and do it well.

Of course It's now still very early in development, but let's hope it will become more and more powerful as time goes by.

Go [take a look at it](https://github.com/forresty/ayi) if you are interested.
