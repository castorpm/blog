---
title: "Dealing with Heroku by using Cloudflare: HTTPS and www redirects"
date: 2021-09-28T20:43:23+03:00
draft: false
---

I've been working with Heroku lately for a project for the first time, and
was wondering how to set a forced HTTPS redirection for those cases where
people just write the URL in the browser bar.

My first thought was that Heroku would provide that, since they take care
of the domain and stuff, but they don't. They, instead send you to their
plugins directory. Most plugins have a very... insufficient free tier and then
you have to pay. "You actually have to pay for a HTTPS redirect?" I wondered.

Then I thought "Hey, can't I do this with Cloudflare?". Yes, indeed you can.

## HTTPS redirect

This is pretty straightforward. You essentially go to your Cloudflare panel,
click on SSL/TLS and in the Edge Certificates you toggle "Always Use HTTPS":

![HTTPS redirect in Cloudflare](/img/https.png)

That's about it. As long as your page is actually redirected through Cloudflare 
(orange cloud thingy), HTTPS will be enforced.

## www to apex redirect

This one was slightly trickier to figure out.

So the first instinct for me was to go to the Rules page and click around. Page Rules
allow you to configure a forwarding URL for a matching URL:

![Page rules](/img/page_rules.png)

$1 will, as you'd expect, catch the value of the *, to be able to forward URL with non /
paths properly.

Then I realized with that pattern, `www.castor.pm` would not be redirected because
of the trailing slash. Then I went into the page rules, and removed the trailing slash,
so it read `www.castor.pm*` but when saving, Cloudflare brings the slash back, bringing
it to `www.castor.pm/*` again.

After looking around, I found out that, to make it work, you can set a DNS record like this:

```
Type  Name  Content    TTL   Proxy status
A     www   192.0.2.1  Auto  Proxied
```

According to a post by @domjh over at [the Cloudflare forums](https://community.cloudflare.com/t/using-page-rules-to-perform-redirects/55386):

>192.0.2.1 is an illustrative IP address often used for router setups on local networks etc. it doesn’t ‘go’ anywhere!

I'm still not sure why this works, but that seems to be the officially accepted way of
handling this. Maybe I'll read more deeply into it later on.