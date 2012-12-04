---
layout: post
title: "Introducing dotjs-pow"
date: 2012-12-04 14:06
comments: true
categories:
---

I just released a small piece of code called [dotjs-pow](https://github.com/forresty/dotjs-pow) on GitHub, before I tell you what it is, let me tell you a short story first.

You see, recently, I read a lot of books. I bought most of them on [amazon.cn](http://amazon.cn) since you know, dead-tree versions of Chinese books are dirt cheap.

And also, there is this Chinese site [douban.com](http://douban.com), which has generally pretty reliable Chinese book ratings and reviews, and you know what, they have some nice APIs too!

Since I find myself constantly switching Safari tabs between pages from amazon.cn and douban.com, I started to think that how can I make this easier.

Initially, I thought about building a Safari extension, but it is not an elegant solution.

Then I found this awesome piece of code called [dotjs](https://github.com/defunkt/dotjs) by GitHub CEO(!) Chris Wanstrath (a.k.a. defunkt), basically it let's you run a custom js file located in your `~/.js` each time you visit a site.

"Yes!", I think, so in a hurry I installed it (albeit I have to install [dotjs.safariextension](https://github.com/wfarr/dotjs.safariextension) as well since defunkt was a Chrome guy) and put following code in my `~/.js/amazon.cn.js` to put it to a test:

```javascript
$(document).ready(function () {
  $.ajax({
    url : 'http://api.douban.com/book/subjects?alt=json&q=9787533935030',
    type : 'get',
    dataType : 'JSON',
    success : function () {
      alert('yay!');
    },
    failure : function () {
      alert('meh');
    }
  });
});
```

And points my Safari to amazon.cn. As you might have already guessed, I saw this in Safari console:

![Access-Control-Allow-Origin.png](/images/Access-Control-Allow-Origin.png)

Hmm. Typical.

Since I don't control either amazon.cn or douban.com, one solution is that I build an API proxy for api.douban.com.

It's quite easy to do so using Sinatra and HTTParty:

```ruby
require 'sinatra'
require 'httparty'

disable :protection

get '/api/douban/book/isbn/:isbn' do
  headers 'Access-Control-Allow-Origin' => '*', 'Content-Type' => 'application/json; charset=utf-8'

  HTTParty.get("http://api.douban.com/book/subjects?alt=json&q=#{params[:isbn]}").body
end
```

But of course I don't want to put the proxy on the Internet! So am I going to run yet another service on my localhost? Hmm, I don't like ugly hacks.

Then I realized `djsd` used by dotjs is just a static file server, and since I already have a Rack-compliant [Pow](https://github.com/37signals/pow) running on my machine (as many Rubyists do), why not just build a Pow powered dotjs server, which can also be used to bypass cross-site security protections as well?

Thus [dotjs-pow](https://github.com/forresty/dotjs-pow) was born.

And with following codes in my `~/.js/amazon.cn.js`, job's done.

```javascript
String.prototype.trim = function () {
  return this.replace(/^\s+|\s+$/g, '');
};

String.prototype.isbnize = function () {
  return this.replace(/,.+$/g, '').trim();
};

// http://stackoverflow.com/questions/1408289/best-way-to-do-variable-interpolation-in-javascript
String.prototype.supplant = function (o) {
  return this.replace(/{([^{}]*)}/g, function (a, b) {
    var r = o[b];
    return typeof r === 'string' || typeof r === 'number' ? r : a;
  });
};

$(document).ready(function () {
  var b = $('b:contains("ISBN:")');
  if (b.length > 0) {
    var isbn = b[0].nextSibling.data.isbnize();
    $.ajax({
      url : 'http://dotjs-pow.dev/api/douban/book/isbn/' + isbn,
      type : 'get',
      dataType : 'JSON',
      success : function (json) {
        var numRaters = json['entry'][0]['gd:rating']['@numRaters'];
        var rating = Number(json['entry'][0]['gd:rating']['@average']);
        var link = json['entry'][0]['link'][1]['@href'];
        // ratings: 7.9 => 4_0, 8.5 => 4_5, 8.6 => 5_0
        var stars_1 = Math.floor((rating + 1.4) / 2);
        var stars_2 = Math.floor(rating + 1.4) === stars_1 * 2 ? 0 : 5;
        var html = '<span class="swSprite s_star_{stars_1}_{stars_2}" title="{numRaters} ratings"></span><span>(<a href="{link}">{rating}</a>)</span>'.supplant({
          stars_1 : stars_1,
          stars_2 : stars_2,
          numRaters : numRaters,
          rating : rating,
          link : link
        });
        console.log(html);
        $(html).insertAfter($('#btAsinTitle')[0].parentNode);
      },
      failure : function () {
        console.log('something was wrong.');
      }
    });
  };
});
```

Check it out in action (please forgive me, I suck at styling web pages...):

![amazon.cn-douban-book-api.png](/images/amazon.cn-douban-book-api.png)

Oh, of course I have to use [a modified version](https://github.com/forresty/dotjs-pow.safariextension) of dotjs.safariextension since I changed the server address from http://localhost:3131 to http://dotjs-pow.dev.

The full source code of app.rb (that is, the dotjs-pow server) is as following:

```ruby
#!/usr/bin/env ruby

require "bundler"
Bundler.require

# sometimes, it is more fun to do it without protection
disable :protection

get '/js/:js_file' do
  headers 'Access-Control-Allow-Origin' => '*', 'Content-Type' => 'text/javascript; charset=utf-8'

  File.read(File.expand_path("~/.js/#{params[:js_file]}")) rescue "{}"
end

get '/api/douban/book/isbn/:isbn' do
  headers 'Access-Control-Allow-Origin' => '*', 'Content-Type' => 'application/json; charset=utf-8'

  HTTParty.get("http://api.douban.com/book/subjects?alt=json&q=#{params[:isbn]}").body
end
```

If you are a hopeless nerd like me, you might want to check it out yourself, visit [dotjs-pow on GitHub](https://github.com/forresty/dotjs-pow) now.

BTW, [I am currently available for hire](/hire).
