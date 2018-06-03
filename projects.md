---
layout: page
permalink: /projects/
title: Projects
---

These are personal projects I am working on or have worked on. If you ever have
questions, please feel free to contact me to ask about them.

## [ensh](https://github.com/afnanenayet/Enayet-Shell)

Ensh stands for "Enayet Shell." It is a basic shell written in Rust, designed
to be modern and efficient. It is currently a stable viable shell for MacOS and
Linux. It uses Travis CI for testing and the Cargo package manager to manage
unit tests. At the time of writing (can't always promise I'll update this page
as soon as I update the project), the `ensh` binary size is only 160K, which I'm
working on paring down.

I originally started this project to learn Rust, and learn Rust as a
systems programming language. This project also taught me about process
management in Unix and a lot about maintaining and deploying a product.

## [cttp](https://github.com/afnanenayet/cttp)

A multithreaded HTTP 1.1 server written in C. I wrote this to get a better handle
on BSD sockets and because I haven't really written any code that deals with
networking. The server can handle a decent number of mime types, and functions
as a fairly basic HTTP server. It can receive `GET` requests to a particular path,
and will serve the file from that path to a browser.
The server is also multithreaded and can accept multiple connections at once.

`cttp` uses the `pthreads` library and [cmocka](http://cmocka.org)
to manage unit testing. Travis CI is used for deployment and testing.

## [afnan.io](http://afnan.io)

A personal website/blog powered by Jekyll and Travis CI. Pushing to
Github triggers a build from Travis, which ensures that the website
builds properly. You're looking at it right now!

## Matasano Cryptopals Challenge

I am working on the Matasano Cryptopals Challenge, a set of challenges designed
to teach some of the basics of cryptography. The challenges can be found
[here](https://cryptopals.com). My solutions, written in C++14, can be found
[here](https://github.com/afnanenayet/Cryptopals_Challenge).

## tiny search engine (TSE)

For a software development class, we created a small search engine in C.
It downloads the contents of a webpage, indexes it, and ranks the results
of a boolean query and displays it to the user. It follows the Unix development
philosphy. At the instruction of our professor and the College, the source code
cannot be made public, but I can provide the source code upon inquiry.
Email me if you want to see the source.

## Android apps

### FreeLoop

An app that replicates a guitar looper pedal for free. Available on
the [Play Store](https://play.google.com/store/apps/details?id=com.enayet.loopr).
Has over 6,000 downloads.

### [21.Days](https://github.com/afnanenayet/21.Days)

Created with three other students at Dartmouth College, 21.Days is designed
to help users consistently build a habit. It utilizes Firebase and good app
design practices.

### Minigma

An encryption app that utilizes Android intents to allow a user to send encrypted
messages through any app on their phone. Download on the
[Play Store](https://play.google.com/store/apps/details?id=com.enayet.minigma).

### Battery Informatics

Provides diagnostic information about a phone's battery, including charge
capacity, temperature, and rate of discharge. Download on the
[Play Store](https://play.google.com/store/apps/details?id=com.enayet.powinfo).

### [MyRuns](https://github.com/afnanenayet/MyRuns6)

A fitness/run tracker app created for a computer science class at Dartmouth College.
Encourages good development practices, modularity, and MVC. Uses a variety of
Google/Android services like location, accelerometer, and is multithreaded.
Also uses machine learning to infer the type of activity a user is carrying out.
