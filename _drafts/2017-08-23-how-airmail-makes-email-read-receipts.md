---
layout: post
title: "How Airmail Makes Email Read Receipts"
description: "A brief exploration of how Airmail detects if a user reads
an email"
date: 2017-08-23
tags: [airmail, javascript, web, short]


comments: true
---

The other day, my friend sent me an email with Airmail. He showed me how he 
could set read receipts on his emails. This way, when he sends an email, he 
gets a notification when the recipient actually opens it, and it seems to 
work on most platforms (as far as I know, it at least works on Gmail and 
Apple's `Mail.app`). I wanted to figure out how it worked.

# Dissecting an Email

In order to view the source of this email, I opened it in `mutt`, a terminal 
mail client. It easily handles plaintext emails and it let me see the raw 
source of the email sent by Airmail, as well as all of the attachments 
associated with it. 

In this email we have:
    - a header
    - the email's contents
    - a script

The script is what we're interested in. 

