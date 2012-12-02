---
layout: post
title: "Prototypal"
date: 2012-12-03 00:59
comments: true
categories:
---

So yesterday I decided it's time for me to stop procrastinating and start to learn JavaScript seriously.

(Yes, judge me, I am a Ruby guy but I suck at JavaScript!)

So I picked up this wonderful little book named _JavaScript: The Good Parts_ by the JavaScript guru Douglas Crockford and started to read it through in depth (Yes I read it twice before but never in depth). It was an immense amount of fun I must admit.

<!-- more -->

So just now I was trying to understand why the fuck following codes work:

### Code Sample #1

```javascript
Function.prototype.method = function (name, func) {
  this.prototype[name] = func;
  return this;
};

Object.method('superior', function (name) {
  var that = this, method = that[name];
  return function () {
    return method.apply(that, arguments);
  }
});
```

I only have a vague idea about prototype chaining in JavaScript, that's when an object failed to find a property in itself it will start to look it up in its prototype, and prototype's prototype and then all the way up to the top.

The codes above implements a `superior` function which mimics the `super` in OO world on the object prototype thus enables it on ALL objects, thus making following codes possible:

### Code Sample #2

```javascript
var father = function (spec) {
  var that = {};
  that.get_name = function () {
    return spec.name;
  };
  return that;
}

var son = function (spec) {
  var that = father(spec);
  var super_name = that.superior('get_name');
  that.get_name = function () {
    return 'proud son of ' + super_name();
  };
  return that;
};

var afu = father({name : 'afu'});
afu.get_name();
// afu

var afu_junior = son({name : 'afu'});
afu_junior.get_name();
// proud son of afu
```

I can easily understand the Code Sample #2. The `superior()` call seems really natural, and that's just the way APIs / libraries are supposed to work, right?

So I head back to Code Sample #1 to try to understand how the magic happened behind the scenes.

```javascript
Function.prototype.method = function (name, func) {
  this.prototype[name] = func;
  return this;
};
```

This part is easy. It defines a `method` function on the function prototype so all functions are benefitted from it, any function calls the `method()` gets its prototype assigned a function as a property. So far so good.

But why `Object.method()` works??

Well, the answer, unlike the stupid poor me, you might already knew, is that the `Object` is a function.

```javascript
typeof Object === function
// true

// in case you are wondering, this is truthy as well:
typeof Function === function
// true
```

I felt my brain had just exploded. Wow. So I did some more experiments:

```javascript
// So, following expressions are all truthy:
typeof Function === 'function'
typeof Function.prototype === 'function'
typeof Object === 'function'
typeof Object.prototype === 'object'
typeof {} === 'object'

// And more fun:
var obj = {};
var func = function () {
};

obj.hello;
// undefined

func.hello;
// undefined

Object.prototype.hello = 'sweetie';

obj.hello;
// sweetie
obj.__proto__;
// { superior: [Function], hello: 'sweetie' }

func.hello;
// sweetie
func.__proto__;
// { [Function: Empty] method: [Function] }
func.__proto__.__proto__;
// { superior: [Function], hello: 'sweetie' }

// Sneak peek into Object and Function using Safari (courtesy of my friend @Leaskh):
Object;
// function Object() {
//   [native code]
// }

Function;
// function Function() {
//   [native code]
// }

// This is interesting:
Object.prototype;
// { superior: [Function], hello: 'sweetie' }

// Yes they are all truthy:
obj.__proto__ === Object.prototype;
func.__proto__ === Function.prototype;
func.__proto__.__proto__ === Object.prototype;
```

So basically, hacking a `method()` function onto `Function.prototype` enables it for all functions including `Object`, and hacking a `superior()` function with invocation of `method()` on `Object` hacks it onto `Object.prototype` thus enables it for all objects.

So, the Code Sample #1 is exactly the same as:

```javascript
Object.prototype.superior = function (name) {
  var that = this, method = that[name];
  return function () {
    return method.apply(that, arguments);
  }
};
```

Per the `this` binding rules, this line in Sample Code #2

```javascript
var super_name = that.superior('get_name');
```

retrieves the `get_name()` function in `that` which carefully protects `that` in its closure and is ready to be called with `arguments`. Neat!

So I think I just figured out some of the most basic ideas about why JavaScript is called a prototypal language and how its prototype chaining really works. (Right? Right??)

This reminds me the moment of ecstasy when I first understood Ruby object / class / singleton-class hierarchies. Aha moments like this is one of the reasons why it is so fascinating to be working in this field.

Yes, my brain hurts now and I need to rest. And yes I <3 programming.

TL;DR:

TIL in JavaScript functions are objects, and Object is a function.
