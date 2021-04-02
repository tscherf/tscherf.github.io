---
layout: post
title: "How I combine fzf with NeoMutt"
date: 2021-04-02
tags: [neomutt, fzf]
---

I use [NeoMutt](https://neomutt.org/) to read my email. And I use it a lot.
For personal and for work email. As a result I have many different mailboxes.
NeoMutt comes comes with a nice feature called
[sidebar](https://neomutt.org/feature/sidebar) which does what you think it
does - it splits your terminal screen and provides you with a list of all your
mailboxes in one of the split windows. This is great because now you can
see all your mailboxes and you easily navigate through them to select the
mailbox you want to read.

Now, the problem is, the more mailboxes you have the longer it takes to
navigate through the list. There are a couple of key bindings you can use to
make the navigation easier, but I was still looking for a more elegant
solution.

<script src="https://gist.github.com/tscherf/d46b822c60c60c40a2544233802dcac5.js?file=neomutt_sidebar_bindings.md"></script>

There is another tool I use a lot in the terminal - called
[fzf](https://github.com/junegunn/fzf). It's a general-purpose command-line
fuzzy finder. I mostly use it to quickly navigate through my zsh history. And
that actually brought me to the idea why not combine `fzf` with `NeoMutt`.

tbc...





