---
title: The Linux Terminal
layout: page
---

This post is about how I use the terminal in my personal and work machines, and how, with a few modifications, you can also change/improve your QoL when using it.

I have been using Linux for 10+ years, with zsh as shell for about 5 years. This post is not a definitive guide, but what I have found to be useful and nice to have in your personal shell. Even though I use zsh, it is very important to also be comfortable with bash, since it is the default shell in most linux servers, so you *will* have to deal with it.

## Things to consider on any environment

1. `Ctrl+r` is your best friend. Reverse search is extremely powerful, and after using it for a while it becomes second nature. Stop using arrows to search for that command that you ran the last month, use reverse search.

2. As mentioned before, learn the basics of bash and vim, most servers will have these programs available, and maybe zsh/nano will not be installed, so you need to be able to edit files and execute commands in those environments too. There are excellent tutorials and cheatsheets for these tools, so be sure to check them out.

3. Have you ever opened a file but forgot what you need to write? You can _pause_ a job by pressing `Ctrl+z`, check what you forgot, and then go back to what you were doing by executing `fg`.

4. The terminal is powerful, but there is no need to do *everything* on it, GUI applications are great tools, so don't drop them just to try to do everything in your terminal.

## My terminal

<figure class="video_container">
  <video controls="true" allowfullscreen="true" width="100%">
    <source src="/assets/terminal-demo/terminal-example1.webm" type="video/webm">
  </video>
</figure>

There are a lot of things happening in the video, I will try to explain what is happening in it and how you can achieve something similar.

The shell you see in the video is zsh. You might be wondering, why zsh instead of bash? Personal preference. There are lots of reasons why I recommend zsh instead of bash, for example, excellent plugins and you can handle any sh script; but in the end is just because I like it and I found a good configuration that fit my needs.

### Software Dependencies

Archlinux:

```sh
sudo pacman -S zsh fd fzf
<your aur command> -S antigen-git
```

### The run command file `.zshrc`

Everything begins with your own `~/.zshrc`. You can find mine in this [link](https://github.com/alevalv/configuration/blob/master/dotfiles/home/zshrc). The core of this rc file is [antigen](https://github.com/zsh-users/antigen), a plugin manager for zsh, think of it as an apt-get/pip/pacman for zsh plugins.

So how this `.zshrc` works?, let us go line by line:

```sh
source /usr/share/zsh/share/antigen.zsh
```

Loads the antigen file installed by `antigen-git`, you can also download it from the github repo and put in your home, then you can source with `source ~/antigen.zsh`

```sh
antigen bundle robbyrussell/oh-my-zsh /lib # pull only the library from oh-my-zsh, needed for some other dependencies

antigen bundle git
antigen bundle pip
antigen bundle command-not-found
antigen bundle micha/resty # rest client for zsh
```

Loads some common modules that are useful with zsh, they are pulled from oh-my-zsh.

```sh
antigen bundle zsh-users/zsh-syntax-highlighting
antigen bundle zsh-users/zsh-autosuggestions
antigen bundle zsh-users/zsh-history-substring-search

ZSH_AUTOSUGGEST_USE_ASYNC=true
bindkey '^[[A' history-substring-search-up
bindkey '^[[B' history-substring-search-down
```

These commands allow the console to autocomplete commands while you are writing them, and also does a search if you press the up/down arrows (similar to `Ctrl+R`):

<figure class="video_container">
  <video controls="true" allowfullscreen="true" width="100%">
    <source src="/assets/terminal-demo/terminal-example2.webm" type="video/webm">
  </video>
</figure>

```sh
antigen theme https://github.com/romkatv/powerlevel10k powerlevel10k
POWERLEVEL9K_PROMPT_ON_NEWLINE=true

antigen apply
```

The first two lines sets the prompt theme, (powerlevel10)[https://github.com/romkatv/powerlevel10k], and set it up to move the prompt to the next line. `antigen apply` executes and saves all antigen commands.

```sh
#fzf config
source /usr/share/fzf/key-bindings.zsh
source /usr/share/fzf/completion.zsh
export FZF_DEFAULT_COMMAND='fd --type f'
```

Lastly, here is the fzf configuration, [fzf](https://github.com/junegunn/fzf) is a command line fuzzy finder. The command `vim **` that was in the first video is using fzf internally, it will read all filenames in the current directory and allows the search of a file using fuzzy search. For example, if you need to open `main.rs`, you can copy some of the letters of the file(e.g. `m.rs`) which will filter the set of files. This functionality is common in IDEA based IDEs (Intellij, PyCharm, android-studio, etc.) with the search file command `Ctrl+Shift+N`, which I'm accustomed to.

```sh
alias ls='ls --color=auto'
alias cd..='cd ..'
```

The alias are for adding color to `ls` and correcting one common mistake that happens to me 😄.

There are more commands in my `.zshrc` but those are not related with the behavior of the shell, so they are ignored in this post.

That's all folks! Feel free to use any of the code mentioned here. I hope it can be useful for your shell adventures.
