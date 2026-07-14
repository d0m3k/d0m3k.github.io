---
title: VPS, AI and from phone to cloud — my pocket-powered dev setup 
fileName: phone-to-cloud
tags:
- self-hosting
- android
- cloudflare
- mikrus
- termux
- pi
- deepseek
categories:
- infrastructure
date: 2026-07-14
lastMod: 2026-07-14
draft: true
---

There have been _changes_ in my life recently. My child is born now, which means:

* I got lot of time off work.
* I got much less focus time than I was used to.

These things did not discourage me from doing stuff though. I just needed to rework the way then can be done with lots of labor and focus shift needed.

## Motivator: the mikrus recycling

There is [mikrus](https://mikr.us/), fairly cheap VPS provider in Poland that does even cheaper thing -- [recycling](https://mikr.us/recykling.html) initiative that allows you to get some compute for literal 5 PLN[^1]. So obviously I did, and now I needed to find an use for it.

I remembered that me and a friend of mine used to send each other photos of Ryboczłek -- this fish with legs (and a bottom) that you will surely see walking around Kraków.

{{< figure src="/images/posts/fish-graffiti.jpg" link="/images/posts/fish-graffiti.jpg" target="_blank" caption="Have you seen one of these?" >}}

I thought then -- let's add a bit more structure to our Rybaspotting. Let's write an _app_ for it. I think the end result is pretty lovely, have nice UI, and is welcoming everyone to play as well:

https://ryby.dom3k.pl/

You can also see the sources for it [here](https://github.com/d0m3k/rybaspotting).

{{< figure src="/images/posts/ryby-map.png" link="/images/posts/ryby-map.png" target="_blank" caption="The map view" >}}
{{< figure src="/images/posts/ryby-profile.png" link="/images/posts/ryby-profile.png" target="_blank" caption="Profile of the most prolific spotter at the point of writing (me)" >}}

But the thing is 


[^1]: Around 1.30 USD at the time of writing.
