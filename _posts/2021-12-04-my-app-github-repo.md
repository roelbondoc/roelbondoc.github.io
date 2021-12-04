---
layout: post
title: My `app` GitHub Repo
---

I often enjoy starting and building new hobby projects. My ideas can come from different places. Sometimes there is an itch I need to scratch, or sometimes I just want to build something to see if it’s possible. In any case, I’ve been able to distill my process to a few general steps. Most, if not all, of my projects start off with a similar foundation. Usually they involve a combination of a Docker container with a Ruby on Rails application. I’ve taken this one step further and I’m releasing the basic structure of my application from which I start all my projects with.

Take a look at it here: [https://github.com/roelbondoc/app](https://github.com/roelbondoc/app)

The idea behind this repo is that anytime I want to start a new project, I can clone this repo and just start building right away. Since it’s a blank Ruby on Rails application with all the mundane tasks already completed, I hope to save time when starting a new application.

Now that I’ve open sourced it, I plan on keeping it up to date with latest released versions.

## Getting Started
Here’s a few set of instructions to get you started if you wanted to check it out.

1. Clone the repo:
```
git clone git@github.com:roelbondoc/app.git
```
2. Start the application:
```
cd app
docker-compose up
```
3. Prepare the database:
```
docker-compose run —-rm app bin/rails db:create
```
4. Navigate to [http://localhost:3000](http://localhost:3000) with your web browser.

This should give you with a good starting point for building your application. Now this requires that you have both `docker` and `git` installed on your computer. If don’t have any of these installed, not to worry! `app` is already setup to run with GitHub codespaces, check it out here: [https://github.com/codespaces](https://github.com/codespaces). Then you can with step 2 with the above process.

## What’s Next
As I mentioned before, my main goal is to keep this continually updated to make it easier to start new applications. In addition, I want this `app` to be both development and production ready. I think in the long run, I hope this lowers the barrier to get more people to start building their own projects to solve their own problems or start their own business. Please give it try and let me know what you think!