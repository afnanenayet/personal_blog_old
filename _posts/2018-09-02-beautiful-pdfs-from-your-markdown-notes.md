---
layout: post
title: "Beautiful PDFs from Your Markdown Notes"
description: "A brief rundown of my markdown note taking setup"
date: 2018-09-02
tags: [markdown, pandoc, pdf, latex, eisvogel, academic, paper, note, taking]
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

If you want to use bibtex for more elaborate papers, you will need to install
the `pandas-citeproc` package, which can be found on Homebrew.

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

### Basic

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
        pandoc $$file -o $(basename -s ".md" $$file).pdf; \
    done

format:
    for file in $(FILES); do \
        prettier --write --prose-wrap always --print-width 80 $$file; \
    done
```

When I type `make`, my PDFs are generated from markdown files. The
`make format` command uses a tool called [prettier](https://github.com/prettier/prettier)
to format my Markdown files.

There are a lot of configurable options both in pandoc and included with the
latex template that are documented [here](https://pandoc.org/MANUAL.html#variables-for-latex<Paste>) for pandoc and
[here](https://github.com/Wandmalfarbe/pandoc-latex-template#custom-template-variables)
for the template.

### Citations

Bibtex is a popular tool for managing bibliographies in latex. Luckily,
we can leverage bibtex and still do most of our writing in markdown, and use
pandoc to incorporate bibtex bibliographies.

This assumes you're familiar with bibtex, or at least have some tool that can
generate bibtex bibliographies. A great, free, and open source tool is
[Zotero](https://www.zotero.org). If you just want a lightweight web interface to generate
bibliographies, you can go to: https://zbib.org

Now that you have your `.bib` file, you can optionally grab a `.csl` style
file. It's an XML file that defines how bibliography entries will be formatted
in your document. This makes it easy to switch formats from say, APA, to MLA.
You don't _have_ to supply a style file, because there is a default style that
will be applied, but if you want to use a particular style, you can find many
`.csl` files for common citation formats [here](https://github.com/citation-style-language/styles). _Warning: this is a massive
repository._

Now that you have your `.bib` bibliography, and your `.csl` style file, you
are all set to generate a bibliography from a markdown file (let's assume you
have some file called `example.md`).

You can generate PDF output via:

```sh
pandoc example.md -t latex -s --filter pandoc-citeproc \
    --bibliography=citations.bib --csl=style.csl -o example.pdf
```

Note that you can also specify the bibliography file in the markdown file's
YAML front matter, like so:

```markdown
---
bibliography: citations.bib
...

# Example markdown file

Hello, world!
```

Either way works.

If you run the command, you may be confused to find that your bibliography
hasn't actually shown up. This is because you haven't used any references.
This tool will only show citations that are actually cited in the paper.

Luckily, the syntax for citing bibtex entries in markdown is simple. In
bibtex, every entry has a short name, which is what you'll refer to in
your citation syntax. This later will be expanded to the proper format as
defined by your `csl` file.

Suppose we have some bibtex file that looks like this:

```bibtex
@article{example_citation,
    ...
}
```

The name you'll reference is `example_citation`, and you can use this citation
like such:

```markdown
# Example paper

Afnan deserves a higher salary [@example_citation, pp. 196].
```

If you just want to cite the paper without the page number, omit `, pp. 196`.
If you want to cite a chapter you can use: `[@example_citation, ch. 5]`.

---

_Note: this article was edited on 05 Sep 2018 with information about how to
use bibtex with the eisvogel template._
