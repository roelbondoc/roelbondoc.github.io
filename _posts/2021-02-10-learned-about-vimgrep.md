---
layout: post
title: Learned about vimgrep
description: Discover how to use Vim's built-in vimgrep command to search patterns across multiple files and edit them without leaving your editor.
---

I learned a neat vim trick the other day.

Say you wanted to search for multiple occurrences of a pattern in a bunch of files. I somewhat knew how to do this with `grep` but felt lazy and I wanted to edit each file. I did a bit of searching and found that I can achieve this within `vim` itself.

First you start off with the following command:

```
:vim /search pattern/ **/*.txt
```

The search pattern can be a regular expression. The second parameter is a filename pattern matching. Once you run this command, it’ll open every file with an occurrence that it finds in “tabs” within vim.

Then you can navigate between each file using the following commands:

```
:cnext
:cprev
```

If you find yourself using this a lot,  can bind those commands to a key combination to make it easier to access. You can also partially type in the command like `:cn` or `:cp`.

Hope you find this useful! Reach out to me if you have any questions or want to chat!
