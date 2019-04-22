---
layout: post
title: Include a Request ID in your API
date: 2019-05-16T08:40:00.000Z
image: /assets/uploads/christopher-robin-ebbinghaus-681475-unsplash.jpg
---
Let's pretend we have a micro-service that will take a blob of text and output an array of name found in it.

Some sample input might be

> To,\
> Troy Hall, 
>
> This is to inform you that your sample order is ready at our workshop and you are cordially invited to have a look and give us your opinion for any further changes if any. We are happy to inform you that this has been made within the time frame you have specified strictly. Sample is very much as per your description, details, design and also have possibility to include more features if there is any need from your end.
>
> I hope to see you soon at our workshop. We will wait for your presence to be felt along with your colleagues for a brief meeting after inspection.
>
> Yours Truly,\
> Jack Harrison

And we the output expected is :

```json
{
  "names": [
    "Troy Hall",
    "Jack Harrison"
 ]
}
```

Now let's add in another service that needs to call this. What happens when our name-finding service returns an error? We are responsible developers and we can't just ignore it. So we file a bug and attach the offending text to it. 

It is a week later and we are trying to get around to fixing this problem. Good thing we saved that input that broke things! Oh, but wait - without any code changes this thing gets the "works on my machine" badge.

Well, let's start grepping the logs. If only we had some kind of identifier we could use.

Request IDs are simple concept. Anytime you process an HTTP request you include some kind of identifier in the response. This is important not only for you (the server), but also the client. 

Implementing them in any language is easy. You generate some kind of unique identifier and include them in the response; typically as a header named `x-request-id`. Here is an example in Node using Koa as our http framework.

```javascript
export async function addRequestId(ctx: Context, next: () => void) {
  let reqId = getAUniqueId(); // could be anything. I like the uuid package
  ctx.set('x-request-id', reqId);
  ctx.state.reqId = reqId; 
  await next();
}
```

That is only half the puzzle as we need our server to actually use it. That is why I include the requestId in the `context.state`. I like to take care of logging in my route, but you could make this another middleware function if you wanted.

```javascript
router.post('/names', KoaBody(),
  async (ctx: Context, next) => {
    try{
      parseNames();
    }
    catch(e){
      log.error(`name parse failed. requestId:  ${ctx.state.reqId}`,e);
    }
  }
);
```

Now, the best part is when the client has a problem all you need to do is document the Request ID and you will be able to find the exact log message you are looking for.

This dovetails nicely with the [Fault-Tolerance](https://joekaiser.dev/2019/04/16/fault-tolerance-an-npm-package-to-format-and-normalize-errors) package I wrote about earlier. You will get a beautifully formatted error if you add the requestID as the metadata.
