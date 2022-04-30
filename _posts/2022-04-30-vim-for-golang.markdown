---
layout: post
title:  "Install Go plugin for Vim"
date:   2022-04-30 15:58:13 +0300
categories: vim golang
---
# Go development environment for Vim
In this article I will describe steps on how to set up the Go development environment for Vim. For that you need pre installed the following tools:

* Go compiler version 1.16+
* Vim version 8.0.1453+

Vim 8.0.1453+ supports plugin loading so we will use this feature to setup Go plugin.

Create the directory which is required by Vim to auto load plugins:
```
$ mkdir -p ~/.vim/pack/plugins/start
```

Clone the `vim-go` plugin to the directory created on the previous step:
```
$ git clone https://github.com/fatih/vim-go.git ~/.vim/pack/plugins/start/vim-go
```

As the authors of the `vim-go` say:
>The latest stable release is the recommended version to use. If you choose to use the master branch instead, please do so with caution; it is a development branch.

With this caveat we need to switch to the latest stable version. To do so, type the following command:
```
$ cd ~/.vim/pack/plugins/start/vim-go
$ git checkout $(git tag -l --sort version:refname | grep -v rc | tail -1)
```  
At the moment of writing this article the latest stable version of the `vim-go` was `v1.26`.

Install the required Go utilities (this command runs Vim, starts `:GoInstallBinaries` command and quits Vim):
```
$ vim -c ":GoInstallBinaries" -c ":q"
```

Configure the help system:
```
$ vim -c ":helptags ALL" -c ":q"
```

Turn on the syntax highlighting and file type detection. Open the ~/.vimrc file:
```
$ vim ~/.vimrc
```

Add this lines:
```
syntax on
filetype plugin indent on
```

Now after opening *.go file you should see the syntax highlighting in action.

To trigger the auto completion menu by typing `Ctrl+X`+`Ctrl+O` in `INSERT` mode.

To navigate to code definition move the cursor to an interested identifier and execute `:GoDef`.

## Resources
[Repository](https://github.com/fatih/vim-go/blob/master/README.md) of the `vim-go` plugin.