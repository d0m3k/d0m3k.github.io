---
title: From phone to cloud — my pocket-powered dev setup
fileName: phone-to-cloud
tags:
- self-hosting
- android
- cloudflare
categories:
- infrastructure
date: 2026-07-12
lastMod: 2026-07-12
draft: true
---

I do all my side-project coding on a phone. It sounds absurd until you actually try it — and then it still sounds absurd, but in a fun way. Here's how the whole stack works, from the Android terminal to a live website, without a traditional computer in sight.

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
