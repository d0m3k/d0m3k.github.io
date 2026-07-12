---
title: From phone to cloud — my pocket-powered dev setup made cheap
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
date: 2026-07-12
lastMod: 2026-07-12
draft: true
---

Lots of things changed in my life recently, causing me to have less focus time and definitely less time in a fully-fledged multi-display setup. So it's hard to have a nice day full of hacking around. But, it's possible to have a small nuggest of time, allowing to brew ideas in between chores.

And luckily for me, it's 2026, everyone is hyped about a thing or two, and one of these things will allow me to still work effectively.

## A little trigger

A man sometimes thinks to himself: I definitely need a VPS.

It's also much, much easier to think so if you have access to  [mikrus](https://mikr.us/), fairly cheak VPS provider in Poland that does even cheaper thing -- [recycling](https://mikr.us/recykling.html) initiative that allows you to get some compute for literal 5 PLN[^1].

So, I ended up with a shell for a dollar, and now I needed an idea how to utilise it.

Luckily, there are fish.

{{< figure src="/images/posts/fish-graffiti.jpg" link="/images/posts/fish-graffiti.jpg" target="_blank" caption="Have you seen one of these?" >}}

We used to send, with a friend of mine, photos whenever we spot one of these. I thought that this may be the thing that will ocuppy my new (almost free) compute.

So I sat at my laptop and started exactly what you would start in 2026: vibe coding.

## Agentic YOLO laptop with `pi`

I started with `pi` and API based usage of [Deepseek](https://platform.deepseek.com/usage). This was pretty good and amazing value for money -- I could deploy a lot using 10 CNY[^2] in credits. A friend of mine, when heard of my setup, sent me the [OpenCode Go link](https://opencode.ai/go?ref=HP6F8JZQP7), which indeed currently offers much more input tokens for the buck you give it.

{{< figure src="/images/posts/pi-on-laptop.png" link="/images/posts/pi-on-laptop.png" target="_blank" caption="Pi running on my fedora. Note the bottom bar showing current limits in OpenCode Go, as well as Deepseek API topup level." >}}

Now, this was pretty conventional, right? You can ask it for a thing or two, including implementing of showing the limit usage you see on screen. If you like this setup, consider [cloning the repo with it](https://github.com/d0m3k/pi-config).

{{< note >}}
**WORD OF WARNING:** `pi` has crazily wide rights if run directly on your machine, including reading/writing/running anything without asking for permissions. If you are ~~more paranoid~~ less reckless than me, consider wrapping it in some docker-based sandbox with limited disk access, or at least run via some less YOLOitic harnesses, like `opencode` or deepseek-pilled `reasonix`.
{{< /note >}}

Now, with kid crying over the head, one cannot simply keep on running around with laptop in hand. You need something that, well, fits in hand.

Enter the Termux.

{{< figure src="/images/posts/pi-on-phone.jpg" link="/images/posts/pi-on-phone.jpg" target="_blank" caption="Pi running on my S23+. What?" >}}

Install the Temux via F-Droid, and you get pretty full-fledged terminal emulator with Linux with pretty proper `pkg` manager and real `bash` shell. Support for rendering pi is also very good, though plugins required some pi-assisted edits to work around the kinks (like [here](https://github.com/d0m3k/pi-config/commit/0f521f6ed7781a786bb5f8459a98461f1e688d0c)).

It also gives you `ssh`. And PI can do everything it wants, so, if you provide your public keys to the remote, it will happily run, deploy, `scp` and do any other stuff at will.

So you have Github for the code, and a small remote as "cloud" for your local dev setup now, that agent can reach on its own from either end.

## A step further: `pi` and `tmux` on VPS

After all, why not? We can push this to extreme and have a real-time rendering on both ends. It even auto-adapts to a smaller screen so that rendering is readable on both.

This I found interesting, but not that usable really. What is really usable is making sure agents always push to github and that we have autodeploy on remote.




# AI slop to remove below

## The phone: Termux

[Termux](https://termux.dev/) is a terminal emulator and Linux environment for Android. It gives you a real `bash` shell, `apt` package manager, and access to pretty much everything you'd expect from a Linux box — `git`, `node`, `ssh`, `vim`, you name it. It runs in a proot-style container under `/data/data/com.termux/files/usr`, which means no root required.

My daily driver is a Samsung Galaxy S23 (SM-S916B) running Android 16, with Termux installed via F-Droid. The F-Droid build is important — the Play Store version lags behind. Inside Termux I have Node.js (v26), git, and the [pi coding agent](https://github.com/earendil-works/pi-coding-agent) for AI-assisted coding directly in the terminal.

The big win is that the phone is always with me. Waiting for a tram? That's 10 minutes of coding. Lying in bed at 1 AM with a random idea? Open Termux and go. The constraint of a small screen and no mouse actually forces you to think more before typing, which isn't the worst thing.

## The AI sidekick: pi



Pi is an AI coding agent that runs as a CLI tool (`pi`). Think of it as a terminal-native alternative to Cursor or Copilot, but it works over SSH, in tmux, and — crucially — inside Termux on a phone. It can read files, execute shell commands, edit code with surgical precision, and reason about architecture. It connects to various LLM providers; I use DeepSeek.

The combination of Termux + pi means I can describe a feature in plain language while pi does the heavy lifting: scaffolding backend handlers, writing database migrations, debugging build errors, you name it. It's like pair programming with someone who never gets bored.

## The VPS: mikr.us

Every project needs a home on the public internet, and mine lives on [mikr.us](https://mikr.us/) — a Polish VPS provider that offers tiny, absurdly cheap servers. We're talking "price of a coffee per month" cheap. The server (`amy135.mikr.us`) runs Debian and hosts whatever I need: Go backends, nginx reverse proxies, static sites, you name it.

The mikrus community has a wiki full of helpful scripts, and the whole vibe is very "here's a VPS, don't do anything illegal, have fun." Perfect for side projects.

## The tunnel: Cloudflare

Here's where it gets interesting. My home ISP (Netia, Poland) blocks incoming connections on pretty much every port. No port forwarding, no direct SSH from outside. The solution is [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) (formerly Argo Tunnel).

`cloudflared` is a daemon that creates an outbound connection to Cloudflare's edge network, and Cloudflare then routes traffic back through that connection. This means:

- **SSH access from anywhere**: I can SSH into my mikrus VPS through Cloudflare's network, even though Netia blocks outgoing SSH too (well, they block port 22, but Cloudflare Tunnel uses HTTPS). My SSH config looks like:

```
Host amy
    Hostname shell.dom3k.pl
    User root
    IdentityFile ~/.ssh/id_ed25519
    ProxyCommand cloudflared access ssh --hostname %h
```

- **Web traffic without open ports**: Services on the VPS are exposed to the internet through Cloudflare's proxy. The tunnel handles TLS termination, and Cloudflare's CDN caches static assets globally. The whole thing works through NAT, CGNAT, and whatever else the ISP throws at it.

- **Zero Trust SSH**: Cloudflare Access sits in front of the SSH endpoint, meaning I can authenticate using my Cloudflare account or a one-time PIN. No need to expose SSH to the raw internet.

The tunnel runs as a systemd service on the VPS, quietly maintaining a permanent outbound connection. If the VPS reboots, the tunnel reconnects automatically.

## The object storage: R2

[Cloudflare R2](https://developers.cloudflare.com/r2/) is S3-compatible object storage with zero egress fees. This is the killer feature — most cloud providers charge you for data going out, but R2 doesn't. For a personal project that might serve images, map tiles, or user uploads, this means the bill stays at zero regardless of traffic.

R2 integrates naturally with the Cloudflare ecosystem. Files stored in R2 can be served through a custom domain with Cloudflare's CDN in front, cached at edge locations worldwide. For the Rybaspotting app (more on that below), R2 stores map tile cache and user-submitted photos.

## The case study: Rybaspotting

[Rybaspotting](https://ryby.dom3k.pl) is a PWA for spotting a specific piece of graffiti around Kraków. It's a full-stack app:

- **Backend**: Go + chi router + sqlx + PostgreSQL (on the mikrus VPS)
- **Frontend**: Preact + Vite + Leaflet maps + PWA manifest
- **Storage**: R2 for images and tile cache
- **Deploy**: GitHub Actions builds both frontend and backend, creates a release, then a script on the VPS pulls the latest release

The entire thing is coded from Termux on the phone, committed via git, and deployed with a single `ssh amy` + curl command. The CI/CD pipeline means I never need to manually build or scp anything — just push to `master` and run the deploy script.

## Replacing GitHub Pages with Cloudflare Pages

This very blog started on GitHub Pages (as [the previous post](/posts/new-page/) describes). But once I was already using Cloudflare for DNS, tunnels, and R2, it made sense to move hosting there too.

Cloudflare Pages is a Jamstack hosting platform that builds and deploys directly from a git repository. It supports Hugo natively — just point it at the repo, set the build command to `hugo`, and the output directory to `public/`. Every push to `main` triggers a new build and deploy.

Migration was simple: delete the GitHub Actions workflow for Pages, remove the `CNAME` file, and configure the custom domain (`dom3k.pl`) in the Cloudflare dashboard. The build times are comparable, the CDN is global, and everything lives under the same Cloudflare account as DNS, tunnels, and R2.

## The whole picture

Here's how all the pieces connect:

```
Phone (Termux + pi)
    │
    │  git push / ssh via cloudflared
    ▼
GitHub (source code)
    │
    │  GitHub Actions → build → release
    ▼
mikrus VPS (backend + nginx)
    │
    │  cloudflared tunnel (outbound)
    ▼
Cloudflare Edge
    ├── ryby.dom3k.pl (PWA)
    ├── dom3k.pl (Cloudflare Pages)
    ├── shell.dom3k.pl (SSH tunnel)
    └── R2 (object storage)
```

The whole setup costs me roughly:
- **Domain**: ~80 PLN/year for `dom3k.pl` (OVH)
- **VPS**: ~40 PLN/year for mikrus
- **Everything else**: Free tier (Cloudflare, GitHub, R2 up to 10GB)

That's about 120 PLN/year total — less than a single month of a mid-tier VPS from the big clouds. And the best part? If I get hit by inspiration at 2 AM, the entire development environment is already in my pocket.

## Wrapping up

Is coding on a phone practical? For professional work with large codebases and heavy IDEs, probably not. But for side projects, infrastructure tinkering, and the sheer joy of making things work in constrained environments, it's surprisingly capable. The ecosystem of Termux, cheap VPS providers, and Cloudflare's free tier makes it possible to run a full-stack application from a device that also takes phone calls.

If any of this sounds interesting, the code is public: [github.com/d0m3k](https://github.com/d0m3k). Feel free to steal ideas, file issues, or just say hi.




[^1]: Around 1.30 USD at the time of writing.
[^2]: Now, this is again around 5.50PLN, which is still below 1.5 USD.