---
layout: post
title: The Hype Around Code Generation
description: A critical perspective on AI code generation tools like GitHub Copilot and why developers need less code, not more automated code generation.
---

Today I read about GitHub's release about their "GitHub Copilot Workspace".

From what I gather it's a tool backed by AI to help write code by describing what you want in a plain natural language. I'll admit, I've been slow to get on the AI bandwagon. I signed up for ChatGPT a few weeks ago to see if I can get a quick pizza dough recipe, as well as write a few lines of ruby code, which surprising did very well for what I needed. However, I still remained skeptical about fully integrating it with my day to day tasks. On top of that, I feel that, even though AI seems to be the tool that developers want, it's not the thing that we actually need. To keep it short, what we need as developers is less code, not more! If you look at the [press release](https://github.blog/2024-04-29-github-copilot-workspace/), GitHub talks about this very problem:

> Around the world, developers add millions of lines of code every single day to evermore complex systems and are increasingly behind on maintaining the old ones.

Yet, developers, media, and all, keep praising tools like Copilot like it's the blessing we've all been waiting for. But why should I be so welcoming to a tool that helps me generate more code, faster, than ever? (If I enjoyed writing code that much, I would have been a Typescript developer). Every line of code that gets written, is another line of code added into an ever growing context. That context needs to be understood by someone at somepoint.

I'm sure that Microsoft/Github has spent a huge amount of resources into this tool. Perhaps they've already done the research as to what makes for a more productive developer and this is it. Perhaps by whatever metric, or quota they've come up with that measures productiveness in some positive way, Copilot helps facilitate that. In my personal experience, I have always reached for those tools and frameworks that helped me create, with as little output on my behalf as possible. This isn't the same as AI generated code. AI may easily write super complex and performant functions for me, but it is still code that I need to commit, run, test, and maintain.

If I'm writing software that is too complex for me to understand, give me a layer of abstraction to simplify it for me. These layers of abstraction and simplification is what we as developers should strive for. Don't write code for other machines and AI. Write code, and simplify, for your teammate and yourself! The dream is where things are so simple, we write (or let AI write) the least amount of code possible!

These tools that generate more code for you, could be a useful tool, but don't become complacent. Less is best. The more code we write, the more liability gets shipped. Let's look into reducing the amount of code we need to write. Let's look into building better frameworks, that require less thinking. Let's move forward on things where people can build on their own!
