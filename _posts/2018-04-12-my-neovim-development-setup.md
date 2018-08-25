---
layout: post
title: "My Neovim Development Setup"
description: "An update to my last neovim guide"
date: 2018-04-12
tags: [neovim, vim, deoplete, dein, cquery, language, server]
comments: true
---

It's been a while since I wrote about my Neovim setup. Since my last post, my
`nvim` config has grown to be a little more sophisticated, and I finally
worked out autocompletion and linting for all of the languages I work with.

Here's what my editor looks like:

![neovim c++](/assets/images/nvim_0.png)

![neovim rust](/assets/images/nvim_1.png)

I have posted my full neovim configuration on [Github](https://github.com/afnanenayet/nvim-dotfiles)

## Split up your `init.vim`

I had a horribly long `init.vim` file before. It gets clunky to manage and
long files are ugly. It's highly recommended that you split up your `init.vim`
files into more manageable chunks. The way you can do this is by sourcing
each chunk in your main init.vim file. Suppose we have a file for our deoplete
settings, and another file for our language client settings. Our `init.vim`
file could look something like this:

```vim
source $HOME/.config/nvim/config/deoplete.vim
source $HOME/.config/nvim/config/lc.vim
```

And in `config/lc.vim` and `config/deoplete.vim`, you could have whatever
settings pertaining to each plugin that you want. I personally have my config
files split up in the following manner:

```
config
├── deoplete.vim
├── init.vim
├── keybindings.vim
├── language_client.vim
├── plugins.vim
└── powerline.vim
```

My `init.vim` file looks like this:

```vim
source $HOME/.config/nvim/config/plugins.vim
source $HOME/.config/nvim/config/init.vim
source $HOME/.config/nvim/config/powerline.vim
source $HOME/.config/nvim/config/deoplete.vim
source $HOME/.config/nvim/config/language_client.vim
source $HOME/.config/nvim/config/keybindings.vim
```

_Note_: You can also use a glob pattern to automatically source every file
in a folder, as such:

```vim
for f in split(glob('~/.config/nvim/config/*.vim'), '\n')
    exe 'source' f
endfor
```

_Credit_: Thanks to commentor Igor Epstein for this suggestion!

Note that I had to put the `plugins.vim` file first because it contains
everything pertaining to my package manager, dein, and dein has some
scripts it needs to run first in Neovim. The order that you put these
scripts in does matter, as they effectively will be set up in order.
Everything in `plugins.vim` will be run before everything in the other
files in my case. `init.vim` files can get messy, and this offers a
way to clean up your setup and separate your neovim/vim config into
logical units.

## Managing packages

There are a lot of package managers for Neovim/vim out there. People
have different preferences, I have no strong feelings one way or the
other. I personally use [dein](https://github.com/Shougo/dein.vim), which has served me pretty well. It uses
all of the great async features in Vim 8/Neovim and is pretty easy to
use.

My `plugins.vim` file has all of the settings pertaining to dein, as
well as the dein startup scripts that it needs to run when vim/nvim
boots up.

After setting it up, installing plugins is as easy as:

```vim
call dein#install("github-username/repo-name")
```

Note that it doesn't automatically update plugins by default. I manually
update my plugins. You can just call `:call dein#update()` in Neovim, and
dein will upgrade all of your plugins for you.

## Autocompletion and linting

I use [deoplete](https://github.com/Shougo/deoplete.nvim)
for completion in Neovim. Deoplete itself is just an autocompletion engine,
you need to install sources for it to be able to give you suggestions relevant
to whatever you're working on. I primarily use language servers for autocompletion,
but there are also non-language server options that work perfectly fine with
deoplete.

### Configuring deoplete

Deoplete is powered by completion sources, and it has a few default ones that
really annoyed me. For example, buffer based completion was fairly unhelpful
for me. I don't want autocompletion in a markdown or text file, and I only
want to have completion from meaningful symbols. You can set the completion
sources for deoplete, and also have completion enabled on a per-buffer
basis.

I have deoplete completion disabled by default

```vim
" disable autocomplete by default
let b:deoplete_disable_auto_complete=1
let g:deoplete_disable_auto_complete=1
```

I also set sources to be empty by default

```vim
let g:deoplete#sources = {}
```

And lastly, I don't want any autocompletion when I'm writing strings and
comments.

```vim
" Disable the candidates in Comment/String syntaxes.
call deoplete#custom#source('_',
            \ 'disabled_syntaxes', ['Comment', 'String'])
```

You can set sources per language like this

```vim
let g:deoplete#sources.python = ['LanguageClient']
```

and replace `python` which whatever language you want.

### Completion sources and language servers

If you want to use a language client, I would suggest using the
[LanguageClient-neovim](https://github.com/autozimu/LanguageClient-neovim)
plugin. Note that this source is called `LanguageClient`, so if you
want to use a language server for a language, you need to set it up
for the LanguageClient plugin, then set the source for that language
to be `LanguageClient`. One thing that tripped me up when trying to
set up sources for deoplete was that I didn't find good documentation
for what each plugin is named. For example, `deoplete-rust` uses `racer`,
but the name of the source is `rust`, but `deoplete-jedi` has a source
called `jedi`.

Here are some sources/language servers you can use (I've included the
names of the sources for entries that aren't language servers).

- python:
  - [deoplete-jedi](https://github.com/zchee/deoplete-jedi) (source) `jedi`
  - [pyls](https://github.com/palantir/python-language-server) (language server)
- rust:
  - [deoplete-rust](https://github.com/sebastianmarkow/deoplete-rust) (source) `rust`
  - [rls](https://github.com/rust-lang-nursery/rls/) (language server)
- C/C++:
  - [deoplete-clangx](https://github.com/Shougo/deoplete-clangx) (source) `clangx`
  - [cquery](https://github.com/cquery-project/cquery) (language server) note that I've found this to be the most effective solution for large projects
  - [clangd](https://clang.llvm.org/extra/clangd.html) (language server) `clang`

## Other plugins

Some of my other favorite plugins are:

- [NERD tree](https://github.com/scrooloose/nerdtree): a nice hierarchical file browser
- [fzy](https://github.com/jhawthorn/fzy): a fast fuzzy file finder I use as to search for files and for text within files
- [vim airline](https://github.com/vim-airline/vim-airline/): a lean powerline plugin for vim
