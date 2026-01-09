---
layout: post
title: "How I use fzf with NeoMutt"
date: 2021-04-02
tags: [neomutt, fzf]
---

I use [NeoMutt](https://neomutt.org/) to read my email. And I use it a lot.
As a result I have many different mailboxes. NeoMutt comes with a nice feature
called [sidebar](https://neomutt.org/feature/sidebar) which does what you
think it does - it splits your terminal screen and displays a list of all your
mailboxes in one of the split windows. This is great because now you can see
all your mailboxes and you can easily navigate through them to select the
mailbox you want to read. Alternatively you can also use the NeoMutt *change
folder* command and then enter the name of the mailbox you'd like to change
to.

The problem though is, the more mailboxes you have the longer it takes to
navigate through the list. Manually entering the name of the mailbox is also
not exactly want I want to do. There are a couple of key bindings you can use
to make the navigation easier, but I was still looking for a more elegant
solution.

<script src="https://gist.github.com/tscherf/d46b822c60c60c40a2544233802dcac5.js?file=neomutt_sidebar_bindings.md"></script>

There is another tool I use a lot on the terminal - called
[fzf](https://github.com/junegunn/fzf). It's a general-purpose command-line
fuzzy finder. I mostly use it to quickly navigate through my zsh history. And
that actually brought me to the idea why not combine fzf with NeoMutt.

I download all my mail from an IMAP server using
[mbsync](https://isync.sourceforge.io/mbsync.html) and then store it in a
local Maildir folder. I use the following script to generate a list of
all the mailboxes that I can pipe as input into fzf:

<script src="https://gist.github.com/tscherf/d46b822c60c60c40a2544233802dcac5.js?file=mutt_fzffolder.md"></script>

At the end of the script I have to make sure that the fzf result is passed to
the *change folder* command in NeoMutt which then let me fuzzy search through
the list of mailboxes.

The only missing piece is to add a macro into the *muttrc* configuration file
to call the *fzffolder* script:

<script src="https://gist.github.com/tscherf/d46b822c60c60c40a2544233802dcac5.js?file=muttrc_macro_fzf.md"></script>

When I now press *space* in the NeoMutt index and start typing some string
that is part of the mailbox I'd like to navigate to, fzf immediately brings me
to the right folder and I can start reading my mail.




