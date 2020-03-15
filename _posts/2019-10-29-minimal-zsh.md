---
title: A Minimalistic Approach to ZSH
toc: true
comments: true
layout: post
description: Up and running with z-shell in no time
categories: [linux]
---

![Imgur](https://imgur.com/OOe4F0e.png)

Although I'm on Debian Linux, Apple's recent announcement about replacing Bash with Zsh on MacOS made me take a look at Z-shell aka zsh. It's a [POSIX](https://en.wikipedia.org/wiki/POSIX) compliant Bash alternative that has been around for quite a long time. While Bash shell's efficiency and ubiquity make it hard to think about changing the default shell of your primary development machine, I find its features as an interactive shell to be somewhat limited. So I did some digging around and soon found out that zsh's lackluster default configurations and bloated ecosystem make it difficult for someone who just want to switch without any extra overhead. So, let's make the process quicker. Here is what we are aiming for:

* A working shell that can (almost always) take bash commands without complaining (looking at you fish)
* History based autocomplete
* Syntax highlighting
* Git branch annotations

Instructions were applied and tested on debian based linux (Ubuntu)

## Install Z Shell

### GNU/Linux
To install on a debian based linux, type:

``` bash
$ apt install zsh
```

### MacOS

Use homebrew to install zsh on MacOs. Run:

``` bash
$ brew install zsh
```

## Make Zsh as Your Default Shell

Run:

``` bash
$ chsh -s $(which zsh)
```

## Install Oh-My-Zsh Framework

[Oh-My-Zsh](https://github.com/robbyrussell/oh-my-zsh) is the tool that makes zsh so much fun and overly configurable at the same time. So we'll tread here carefully. To install `oh-my-zsh` , type:

``` bash
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

## Set Firacode As the Default Terminal Font

Your selected theme may not display all the glyphs if the default terminal font doesn't support them. Installing a font with glyphs and ligature support can solve this. I recommend installing `firacode` and setting that as your default terminal font. Install Fira Code From [here.](https://github.com/tonsky/FiraCode)

## Set Syntax Highlighting

Using [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) to achieve this.

* Clone this repository in oh-my-zsh's plugins directory

  ```bash
  $ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
  ```

* Activate the plugin in `~/.zshrc`

  ```bash
  plugins=( [plugins...] zsh-syntax-highlighting)
  ```

* Source `~/.zshrc`

## Set Suggestions

Using [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) to achieve this.

* Clone this repository into $ZSH_CUSTOM/plugins (by default ~/.oh-my-zsh/custom/plugins)

  ```bash
  $ git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
  ```

* Add the plugin to the list of plugins for Oh My Zsh to load (inside `~/.zshrc` )

  ```bash
  plugins=(zsh-autosuggestions)
  ```

* Source `~/.zshrc`

Start a new terminal session to see the effects!!! You might need to log out and log in again for the changes to be effective.

## Load .profile from .zprofile

Add the following lines to `~/.zprofile` and source via the command:
`source ~/.zprofile`. Make sure you are on zsh before running the source command.

  ```bash
  [[ -e ~/.profile ]] && emulate sh -c 'source ~/.profile'
  ```

## A Barebone ~/.zshrc

Instead of adding the plugins individually, you can just install the plugins and then add this barebone config to your `~/.zshrc` . Don't forget to replace `YourUserName` with your username. Source your zshrc once you are done.

```zsh
# =====================
# MINIMALIST ZSHRC
# AUTHOR: REDNAFI
# =====================

# omz path
export ZSH="$HOME/.oh-my-zsh"

# theme settings
setopt PROMPT_PERCENT
setopt PROMPT_SUBST
PROMPT='[%n:%F{yellow}%25<..<%~%f%<<]$ '
precmd() {
    precmd() {
        #echo
    }
}

# pluging settings
plugins=(git zsh-syntax-highlighting zsh-autosuggestions)

# autosuggestion highlight
ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE="fg=4"

# source omz
source $ZSH/oh-my-zsh.sh

#History setup
HISTFILE=$HOME/.zsh_history
HISTSIZE=100000
SAVEHIST=$HISTSIZ

zstyle ':completion:*' menu select
zstyle ':completion:*' group-name ''                                        # group results by category
zstyle ':completion:::::' completer _expand _complete _ignored _approximate #enable approximate matches for completion

#disable auto correct
unsetopt correct_all

# add snap path
export PATH=$PATH:/snap/bin
```

## Set Terminal Color (Optional)
Optionally you customize your terminal color and in this case I've used [Gogh](http://mayccoll.github.io/Gogh) to achieve this.

  * Pre Install

  ```bash
  $ sudo apt-get install dconf-cli uuid-runtime
  ```

  * Install on Linux

  ```bash
   $ bash -c  "$(wget -qO- https://git.io/vQgMr)"
  ```

  * Install on MacOS
  ```bash
  $  bash -c  "$(curl -sLo- https://git.io/vQgMr)"
  ```

  * Put the code associated with your desired color scheme.
  ![alt](https://raw.githubusercontent.com/Mayccoll/Gogh/master/images/demos/gogh-demo-profile.gif)


## Updating OMZ

```bash
$ upgrade_oh_my_zsh
  ```

## Uninstall Zsh

```bash
$ sudo apt-get --purge remove zsh
```

## Uninstall OMZ

```bash
$ uninstall_oh_my_zsh
```

## Switch Back to Bash

```bash
$ chsh -s $(which bash)
```

## Reference
1. [Oh-My-Zsh](https://ohmyz.sh/)
2. [FiraCode](https://github.com/tonsky/FiraCode)
3. [Gogh](https://github.com/Mayccoll/Gogh)
