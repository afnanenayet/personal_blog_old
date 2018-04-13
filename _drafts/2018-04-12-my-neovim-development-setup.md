---
layout: post
title: "My Neovim Development Setup"
description: "An updated to my "
date: 2018-04-12
tags: [neovim, vim, deoplete, dein, cquery, language, server]
comments: true
---

It's been a while since I wrote about my Neovim setup. Since my last post, my
`nvim` config has grown to be a little more sophisticated, and I finally
worked out autocompletion and linting for all of the languages I work with.

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

Note that I had to put the `plugins.vim` file first because it contains
everything pertaining to my package manager, dein, and dein has some
scripts it needs to run first in Neovim.
