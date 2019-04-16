---
layout: post
title: 'Fault-Tolerance: An NPM Package to Format and Normalize Errors'
date: 2019-04-16T19:30:18.717Z
image: /assets/uploads/pankaj-patel-515219-unsplash.jpg
---

Developers don't spend enough time thinking about their errors. I see too many projects that `throw Error('invalid data')` or even worse `throw 'invalid data'` 😱. That isn't useful! Give me some context, bud.

But we (myself included) aren't usually thinking about the failure case. We are thinking about the solution and (at best) are simply guarding against some bad input. That need to change. Errors need context to be useful. String interpolation isn't good enough - it still requires thought. An error databag is what we need.

Dead simple and effortless - that was my goal. What I ended up with is a project I am calling [ Fault-Tolerance](https://www.npmjs.com/package/fault-tolerance). The concept behind it is simple - extend the default Error object to format errors better.

In the most basic example you can `throw new Fault('Move along');`. In reality, though, that kind of error isn't as helpful as you want. Errors have context and we don't want to lose that. 

```
function checkpoint(droids:[]){
  try{
    droids.forEach(d => {
      if(d.isWanted) {
        if(jediIsPresent) {
          throw new Error('These are not the droids you are looking for');
        }
        detain(d);
      }
    })
  }
  catch(e) {
    // a Fault will preserve the original stacktrace
    throw new Fault(e, {droids: droids});
  }
}
```

The output from that would look like:

```
Error: These are not the droids you are looking for
    at ......
    at ......
    at ......
metadata:
{
  "droids": [
      {
       "name": "R2D2",
       "isWanted": true
      },
      {
       "name": "C3PO",
       "isWanted": true
      }
  ]
}
```

This gives you a much better way to include addtional information with the added benefit of a consistent log format. 

Check out the [Fault-Tolerance](https://gitlab.com/joekaiser/fault-tolerance) on Gitlab for more examples. It is also available as an [NPM package](https://www.npmjs.com/package/fault-tolerance).
