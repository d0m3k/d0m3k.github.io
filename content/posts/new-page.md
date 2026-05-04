---
title: How I saved myself few bucks by starting this blog
fileName: new-page
tags:
- hosting
categories:
- hosting
date: 2026-05-01
lastMod: 2026-05-01
--- 

  
Well, it’s actually more of a story about OVH, Cloudflare, Resend and some help from Gemini.

So, I've kept a hold on this domain for around 10 years already. I think I initially wanted it to have cool revDNS on IRC channels, and then to keep some kind of personal and professional email address, as I was too young to get a good GMail back in the day (it's definitely not possible now, unless I consider `dominik.adamiak514` professional).

So the plan was simple: get the domain and the cheapest hosting possible with email (preferably with forwarding) to keep this up.

For years I've been using a few Polish operators in sequence that went like "we have nice cheap starter setup, and around year 3, we'll start billing 3x for the domain and 4x for the hosting, because you probably don't ever remember about this yearly invoice". Well, given it's the sidest project of them all, I actually do remember.

My last move went to OVH, wich is definitely cheap enough for .pl domain (and without predator pricing for renewals), but has slightly too high price for its starter hosting. Not a big deal, but why pay if you can just not pay?

So I knew I can do a following thing:

* Host a static page on GitHub Pages and point it to my domain
* Use Cloudflare mail forwarding

The missing part was, can I _send responses_ via this free setup? I didn't know that, but I asked someone who does, and Gemini told me: you can use Brevo what will allow you to send 300 messages per SMTP per day, and if you don't like the watermark in the email, use Resend.com. I actually didn't like the watermark, and daily limit of 100 mails sounds definitely good enough for my human fingers. It was settled.

Buckle up, we're changing DNS and waiting for a lot of stuff!

Prepare and open:
* Your domain provider DNS console
* Cloudflare account and setup page
* resend.com account setup
* GitHub account

The recipe:
1. Go to Cloudflare and start setting up your domain. When you do, it will give you two NS servers to redirect your domain to. It's very important you do if _first_, because this will take 1-2 hours for every service to "get". In case of OVH, thesre is "DNS Servers" section in OVHcloud -- change it there, making sure you have DNSSEC disabled on the main domain page.
2. 