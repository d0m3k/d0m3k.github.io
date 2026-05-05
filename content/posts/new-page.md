---
title: How I saved myself few bucks by starting this blog
fileName: new-page
tags:
- hosting
categories:
- hosting
date: 2026-05-01
lastMod: 2026-05-05
--- 

  

So, I've kept a hold on this domain for around 10 years already. I think I initially wanted it to have cool revDNS on IRC channels, and then to keep some kind of personal and professional email address, as I was too young to get a good GMail back in the day (it's definitely not possible now, unless you consider `dominik.adamiak514 [at] gmail.com` professional).

So the plan was simple: get the domain and the cheapest hosting possible with a custom email (preferably with forwarding) to keep this up.

For years I've been using a few Polish operators in sequence that went like "we have nice cheap starter setup, and around year 3, we'll start billing 3x for the domain and 4x for the hosting, because you probably don't ever remember about this yearly invoice". Well, I actually do remember.

My last move went to [OVH](https://www.ovhcloud.com/pl/domains/), which is definitely cheap enough for .pl domain (and without predatory pricing for renewals), but is slightly too pricey for its starter hosting. Not a big deal, but why pay if you can just _not pay_?

So I knew I could do the following things for free:

* Host a static page on GitHub Pages and point it to my domain
* Use Cloudflare mail forwarding

The missing part was, can I _send responses_ via this free setup? I didn't know that, but I asked someone who did, and Gemini told me: you can use [Brevo](https://www.brevo.com/) what will allow you to send 300 messages per SMTP per day, and if you don't like the watermark in the email, use [Resend.com](https://resend.com/). I actually didn't like the watermark, and daily limit of 100 mails sounded definitely good enough for my needs. It was settled.

Buckle up, we're changing DNS and waiting for a lot of stuff.

The recipe:
1. Go to [Cloudflare](https://dash.cloudflare.com/) and start setting up your domain. When you do, it will give you two NS servers to redirect your domain to. It's very important you do if _first_, because this will take 1-2 hours for every service below to "get". In case of OVH, there is "DNS Servers" section in OVHcloud -- change it there, making sure you have DNSSEC disabled on the main domain page. All the steps below will happily throw random errors at you before DNS caches invalidate globally.
2. In [GitHub](https://github.com/), go to your account settings > Pages, and add your domain. It will expect you to add TXT record in the domain DNS. Do it in Cloudflare panel.
3. While waiting for 1 to happen, you can create GitHub public repository `<your-username>.github.io`. You can start it with some index.html, and go its settings to point it to your custom domain. This may fail while 1 haven't propagated yet. Keep on retrying.
4. There are IPs for Github your A records should point to. Also add these in Cloudflare, keeping "orange proxy" OFF for the time being.
5. In the meantime, you can go to [resend.com](https://resend.com/), and insert your domain there as well. It will also generate some DNS records for you, and you should get them back into Cloudflare domain setup. Make sure you save your API key.
6. You can get back to Cloudflare and find "email forwarding" somewhere in the panel. Use your domain to forward it to your favourite email, adding catch-all email after the initial wizard. You will also need to confirm email you want to forward to.
7. If resend.com stopped throwing errors, you can configure your gmail sending as SMTP server in the GMail settings:

Server: `smtp.resend.com`

Port: `587`

User: `resend`

Password: `<your api key>`

When steps 3 and 5 will stop failing, you can enable "orange proxy" in Cloudflare for stats and free caching, if you're fine with being down when Cloudflare is down.

That's it! The only thing I remain paying for is the .pl domain, which is around 80 PLN/year at the time this is written. And if you want to see how to setup hugo for simple generation of blogposts like this, just go and see [the repo that produces this page](https://github.com/d0m3k/d0m3k.github.io). 

Don't hesitate to let me know if this is useful, and especially if it is not. You can find my contact on the [home page](/).
