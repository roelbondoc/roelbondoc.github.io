---
layout: post
title: Introducing Discode
---

For years I have had a small collection of Rails projects sitting in private GitHub repos. Some were weekend experiments, other launched without any marketing before I moved on to the next idea. They all shared the same fate: running nowhere and earning nothing. Earlier this year I finally decided to solve my own problem and give those projects a second life. The result is **Discode**.

## Why build Discode

Selling a self‑hosted Rails app sounds simple until you try to do it:

* You need an installer that works on any fresh VPS.
* You need licensing, download links, and customer accounts.
* You need updates and a way for customers to pull them down.

Most tools focus on SaaS billing or Docker images. I wanted a workflow that stays close to plain Rails while still feeling turnkey for buyers. Discode grew out of that itch.

## What Discode is

Discode is a self‑contained Rails application you install on your own server. Once it is running you can:

* Create product entries for each app you want to sell.
* Push your app code via git.
* Generate a one‑line installer your customers can run on their own server.
* Push updates with a version tag and let customers upgrade with a single command.

Think of it as your private “App Store” for Rails projects.

## Installing Discode

If you have a Linux box with a public IP and a domain pointed at it, you can be up and running in minutes:

```bash
/bin/bash -c "$(curl -fsSL https://rubyup.dev/install/<install key>)"
```

The script asks for your domain name, installs dependencies, configures HTTPS with Let’s Encrypt, and boots the app. When the installer finishes, visit your domain, choose a username and password, and you are in.

## Selling your first app

Inside the Discode dashboard:

1. Click **New App** and fill in the name and a short tagline.
2. Push your Rails app to the git repo.
3. Set a price or mark it free while you test.
4. Copy the generated install command and share it with a customer.

Your customer ends up with a clean, running copy of your app on their own server.

## See it in action

I recorded two short videos that walk through the whole flow:

* **Installing Discode** — the single‑command setup and first login
  [https://www.youtube.com/watch?v=N0ZxADS3SpQ](https://www.youtube.com/watch?v=N0ZxADS3SpQ)

* **Selling a Rails app with Discode** — creating an app entry, uploading code, and generating the installer
  [https://www.youtube.com/watch?v=t2D8ZmOMNEM](https://www.youtube.com/watch?v=t2D8ZmOMNEM)

## Try Discode

If you have a Rails project gathering dust, give Discode a spin and tell me what you think. The code and pricing details live here: [https://rubyup.dev/discode](https://rubyup.dev/discode). Feedback, questions, and pull requests are always welcome.
