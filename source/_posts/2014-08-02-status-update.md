---
layout: post
title: "Status Update"
date: 2014-08-02 21:08
categories:
---

I started working on this project since last November:

[modouwifi.com](http://modouwifi.com/router.html)

It's pretty overwhelming but with a lot of fun!

What is so special in modou router? We have an awesome touch screen. Configuring your WiFi network has never been this easy, even your mom can do this.

Curious about what the experience of the router will be? We've created a touch screen emulator

You definitely should try it out now:

`$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$`
`$$$$$$$$$$$$$$$$$ BEGIN OF AWESOMENESS $$$$$$$$$$$$$$$$$`
`$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$`

[http://modouwifi.github.io/TP-Emulator/](http://modouwifi.github.io/TP-Emulator/)

`$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$`
`$$$$$$$$$$$$$$$$$$ END OF AWESOMENESS $$$$$$$$$$$$$$$$$$`
`$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$`

With this screen, you also don't need to worry if anyone is stealthily stealing your WiFi, because we have anti-theft built-in, enable it, you can still get notified / block access of suspicious clients even they somehow have the password of your WiFi network:

![anti-theft](/images/modouwifi-anti-theft.png)

We'd love to build an awesome community, therefore we have open-sourced a bunch of stuffs on GitHub at [https://github.com/modouwifi](https://github.com/modouwifi)

For example, the modou router has a set of nice HTTP/JSON API built in: [https://github.com/modouwifi/modouwifi-api](https://github.com/modouwifi/modouwifi-api)

For the nerds like us, we have command line tools built on top of the HTTP API for controlling the router:

- [Ruby client `modou-gem`](https://github.com/modouwifi/modou-gem)
- [Go client `md`](https://github.com/modouwifi/md)

What is more fun? We have a framework for building applications which will run on the modou router: [https://github.com/modouwifi/app-framework](https://github.com/modouwifi/app-framework)

Sample applications (plugins):

- [samba file sharing](https://github.com/modouwifi/modou-samba)
- [HDNS (dnsmasq config) for easy fighting with the GFW](https://github.com/modouwifi/hdns)
- [customizable captive portal welcome page](https://github.com/modouwifi/welcome-page)

We also have API for building apps for the screen/touch-panel: [https://github.com/modouwifi/modou-tp-api](https://github.com/modouwifi/modou-tp-api)

Wait, API for the touch screen? Is that even possible?

Well, check this post from our forum, see what good folks have already done with the touch screen API:

[插件Modou-Love - 七夕送路由器给妹子吧](http://bbs.modouwifi.cn/thread-7223-1-1.html)

One more thing...

We are looking for awesome iOS engineer(s) to join us, ping me on Twitter [@forresty](https://twitter.com/forresty) if you are interested.
