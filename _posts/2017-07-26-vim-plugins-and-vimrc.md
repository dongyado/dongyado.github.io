---
layout: post
title: 几款好用的 vim 插件和使用 vundle 安装 vim 插件
date: 2017-07-26
categories:
- tool
tags: [linux, vim]
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  views: '2'
author:
  login: slayer
  email: dongyado@gmail.com
  display_name: slayer
---

vim 作为一个有历史的编辑器，经过了几十年的洗礼，依然是相当一部分程序员最喜欢的编辑器。
这和它的高效便捷是有很大的关系，如何在你熟悉vim之后，你开始觉得 vim 不是很方便高效，
这时你可以使用数量众多优质的插件打造出一个的更高效的编辑器。

vim 的插件丰富多彩，质量也很高，下面推荐个人觉得不错的几款插件。


### nerdtree

查看当前目录下的目录文件树，可以很方便的找到对应的文件并进行切换编辑。

项目地址：
https://github.com/scrooloose/nerdtree

### nerdcommenter

快速注释和删除注释，支持各种注释方法，比如 /* */ 和 //， 支持块级注释，如果觉得 vim 下的多行注释很麻烦，这就是你要
找的注释插件。

项目地址:
https://github.com/scrooloose/nerdtree

### vim-colors-solarized 

vim 的 solarized 主题， 这个就不用多介绍了。

项目地址:
https://github.com/altercation/vim-colors-solarized

### ctrlp

模糊文件搜索，包括搜索最近文件和搜索当前目录下和子目录下的所有文件，使用频率比较高的插件。

项目地址:
https://github.com/kien/ctrlp.vim

### YouCompleteMe

很强大的自动补全引擎，支持的语言包括 C-family, C#, Go, JavaScript, Python, Rust, TypeScript , 当然也可以支持其他语言。
功能强大，比较重量级，安装起来也比较麻烦。

项目和安装参见：

https://github.com/Valloric/YouCompleteMe


### vundle

vim 的插件管理器，使用这个插件可以很方便的安装和卸载插件。在你决定使用 vim 的插件的时候，先手动安装 vundle, 再使用
vundle 安装其他插件，会帮你节省不少时间。

项目和安装方法参见：
https://github.com/VundleVim/Vundle.vim

## 使用 vundle 安装插件

使用下面的命令安装 vundle:

git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim

然后编辑 .vimrc, 加入 vundle 的配置信息，并加入要安装的插件，再次启动 vim, 
执行 :PluginInstall 即可完成安装。

参见文档： 
https://github.com/VundleVim/Vundle.vim/blob/master/README_ZH_CN.md

个人使用的 .vimrc 文件如下:
~~~
" 
" vim settings 
"
" ----------------------------------------------------
" vim plugin manager 
" 
" use command bellow install vundle for vim:
"   git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
"
set nocompatible
filetype off
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

" add your plugin here

Plugin 'VundleVim/Vundle.vim'               " required
Plugin 'scrooloose/nerdtree'                " file/directory treee
Plugin 'scrooloose/nerdcommenter'           " code commenter
Plugin 'kien/ctrlp.vim'                     " Fuzzy file, buffer, mru, tag, etc finder
Plugin 'altercation/vim-colors-solarized'   " solarized theme

call vundle#end()            " required
filetype plugin indent on    " required 


" ----------------------------------------------------
"  basic settings
"
set nu
syntax enable
syntax on
" let mapleader=','
" let g:mapleader=','

" theme 
set background=dark             " dark | light
colorscheme solarized           " use solarized theme
let g:solarized_termcolors=256  " if you are use terminal

" encoding, autoindent
set autoindent
set encoding=utf-8
set fileencodings=utf-8
set fileencoding=utf-8
set fileformats=unix,dos
set ts=4

" replace tab with 4 whitespace
set shiftwidth=4
set expandtab
set pastetoggle=<F3>

" nerd tree toggle
map <silent>,, :NERDTreeToggle<CR>
" map <silent><C-c> :close<CR>

" nerd commenter
let g:NERDSpaceDelims=1
map cc          <plug>NERDCommenterComment<CR>
map cu          <plug>NERDCommenterUncomment<CR>
map cs          <plug>NERDCommenterSexy<CR>
map ci          <plug>NERDCommenterInvert<CR>
map cy          <plug>NERDCommenterYank<CR>
map ce          <plug>NERDCommenterToEOL<CR>
map c<space>    <plug>NERDCommenterToggle<CR>

~~~


暂时就更新这几款插件，后面发现好用的插件会持续更新。


