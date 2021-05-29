[TOC]

# 预准备

- 系统：Linux(本文使用Arch Linux)
- 环境：VIM8，GCC，GDB(Arch Linux直接安装base-devel包)
- 时常：30分钟+



# 将获得

淘汰 [YCM](https://github.com/ycm-core/YouCompleteMe)，[ctags](https://github.com/universal-ctags/ctags)，[nerdtree](https://github.com/preservim/nerdtree)等经典插件（笨重、性能问题、易出错）。  

通过最新的LSP(Language Server Protocal)技术，和一系列异步辅助插件，为C++开发环境配置高性能VIM。  



# 必备的功能特性

以下是高效IDE所需要具备的功能特性。    

## LSP服务端

选择[clangd](https://clangd.llvm.org/)作为语言服务（背景雄厚ccls没法比）。Arch Linux 直接安装`clang`包即可：  

```
yay clang
```

## VIM插件管理

安装 [vim-plug](https://github.com/junegunn/vim-plug)。在`.vimrc`文件中加入：  

```
" vim-plug auto installation
if empty(glob('~/.vim/autoload/plug.vim'))
  silent !proxychains curl -fLo ~/.vim/autoload/plug.vim --create-dirs
    \ https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
  autocmd VimEnter * PlugInstall --sync | source $MYVIMRC
endif

" plug list
call plug#begin('~/.vim/plugged')
" what u want to install
call plug#end()
```

再次启动vim后，将自动安装`vim-plug`。  

## VIM LSP客户端

选择[coc.nvim](https://github.com/neoclide/coc.nvim)作为LSP的客户端（国人开发，扩展性极强）。  

LSP客户端将提供以下功能：  

- 代码补全
- 静态检测
- 函数跳转

**依赖**  

coc依赖与`nodejs`，键入以下命令安装nodejs和npm：  

```
yay -S nodejs npm
```

**安装**  

安装coc，首先需在`.vimrc`文件中的plug list配置中添加：  

```
Plug 'neoclide/coc.nvim', {'for':['c', 'cpp'], 'branch': 'release'}
```

然后在vim中输入`:PlugInstall`命令，完成安装。  

**配置**  

进入vim，输入`:CocConfig`，编辑json格式的配置文件，将我们的LSP服务端配置进去：  

```json
"languageserver": {
  "clangd": {
    "command": "clangd",
    "rootPatterns": ["compile_flags.txt", "compile_commands.json"],
    "filetypes": ["c", "cc", "cpp", "c++", "objc", "objcpp"]
  }
}
```

保存后，会在`~/.vim/`文件夹下生成`coc-settings.json`配置文件，这时，编辑文件时就会发现代码提示生效了。  

**常用快捷键** 

- 查看中文说明文档：`：h coc-nvim.txt@cn`
- 采用第一行代码提示：TAB键
- 强制代码提示：ctrl + 空格
- 代码跳转-定义位置： gd
- 代码跳转-类型定义位置：gy
- 代码跳转-跳转到实现位置：gi
- 代码跳转-跳转到引用位置：gr
- 代码跳转-跳转到上一错误处：[g
- 代码跳转-跳转到下一错误处：]g
- 重命名当前位置的符号：rn
- 



# 建议安装的插件

## 命令和绑定键提示

 [liuchengxu](https://github.com/liuchengxu)/[vim-which-key](https://github.com/liuchengxu/vim-which-key)

## 代码模板

[SirVer](https://github.com/SirVer)/[ultisnips](https://github.com/SirVer/ultisnips)

## GIT集成

 [tpope](https://github.com/tpope)/[vim-fugitive](https://github.com/tpope/vim-fugitive)

## GIT gutter

[airblade](https://github.com/airblade)/**[vim-gitgutter](https://github.com/airblade/vim-gitgutter)

## 可视化缩进

[Yggdroot](https://github.com/Yggdroot)/[indentLine](https://github.com/Yggdroot/indentLine)

## 更美观的状态栏

[vim-airline/vim-airline](https://github.com/vim-airline/vim-airline)
[vim-airline/vim-airline-themes](https://github.com/vim-airline/vim-airline-themes)

[主题一览]()  

切换主题：`:AirlineTheme <theme>`

安装之后在`.vimrc`中加入：  

```
" airline-coc config
set noshowmode
set noruler
set laststatus=0
set noshowcmd
set cmdheight=1
let g:airline#extensions#coc#enabled = 1
```

