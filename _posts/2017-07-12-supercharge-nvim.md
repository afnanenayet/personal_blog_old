---
layout: post
title: "Setting up Neovim"
description: "Tutorial on setting up Neovim with a plugin manager and various helpful plugins"
share: true
comments: true
tags: [neovim, vim, C++, C, dein, plugins]
category: posts
---

_Note_: You can find an updated version of this article [here](http://afnan.io/2018-04-12/my-neovim-development-setup/)

---

Vim is an excellent text editor. It's fast and has a light footprint, and tends
to be installed in most Unix systems you'll come across. You can use it across
SSH, and once you get the hang of the keybindings, you'll find that it's
actually very fast to use.

[Neovim](https://neovim.io) tries to strip some of the cruft of Vim. It's
completely asynchronous (though Vim also introduced async in Vim 8), so
plugins shouldn't block normal operations. It includes nice features like
everything that's found in the [vim-sensible](https://github.com/tpope/vim-sensible)
plugin and true 24-bit color support.

Of course, you might be missing some goodies from goodies that you had in
your IDE, I won't deny that Visual Studio and CLion have some convenient
goodies like autocompletion and formatting.

There are some very powerful autocompletion plugins available for Neovim and Vim
(I will specifically be discussing my Neovim setup).

## Setup

Installing Neovim is relatively simple. If you're on a mac, you can install it
with Hombrew, or you can use `apt`. Instructions are provided on the website.

The autocompletion plugins we will be using utilize LLVM/Clang. You can download
the latest LLVM source from their [website](http://llvm.org). You can also get the
compiler from Homebrew and other package managers.

If you want Python 2/3 support, you will want to install the Neovim pip package:

```shell
pip install neovim
pip3 install neovim
```

For the rest of this tutorial, you may want to check out my
[dotfiles](https://github.com/afnanenayet/dotfiles)
for some inspiration on how to customize your Vim/Neovim setup.

## dein: a neovim package manager

Installing packages in both Vim and Neovim is best done with a package manager.
A package manager that was specifically made for Neovim is [dein](https://github.com/Shougo/dein.vim)
a "dark" package manager for Neovim. The instructions for installation are
available on the dein Github page, which has been linked above.

The dein instructions want you to specify an installation directory when
you run the installer script. I ended up using

```bash
./installer.sh ~/.config/nvim/bundle/repos
```

So I could easily manage dein as with any other plugin.

Your plugin base directory will probably be: `~/.config/nvim/bundle`.

You will probably also want to use an absolute path rather than a relative
path like I posted above

### Installing plugins

Installing plugins with dein is extremely easy, you simply add a line in
your `init.vim` file. After the `dein.begin` line, add another line with
the format

```vimscript
call dein#add('github_username'/'project_name')
```

This will prompt dein to clone the repo into your `bundle/repos` directory.
Dein will automatically pull from Github to keep your plugins updated. If
you want to remove a plugin, simply delete the line that specifies that plugin
and Dein will remove it for you.

### Recommended plugins

These are the plugins I use for autocompletion:

- [deoplete](https://github.com/Shougo/deoplete.nvim): the actual autocompletion engine. Note that it won't handle
  your C/C++ files on its own. It's just a completion framework, and it will
  need to pull data from somewhere...
- [clang-complete](https://github.com/Rip-Rip/clang_complete): this is the
  part that actually pulls data from clang to power the autocompletion engine.
  For this, you will need to have LLVM installed, and you will need to specify
  the LLVM installation path
- [deoplete-rust](https://github.com/sebastianmarkow/deoplete-rust):
  This contains the deoplete completions for the Rust programming language. It
  requires you to download the Rust source code and documentation.
- [deoplete-jedi](https://github.com/zchee/deoplete-jedi):
  This contains completions for Python using deoplete
- [vim-polyglot](https://github.com/sheerun/vim-polyglot): a language pack for
  Neovim/Vim which contains rules for many languages. It allows for more robust
  syntax highlighting.

## Usage

Verify that Neovim is working properly by running `:CheckHealth` to ensure that
your Python and Ruby plugins are working properly (if you wanted a Python or
Ruby plugin). This will also check to see if Dein is reporting any issues.

Using Neovim with these plugins is relatively easy - you just type and autocomplete
suggestions will pop up if deoplete finds a potential match. Using your arrow keys
to navigate these suggestions can bring up function previews. The repos for these
plugins contain instructions for customizing the plugins.
