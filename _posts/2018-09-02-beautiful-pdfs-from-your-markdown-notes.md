---
layout: post
title: "Beautiful PDFs from Your Markdown Notes"
description: "A Brief Rundown of my Markdown Note Taking Setup"
date: 2018-09-02
tags: [markdown, pandoc, pdf, latex]
comments: true
---

This is a short post on how I write my notes in Markdown and convert them to
beautiful PDFs using [pandoc](https://pandoc.org) and the [eisvogel template](https://github.com/Wandmalfarbe/pandoc-latex-template).
I really like markdown - it's simple and has just enough formatting features
to be useful and expressive. It's quick, too. I would not write notes in
real-time with latex (way too slow), but it's perfectly doable with Markdown.

You can turn Markdown files into PDFs that look like this:

![formatted PDF paper](/assets/images/eisvogel_example.jpg)

## Setup

### Pandoc

First, you need to have pandoc installed. If you're on MacOS, you can use
[brew](https://brew.sh) to install pandoc:

```sh
brew install pandoc
```

If you're using [Haskell Stack](https://docs.haskellstack.org/en/stable/README/), you can install pandoc with:

```sh
stack install pandoc
```

On Linux distributions, you should be able to find pandoc in your package
manager.

### Eisvogel Template

Now that you have pandoc, you need to install the theme. Clone
[this repo](https://github.com/Wandmalfarbe/pandoc-latex-template) in a
location of your choosing. Now that you have the template, you need to install
it so pandoc can use the template.

Check to see if `~/.pandoc/templates` already exists on your computer. If not,
run `mkdir -p ~/.pandoc/templates`. Then you need to add the template to the
directory. I recommend creating a symlink rather than copying it, so you can
update the git repo and simply get template updates from there, rather than
having to add the extra step of copying it over and maintaining synchronization
yourself.

You can do this with:

```sh
ln -sf ./eisvogel.tex ~/.pandoc/templates/eisvogel.latex
```

If you want to use the eisvogel template by default whenever you convert
markdown to latex or pdf, you can run

```sh
ln -sf ./eisvogel.tex ~/.pandoc/templates/default.latex
```

## Usage

If you have the default template, then converting a markdown file to PDF with
the template is as simple as:

```sh
pandoc example.md -o example.pdf
```

If you want to select your theme, then simply run:

```sh
pandoc example.md -o example.pdf --template eisvogel
```

I recommend creating a Makefile for directories that you will store a lot of
markdown files in that you want to convert to PDF (such as a notes folder).
You can leave your commands to generate PDFs in the Makefile, so once you've
edited your files, you can simply run `make` and have your PDFs generated
without much fuss.

My makefile looks like this:
```make
.PHONY: all format

FILES := $(wildcard ./*.md)

all:
	for file in $(FILES); do \
		pandoc $$file -o $(basename $$file).pdf; \
	done

format:
	for file in $(FILES); do \
		prettier --write --prose-wrap always --print-width 80 $$file; \
	done
```

When I type `make`, my PDFs are generated from markdown files. The
`make format` command uses a tool called [prettier](https://github.com/prettier/prettier)
to format my Markdown files.
