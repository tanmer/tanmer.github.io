# Vim

## Cheatsheet 

[https://vim.rtorr.com/](https://vim.rtorr.com/)

## 快捷键

### 代码块复制

⇧+v 进入可视行模式

按上下键选择行，然后按`y`, 走到要粘贴的行，按`p`

### 多行注释

在需要注释的第一行，按`Ctrl` +`V，`然后按向下方向键，走到想要注释的最后一行，按大写`i`进入插入模式，输入`#` ，按`Esc`退出插入模式，再按一次`Esc`即可实现多行注释。

## 配置详解

{% code-tabs %}
{% code-tabs-item title="~/.vimrc" %}
```text
" 不要使用vi的键盘模式，而是vim自己的  
set nocompatible  
  
" 语法高亮  
set syntax=on  
  
" 去掉输入错误的提示声音  
set noeb  
  
" 在处理未保存或只读文件的时候，弹出确认  
set confirm  
  
" 自动缩进  
set autoindent  
set cindent  
  
" Tab键的宽度  
set tabstop=4  
  
" 统一缩进为4  
set softtabstop=4  
set shiftwidth=4  
  
" 不要用空格代替制表符  
set noexpandtab  
  
" 在行和段开始处使用制表符  
set smarttab  
  
" 显示行号  
set number  
  
" 历史记录数  
set history=1000  
  
"禁止生成临时文件  
set nobackup  
set noswapfile  
  
"搜索忽略大小写  
set ignorecase  
  
"搜索逐字符高亮  
set hlsearch  
set incsearch  
  
"行内替换  
set gdefault  
  
"编码设置  
set enc=utf-8  
set fencs=utf-8,ucs-bom,shift-jis,gb18030,gbk,gb2312,cp936  
  
"语言设置  
set langmenu=zh_CN.UTF-8  
set helplang=cn  
  
" 我的状态行显示的内容（包括文件类型和解码）  
set statusline=%F%m%r%h%w\ [FORMAT=%{&ff}]\ [TYPE=%Y]\ [POS=%l,%v][%p%%]\ %{strftime(\"%d/%m/%y\ -\ %H:%M\")}  
"set statusline=[%F]%y%r%m%*%=[Line:%l/%L,Column:%c][%p%%]  
  
" 总是显示状态行  
set laststatus=2  
  
" 在编辑过程中，在右下角显示光标位置的状态行  
set ruler             
  
" 命令行（在状态行下）的高度，默认为1，这里是2  
set cmdheight=2  
  
" 侦测文件类型  
filetype on  
  
" 载入文件类型插件  
filetype plugin on  
  
" 为特定文件类型载入相关缩进文件  
filetype indent on  
  
" 保存全局变量  
set viminfo+=!  
  
" 带有如下符号的单词不要被换行分割  
set iskeyword+=_,$,@,%,#,-  
  
" 字符间插入的像素行数目  
set linespace=0  
  
" 增强模式中的命令行自动完成操作  
set wildmenu  
  
" 使回格键（backspace）正常处理indent, eol, start等  
set backspace=2  
  
" 允许backspace和光标键跨越行边界  
set whichwrap+=<,>,h,l  
  
" 可以在buffer的任何地方使用鼠标（类似office中在工作区双击鼠标定位）  
set mouse=a  
set selection=exclusive  
set selectmode=mouse,key  
  
" 通过使用: commands命令，告诉我们文件的哪一行被改变过  
set report=0  
  
" 启动的时候不显示那个援助索马里儿童的提示  
set shortmess=atI  
  
" 在被分割的窗口间显示空白，便于阅读  
set fillchars=vert:\ ,stl:\ ,stlnc:\  
  
" 高亮显示匹配的括号  
set showmatch  
  
" 匹配括号高亮的时间（单位是十分之一秒）  
set matchtime=5  
  
" 光标移动到buffer的顶部和底部时保持3行距离  
set scrolloff=3  
  
" 为C程序提供自动缩进  
set smartindent  
  
" 只在下列文件类型被侦测到的时候显示行号，普通文本文件不显示  
if has("autocmd")  
   autocmd FileType xml,html,c,cs,java,perl,shell,bash,cpp,python,vim,php,ruby set number  
   autocmd FileType xml,html vmap <C-o> <ESC>'<i<!--<ESC>o<ESC>'>o-->  
   autocmd FileType java,c,cpp,cs vmap <C-o> <ESC>'<o/*<ESC>'>o*/  
   autocmd FileType html,text,php,vim,c,java,xml,bash,shell,perl,python setlocal textwidth=100  
   autocmd Filetype html,xml,xsl source $VIMRUNTIME/plugin/closetag.vim  
   autocmd BufReadPost *  
      \ if line("'\"") > 0 && line("'\"") <= line("$") |  
      \   exe "normal g`\"" |  
      \ endif  
endif " has("autocmd")  
  
" F5编译和运行C程序，F6编译和运行C++程序  
" 请注意，下述代码在windows下使用会报错  
" 需要去掉./这两个字符  
  
" C的编译和运行  
map <F5> :call CompileRunGcc()<CR>  
func! CompileRunGcc()  
exec "w"  
exec "!gcc % -o %<"  
exec "! ./%<"  
endfunc  
  
" C++的编译和运行  
map <F6> :call CompileRunGpp()<CR>  
func! CompileRunGpp()  
exec "w"  
exec "!g++ % -o %<"  
exec "! ./%<"  
endfunc  
  
" 能够漂亮地显示.NFO文件  
set encoding=utf-8  
function! SetFileEncodings(encodings)  
    let b:myfileencodingsbak=&fileencodings  
    let &fileencodings=a:encodings  
endfunction  
function! RestoreFileEncodings()  
    let &fileencodings=b:myfileencodingsbak  
    unlet b:myfileencodingsbak  
endfunction  
  
au BufReadPre *.nfo call SetFileEncodings('cp437')|set ambiwidth=single  
au BufReadPost *.nfo call RestoreFileEncodings()  
  
" 高亮显示普通txt文件（需要txt.vim脚本）  
au BufRead,BufNewFile *  setfiletype txt  
  
" 用空格键来开关折叠  
set foldenable  
set foldmethod=manual  
nnoremap <space> @=((foldclosed(line('.')) < 0) ? 'zc' : 'zo')<CR>  
  
" minibufexpl插件的一般设置  
let g:miniBufExplMapWindowNavVim = 1  
let g:miniBufExplMapWindowNavArrows = 1  
let g:miniBufExplMapCTabSwitchBufs = 1  
let g:miniBufExplModSelTarget = 1   
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 安装Vundle插件管理工具

用Vim，第一件事就是下载插件管理器 [Vundle](https://github.com/VundleVim/Vundle.vim) \(vim bundler\)，有了他，就能让Vim用上丰富的插件

```bash
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

然后是修改`.vimrc`配置文件文件，使用`Vundle`

```text
set nocompatible              " be iMproved, required
filetype off                  " required
syntax  on

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'


" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line
```

### 通过Vundle安装插件

{% code-tabs %}
{% code-tabs-item title="~/.vimrc" %}
```text
Plugin 'scrooloose/nerdtree'
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="vim 编辑器中执行" %}
```text
:source %
:PlugInstall
```
{% endcode-tabs-item %}
{% endcode-tabs %}



## 常用插件

### NERDTree

> [https://github.com/scrooloose/nerdtree](https://github.com/scrooloose/nerdtree)

NERDTree是Vim的文件系统浏览器，通过这个插件，用户能够可视化地查看目录结构，快速打开，查看和编辑文件。该插件还可以通过特定的API自定义功能映射。

![](../.gitbook/assets/image%20%2848%29.png)

#### 如何安装

编辑`.vimrc`，添加`Plugin 'scrooloose/nerdtree'`

{% code-tabs %}
{% code-tabs-item title="~/.vimrc" %}
```text
" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'

Plugin 'scrooloose/nerdtree'

" All of your Plugins must be added before the following line
```
{% endcode-tabs-item %}
{% endcode-tabs %}

安装插件

```text
:source %
:PluginInstall
```

![](../.gitbook/assets/image%20%2841%29.png)



