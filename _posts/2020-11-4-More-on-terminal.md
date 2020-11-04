---
layout: post
title: PyTorch
subtitle: A reading note for pytorch source code 
thumbnail-img: /assets/img/pytorch-logo.png
cover-img: /assets/img/202011/1.png
share-img: /assets/img/202011/1.png
tags: [ note, terminal]
comments: false
---

Is there only me, or you also think it's awesome to do somethings seem unrelevant to the terminal.

I want to intro a new theme of `zsh` , and focus on a plugin of this theme. If you haven't change your `bash` to `zsh`, do it now(There is a nice [tutor](https://www.howtoforge.com/tutorial/how-to-setup-zsh-and-oh-my-zsh-on-linux/))! `zsh` can offer better interactive experiences than `bash`.

The theme is called `powerlevel10k`, see more details in [this repo](https://github.com/romkatv/powerlevel10k). The repo offers many ways to install  `powerlevel10k`  no matter which os you're using. My base is MacOS in the following. 

![performance](https://raw.githubusercontent.com/romkatv/powerlevel10k-media/master/performance.gif)



Where you're done install, I believe you already used the command `p10k configure`  to setup some basic appearence. But let's look deeper. 

```shell
cat ~/.p10k.zsh
```

Look at the first block seems configureable

```
# The list of segments shown on the left. Fill it with the most important segments.
  typeset -g POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(
    # =========================[ Line #1 ]=========================
    os_icon               # os identifier
    dir                     # current directory
    vcs                     # git status
    # =========================[ Line #2 ]=========================
    newline                 # \n
    prompt_char             # prompt symbol
  )
```

> *NOTICE*: this block maybe slightly different with yours, because I screenshot it from my own `.p10k.zsh`.

We can see the name of the variable is `POWERLEVEL9K_LEFT_PROMPT_ELEMENTS`, which indicates this variable config the left prompt of the terminal:

![left](/assets/img/202011/left.png)

We can see

* 🚀 should be the `os_icon` segment
* `~/Desktop/Github` should be the `dir` segment
* `master` corresponding to the `vcs`, which stands for `git status`
* `newline` - `\n`
* `➜` should be the `prompt_char` 

Again, no worry about the above image or config is not the same as your's right now. We  will see why.

So, can we config those segments? Of course, use any code editor to open `~/.p10k.zsh`. 

* search `os_icon`

We can actually find a block

```
#################################[ os_icon: os identifier ]##################################
  # OS identifier color.
  typeset -g POWERLEVEL9K_OS_ICON_FOREGROUND=
  # Custom icon.
  typeset -g POWERLEVEL9K_OS_ICON_CONTENT_EXPANSION='🚀'
```

Here it is! This block pretty much said for itself. One more word, `*_FOREGROUND` always is the variable for color, and `*_EXPANSION` is something will actual display in terminal.

* search `prompt_char`

```shell
################################[ prompt_char: prompt symbol ]################################
  # Green prompt symbol if the last command succeeded.
  typeset -g POWERLEVEL9K_PROMPT_CHAR_OK_{VIINS,VICMD,VIVIS,VIOWR}_FOREGROUND=yellow
  # Red prompt symbol if the last command failed.
  typeset -g POWERLEVEL9K_PROMPT_CHAR_ERROR_{VIINS,VICMD,VIVIS,VIOWR}_FOREGROUND=red
  # Default prompt symbol.
  typeset -g POWERLEVEL9K_PROMPT_CHAR_{OK,ERROR}_VIINS_CONTENT_EXPANSION='%B➜'
  # Prompt symbol in command vi mode.
  typeset -g POWERLEVEL9K_PROMPT_CHAR_{OK,ERROR}_VICMD_CONTENT_EXPANSION='<'
  # Prompt symbol in visual vi mode.
  typeset -g POWERLEVEL9K_PROMPT_CHAR_{OK,ERROR}_VIVIS_CONTENT_EXPANSION='V'
  # Prompt symbol in overwrite vi mode.
  typeset -g POWERLEVEL9K_PROMPT_CHAR_{OK,ERROR}_VIOWR_CONTENT_EXPANSION='^'
  typeset -g POWERLEVEL9K_PROMPT_CHAR_OVERWRITE_STATE=true
  # No line terminator if prompt_char is the last segment.
  typeset -g POWERLEVEL9K_PROMPT_CHAR_LEFT_PROMPT_LAST_SEGMENT_END_SYMBOL=''
  # No line introducer if prompt_char is the first segment.
  typeset -g POWERLEVEL9K_PROMPT_CHAR_LEFT_PROMPT_FIRST_SEGMENT_START_SYMBOL=
```

A little messy, but we can still find some clues

* `POWERLEVEL9K_PROMPT_CHAR_OK_{VIINS,VICMD,VIVIS,VIOWR}_FOREGROUND` the color of the prompt when "OK", which means if the command is accepted, then the color will be yellow
* same as the `POWERLEVEL9K_PROMPT_CHAR_ERROR_{VIINS,VICMD,VIVIS,VIOWR}_FOREGROUND`. When the command isn't accepted, the color will turn red.
* The thrid variables says for itself, the `%B➜` means bold `➜`.

We now have a understand of how to configure `~/.p10k.zsh` . In a word, this file provide a easy to set up your left prompt, as well as the right prompt. I give you how I set up my right prompt

```shell
typeset -g POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(
    # =========================[ Line #1 ]=========================
    status                  # exit code of the last command
    command_execution_time  # duration of the last command
    background_jobs         # presence of background jobs
    direnv                  # direnv status (https://direnv.net/)
    # asdf                    # asdf version manager (https://github.com/asdf-vm/asdf)
    # virtualenv              # python virtual environment (https://docs.python.org/3/library/venv.html)
    anaconda                # conda environment (https://conda.io/)
    # pyenv                   # python environment (https://github.com/pyenv/pyenv)
    # goenv                   # go environment (https://github.com/syndbg/goenv)
    # nodenv                  # node.js version from nodenv (https://github.com/nodenv/nodenv)
    # nvm                     # node.js version from nvm (https://github.com/nvm-sh/nvm)
    # nodeenv                 # node.js environment (https://github.com/ekalinin/nodeenv)
    # node_version          # node.js version
    # go_version            # go version (https://golang.org)
    # rust_version          # rustc version (https://www.rust-lang.org)
    # dotnet_version        # .NET version (https://dotnet.microsoft.com)
    # php_version           # php version (https://www.php.net/)
    # laravel_version       # laravel php framework version (https://laravel.com/)
    # java_version          # java version (https://www.java.com/)
    # package               # name@version from package.json (https://docs.npmjs.com/files/package.json)
    # rbenv                   # ruby version from rbenv (https://github.com/rbenv/rbenv)
    # rvm                     # ruby version from rvm (https://rvm.io)
    # fvm                     # flutter version management (https://github.com/leoafarias/fvm)
    # luaenv                  # lua version from luaenv (https://github.com/cehoffman/luaenv)
    # jenv                    # java version from jenv (https://github.com/jenv/jenv)
    # plenv                   # perl version from plenv (https://github.com/tokuhirom/plenv)
    # phpenv                  # php version from phpenv (https://github.com/phpenv/phpenv)
    # scalaenv                # scala version from scalaenv (https://github.com/scalaenv/scalaenv)
    # haskell_stack           # haskell version from stack (https://haskellstack.org/)
    # kubecontext             # current kubernetes context (https://kubernetes.io/)
    # terraform               # terraform workspace (https://www.terraform.io)
    # aws                     # aws profile (https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html)
    # aws_eb_env              # aws elastic beanstalk environment (https://aws.amazon.com/elasticbeanstalk/)
    # azure                   # azure account name (https://docs.microsoft.com/en-us/cli/azure)
    # gcloud                  # google cloud cli account and project (https://cloud.google.com/)
    # google_app_cred         # google application credentials (https://cloud.google.com/docs/authentication/production)
    # context                 # user@hostname
    # nordvpn                 # nordvpn connection status, linux only (https://nordvpn.com/)
    # ranger                  # ranger shell (https://github.com/ranger/ranger)
    # nnn                     # nnn shell (https://github.com/jarun/nnn)
    # vim_shell               # vim shell indicator (:sh)
    # midnight_commander      # midnight commander shell (https://midnight-commander.org/)
    # nix_shell               # nix shell (https://nixos.org/nixos/nix-pills/developing-with-nix-shell.html)
    # vpn_ip                # virtual private network indicator
    # load                  # CPU load
    # disk_usage            # disk usage
    # ram                   # free RAM
    # swap                  # used swap    
    # timewarrior             # timewarrior tracking status (https://timewarrior.net/)
    # time                  # current time
    # =========================[ Line #2 ]=========================
    newline                 # \n
    # todo                    # todo items (https://github.com/todotxt/todo.txt-cli)
    taskwarrior             # taskwarrior task count (https://taskwarrior.org/)
    # ip                    # ip address and bandwidth usage for a specified network interface
    # public_ip             # public IP address
    # proxy                 # system-wide http/https/ftp proxy
    # battery               # internal battery
    # wifi                  # wifi speed
    # example               # example user-defined segment (see prompt_example function below)
  )
```

You can see, it's exciting the `powerlevel10k` provides segments for so many command. But I don't want my shell grow up to messy and heavily. So I only use segments `anaconda` and `taskwarrior`. The latter is a great command to manage your task on terminal. 

And I update the segment of `taskwarrior` a little.

```shell
##############[ taskwarrior: taskwarrior task count (https://taskwarrior.org/) ]##############
  # Taskwarrior color.
  typeset -g POWERLEVEL9K_TASKWARRIOR_FOREGROUND=yellow
  
  # Taskwarrior segment format. The following parameters are available within the expansion.
  #
  # - P9K_TASKWARRIOR_PENDING_COUNT   The number of pending tasks: `task +PENDING count`.
  # - P9K_TASKWARRIOR_OVERDUE_COUNT   The number of overdue tasks: `task +OVERDUE count`.
  #
  # Zero values are represented as empty parameters.
  #
  # The default format:
  #
    # '${P9K_TASKWARRIOR_OVERDUE_COUNT:+"!$P9K_TASKWARRIOR_OVERDUE_COUNT/"}$P9K_TASKWARRIOR_PENDING_COUNT'
  #
  typeset -g POWERLEVEL9K_TASKWARRIOR_CONTENT_EXPANSION='`task due.before:1wk count`/`task count`'
```

Now my terminal will look like 

![total1](/assets/img/202011/total1.png)

**Tips**: if you think the little `py` and `task` in the right prompt is annoying, you can set

```
typeset -g POWERLEVEL9K_ANACONDA_VISUAL_IDENTIFIER_EXPANSION=''
...
typeset -g POWERLEVEL9K_TASKWARRIOR_VISUAL_IDENTIFIER_EXPANSION=' '
```

