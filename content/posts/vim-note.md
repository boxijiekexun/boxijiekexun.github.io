---
title: "Vim 使用笔记：从基础命令到我的 .vimrc 配置"
date: 2026-10-13
draft: false
description: "整理 Vim 的常用命令、编辑技巧和个人 Vim 配置。"
tags:
  - Vim
  - Linux
  - WSL
categories:
  - 技术笔记
showTableOfContents: true

---

> Vim 的门槛不在于快捷键太多，而在于它把“编辑”拆成了**操作、对象和范围**。熟悉这套思路以后，许多看似复杂的命令其实都能自然组合出来。

最早接触 Vim 时，我只会 `i`、`:wq` 和 `:q!`，稍复杂一点的修改就想切回 VS Code。后来在 Linux、WSL 和 C/C++ 项目里用得越来越多，才逐渐意识到：Vim 真正好用的地方，不是手速，而是可以非常准确地描述“我要修改哪一段文字”。

这篇笔记分成两部分：前半部分整理 Vim 原生命令，后半部分记录我实际使用的 `.vimrc` 配置。**除特别标注外，文中的命令都是 Vim 自带的；`Space` 快捷键和 `gcc` 分别来自个人配置与插件。**

## 1. 先弄清楚 Vim 的几种模式

| 模式 | 如何进入 | 主要用途 |
| --- | --- | --- |
| Normal（普通模式） | 启动 Vim，或按 `Esc` | 移动、删除、复制、执行命令 |
| Insert（插入模式） | `i`、`a`、`o` 等 | 像普通编辑器一样输入文字 |
| Visual（可视模式） | `v`、`V`、`Ctrl+v` | 选择字符、整行或矩形块 |
| Command-line（命令行模式） | 普通模式输入 `:`、`/`、`?` | 保存退出、搜索、替换等 |
| Select（选择模式） | 某些配置或 `Ctrl+g` 从 Visual 切换 | 更接近传统编辑器的选中即替换行为 |

常见的插入方式：

| 命令 | 作用 |
| --- | --- |
| `i` / `a` | 在光标前 / 后插入 |
| `I` / `A` | 在本行首个非空白字符前 / 行尾插入 |
| `o` / `O` | 在下方 / 上方新建一行并插入 |
| `Esc` | 返回普通模式 |

**建议养成习惯：输入文字时进入 Insert，输入完回到 Normal。** 不必一直停留在插入模式里，用方向键完成全部编辑。

## 2. 光标移动：先学会准确定位

| 命令 | 含义 |
| --- | --- |
| `h` / `j` / `k` / `l` | 左 / 下 / 上 / 右 |
| `w` / `b` | 移到下一个 / 前一个单词的开头 |
| `e` | 移到当前或下一个单词的末尾 |
| `W` / `B` | 按空白分隔的 WORD 移动，适合路径等内容 |
| `0` / `^` / `$` | 行首第一列 / 首个非空白字符 / 行尾 |
| `gg` / `G` | 文件第一行 / 最后一行 |
| `42G` / `:42` | 跳到第 42 行 |
| `Ctrl+d` / `Ctrl+u` | 向下 / 向上滚动半屏 |
| `%` | 在匹配的括号之间跳转（普通模式） |
| `f,` / `t,` | 在当前行找到下一个逗号 / 移到逗号前 |
| `;` / `,` | 重复上一次 `f`、`t` 搜索 / 反方向重复 |

数字可以和很多移动命令组合：`5j` 向下五行，`3w` 向前三个单词。对于代码编辑，`^`、`$`、`%`、`f` 往往比逐字符移动更实用。

注意：**普通模式里的 `%` 是跳转匹配括号；命令行里的 `:%` 则表示“整个文件的行范围”**。同一个符号在不同上下文中有不同含义。

## 3. Vim 的核心：操作符 + 移动范围

可以把大部分编辑动作理解成一个简单公式：

```
[次数] 操作符 + 移动范围 / 文本对象
```

例如，`d` 表示删除，`y` 表示复制（yank），`c` 表示修改（change）。把它们与 `w`、`$`、`iw` 等范围组合，便得到一系列命令。

| 命令 | 作用 |
| --- | --- |
| `x` / `X` | 删除光标处 / 光标前一个字符 |
| `dd` / `3dd` | 删除当前行 / 从当前行开始的三行 |
| `dw` / `d$` | 删除到下一个单词开头 / 删除到行尾 |
| `diw` | 删除光标所在单词，不包含周围空白 |
| `ciw` | 删除当前单词并进入插入模式 |
| `cc` / `C` | 修改当前整行 / 从光标修改到行尾 |
| `yy` / `3yy` | 复制当前行 / 连续三行 |
| `yw` / `y$` | 复制到下一个单词开头 / 复制到行尾 |
| `p` / `P` | 在光标后 / 前粘贴；整行内容则粘到下 / 上一行 |
| `u` / `Ctrl+r` | 撤销 / 重做 |
| `.` | 重复最近一次可重复的修改 |

### 3.1 文本对象：`i` 和 `a`

在操作符后，`i` 常表示 *inside*，`a` 常表示 *around*：

```cpp
process(user.name, get_value());
```

当光标在 `get_value()` 的括号内部时：

- `di(`：删除括号内的内容，保留括号。
- `ci(`：清空括号内的内容，并进入 Insert 模式。
- `da(`：连同这一对括号一起删除。
- `ci"`：修改当前双引号包围的字符串内容。
- `dap`：删除当前段落及其相邻分隔空行（按 Vim 的段落定义）。

这是 Vim 很值得学习的一部分：**不需要先框选，只需描述目标对象。**

### 3.2 批量操作与整个文件

| 目标 | 命令 |
| --- | --- |
| 复制整个文件到 Vim 寄存器 | `ggVGy` 或 `:%y` |
| 删除整个文件的内容 | `ggVGd` 或 `:%d` |
| 全选整个文件（Visual 模式） | `ggVG` |
| 删除但不覆盖之前复制的内容 | `"_dd`（当前行），`:%d _`（整个文件） |
| 撤销误删 | `u` |

这里 `ggVG` 的步骤是：文件开头 → 按行选中 → 扩展到文件末尾。`:%d` 只会修改当前缓冲区；**只有执行 `:w` 后，磁盘上的文件内容才会被覆盖**。

Vim 的“删除”通常还会把内容保存到寄存器，所以 `dd` 然后 `p`，就可以完成整行的剪切和粘贴。如果不想影响之前复制的内容，可以使用黑洞寄存器 `"_`。

## 4. Visual：多行选择与块操作

| 命令 | 用途 |
| --- | --- |
| `v` | 按字符选择 |
| `V` | 按整行选择 |
| `Ctrl+v` | 按矩形区域选择（Visual Block） |
| `V` → `4j` → `d` | 删除连续五行 |
| `V` → 选择多行 → `>` / `<` | 增加 / 减少缩进 |
| `V` → 选择多行 → `=` | 根据缩进规则重新缩进 |

块选择模式很适合给对齐的代码添加前缀。例如连续三行 C++：

```cpp
int a = 1;
int b = 2;
int c = 3;
```

把光标放在第一行行首，按 `Ctrl+v`、`2j`、`I`，输入 `// `，最后按 `Esc`，即可对选中行插入相同前缀。反过来，选中每行前两个 `/`，按 `x` 就能批量取消注释。要注意：**块编辑按列操作，缩进不一致时不一定合适**。

## 5. 注释与取消注释：交给 vim-commentary

Vim 原生没有对所有语言通用的注释切换快捷键。C++、Go 使用 `//`，Python、Shell 使用 `#`，因此我更倾向于安装 [vim-commentary](https://github.com/tpope/vim-commentary)，让它利用文件类型及 `commentstring` 选择注释形式。

在 WSL 中安装：

```bash
mkdir -p ~/.vim/pack/vendor/start
git clone https://github.com/tpope/vim-commentary.git \
  ~/.vim/pack/vendor/start/vim-commentary
```

重新打开 Vim 后：

| 命令 | 作用 |
| --- | --- |
| `gcc` | 切换当前行注释 |
| `3gcc` | 切换当前行起连续三行的注释 |
| `V` 选中若干行，再按 `gc` | 批量注释 / 取消注释 |
| `ggVGgc` | 注释或取消注释整个文件 |
| `gcap` | 切换当前段落注释 |

例如，光标停在 `int a = 1;` 上，按一次 `gcc` 添加 `//`，再按一次便取消。**这比为每种语言记不同的替换表达式更方便。**

检查插件是否生效：

```vim
:verbose nmap gcc
:set filetype?
:set commentstring?
```

第一条查看按键映射及其来源；第二条确认文件类型，例如 `cpp`；第三条确认注释模板。`gcc` 是插件提供的，不是 Vim 默认快捷键；**我的原始 `.vimrc` 并没有配置这一映射**。

## 6. 搜索、替换与跳转

普通模式中：

| 命令 | 用途 |
| --- | --- |
| `/keyword` | 向下搜索 |
| `?keyword` | 向上搜索 |
| `n` / `N` | 沿当前搜索方向 / 反方向查找下一个匹配 |
| `*` / `#` | 向前 / 向后搜索光标所在单词 |
| `:nohlsearch` | 暂时关闭搜索结果高亮 |
| `:%s/old/new/g` | 在整个文件替换所有匹配 |
| `:%s/old/new/gc` | 全文件替换，但逐个确认 |
| `:10,20s/old/new/g` | 只替换第 10～20 行 |

这里 `s` 是 substitute，`g` 表示每一行替换所有匹配，`c` 表示替换前确认；`%` 则表示文件所有行。

我的 `.vimrc` 开启了 `incsearch`、`hlsearch`、`ignorecase`、`smartcase`：搜索过程会即时显示匹配，默认忽略大小写，但输入大写字母时区分大小写。另外，我把普通模式的 `Esc` 映射为清除搜索高亮，所以不必每次手输 `:nohlsearch`。

## 7. 文件、Buffer 与分屏

编辑多个文件时，要区分三个概念：**Buffer 是已加载的文件内容，Window 是显示 Buffer 的视图，Tab 是窗口布局的集合**。并不是打开一个文件就一定需要新建一个 Tab。

| 命令 | 用途 |
| --- | --- |
| `:e file.cpp` | 打开文件到当前窗口 |
| `:w` / `:wq` | 保存 / 保存并退出 |
| `:q` / `:q!` | 退出 / 放弃未保存修改并退出 |
| `:wa` / `:qa` | 保存所有已修改 Buffer / 退出全部窗口 |
| `:ls` | 列出 Buffer |
| `:bn` / `:bp` | 下一个 / 上一个 Buffer |
| `:b 3` | 切换到编号为 3 的 Buffer |
| `:sp file.cpp` | 水平分屏 |
| `:vsp file.cpp` | 垂直分屏 |
| `Ctrl+w` 再按 `h/j/k/l` | 在相邻分屏之间切换 |
| `Ctrl+w` 再按 `q` | 关闭当前分屏 |
| `:tabnew` / `gt` / `gT` | 新建 Tab / 下一个 Tab / 上一个 Tab |
| `:Ex` | 打开 Vim 自带的 netrw 文件浏览器 |

在我的配置里，`splitright` 和 `splitbelow` 会让新垂直分屏出现在右侧、新水平分屏出现在下侧；`hidden` 则允许切换 Buffer 时暂时保留未保存的修改，但**不等于自动保存到磁盘**。

## 8. 我的 `.vimrc`：为什么这样配置

我的主要环境是 **Windows + WSL Ubuntu + 终端 Vim**，偶尔阅读 C/C++ 源码，因此配置重点不在花哨界面，而是行号、缩进、搜索、系统剪贴板和代码跳转。下面是当前配置的关键部分，不是让所有人原样复制的万能模板。

### 8.1 显示与编辑体验

```vim
set nocompatible
syntax enable
filetype plugin indent on
set number
set relativenumber
set laststatus=2
set showcmd
set ruler
set wildmenu
set encoding=utf-8
set fileencoding=utf-8
set mouse=a
set nowrap
set scrolloff=5
set sidescrolloff=5
```

`number` 配合 `relativenumber`，使当前行显示绝对行号，其他行显示相对距离。这样需要往下移动七行时，看到行号直接输入 `7j` 即可。`scrolloff=5` 让光标靠近屏幕上下边缘时预留几行；`nowrap` 只是不在屏幕上折行，**不会改变文件中的真实换行**。

### 8.2 缩进与 Tab

```vim
set autoindent
set smartindent
set copyindent
set preserveindent
set tabstop=4
set shiftwidth=4
set softtabstop=4
set expandtab
```

这组设置让编辑器默认采用四列宽的缩进，并尽可能继承已有缩进：

- `tabstop=4`：一个实际 Tab 字符显示为四列宽。
- `shiftwidth=4`：`>>`、`<<`、自动缩进等通常按四列调整。
- `softtabstop=4`：在插入模式按 Tab / Backspace 时，以四列为编辑单位。
- `expandtab`：输入 Tab 键时通常插入空格，而非真正的 Tab 字符。
- `copyindent`、`preserveindent`：尽量沿用、保留已有缩进形式；它们不等同于“强制全部改为空格”。

**这套缩进不是所有语言的统一标准。** 例如 Go 项目通常交给 `gofmt` 统一格式，Makefile 的命令缩进行又可能需要真正的 Tab。遇到特殊文件应由文件类型配置或格式化工具覆盖全局设置。

### 8.3 搜索与按键映射

```vim
set incsearch
set hlsearch
set ignorecase
set smartcase
nnoremap <Esc> :nohlsearch<CR><Esc>

let mapleader = " "
nnoremap <leader>w :w<CR>
nnoremap <leader>q :q<CR>
nnoremap <leader>x :wq<CR>
nnoremap Q <Nop>
```

`<leader>` 被设成空格，因此：

| 自定义快捷键 | 功能 |
| --- | --- |
| `Space w` | 保存 |
| `Space q` | 退出（有未保存修改时会提示） |
| `Space x` | 保存并退出 |
| `Esc`（Normal） | 清除搜索高亮 |
| `Q`（Normal） | 禁止进入 Ex 模式 |

映射写在 `.vimrc` 中，**不是 Vim 原生默认行为**。排查快捷键时，可以使用 `:verbose nmap <Space>w` 或 `:verbose map` 检查实际加载的映射与来源。

### 8.4 WSL 与 Windows 剪贴板

这是我较常使用、也容易产生误解的一部分。普通 Vim 的 `yy` 复制到寄存器，不保证 Windows 的 `Ctrl+V` 就能直接粘贴；具体取决于 Vim 编译特性和系统剪贴板环境。

我的配置先检测 `has('clipboard')`，如果支持便设置：

```vim
if has('clipboard')
    set clipboard=unnamedplus
endif
```

同时在检测到 WSL 环境后，额外提供了这些快捷键：

| 操作 | 当前 `.vimrc` 的映射 |
| --- | --- |
| 将当前整行复制到 Windows 剪贴板 | Normal：`Space yy` |
| 将选中内容复制到 Windows 剪贴板 | Visual：`Space y` |
| 从 Windows 剪贴板读取后粘贴 | Normal：`Space p` |

其实现思路是调用 Windows 的 `clip.exe` 写入剪贴板，使用 `powershell.exe` 的 `Get-Clipboard -Raw` 读取剪贴板，并清除 Windows 换行中可能包含的 `\r`。这套快捷键**依赖 WSL 可以启动相应 Windows 程序**，与普通 Vim 的 `p` 不能完全等同。

一个实用场景是：`ggVG` 全选文件 → `Space y` 复制到 Windows 剪贴板 → 浏览器里 `Ctrl+V`。这是个人映射，不是 Vim 自带的命令。

### 8.5 自动括号补全：方便，但要知道副作用

我使用了一组简单的 Insert 映射：

```vim
inoremap ( ()<Left>
inoremap [ []<Left>
inoremap { {}<Left>
inoremap " ""<Left>
inoremap ' ''<Left>
```

输入 `(` 时自动插入配对括号，并把光标移到中间。另外，配置里还用 `inoremap <expr>` 检查右侧是否已有 `)`、`]`、`}`，若有则只移动光标。

这种手写补全虽然轻量，但在粘贴 TOML、Markdown、CSS 或包含大量引号的文本时，**可能与终端粘贴方式发生冲突，造成字符重复或结构损坏**。因此我习惯先备份配置，长段文本优先使用 VS Code 编辑；修改后立刻构建验证。出现 Vim 的 `E325` 交换文件警告时，也会先检查是否仍有其他 Vim 进程，而不是直接删除 `.swp`。

### 8.6 C/C++：为 LSP 预留跳转快捷键

配置中还有一组在 C/C++ 文件启用 LSP 后才设置的快捷键：

```vim
nmap <buffer> gd <plug>(lsp-definition)
nmap <buffer> gr <plug>(lsp-references)
nmap <buffer> K <plug>(lsp-hover)
nmap <buffer> <leader>rn <plug>(lsp-rename)
```

预期分别用于**跳转定义、查找引用、查看悬浮信息、重命名符号**。它们依赖支持这些 `<plug>` 映射的 LSP 插件以及正确启动的语言服务器（例如 clangd），**不是仅有 `.vimrc` 就能生效**。可以用 `:verbose nmap gd` 和 `:LspStatus`（仅部分插件支持）等方式排查，具体以所安装插件的文档为准。

此外，我把 `Ctrl+A` 改成了全选文件并进入 Select 模式：

```vim
nnoremap <C-a> ggVG<C-g>
inoremap <C-a> <Esc>ggVG<C-g>
```

这和 Vim 原生的 `Ctrl+A`（将光标处或附近数字加一）不同；它也不完全等于 `ggVG` 的 Visual 模式。如果要在选择后使用 `y`、`d`、`gc` 等 Vim 操作符，我仍习惯显式输入 `ggVG`。

### 8.7 其他配置：文件浏览、提示音和选择行为

`.vimrc` 里还有一些不太显眼、但会影响日常使用的设置：

```vim
set noerrorbells
set visualbell
set t_vb=
set history=1000
set whichwrap+=<,>,h,l
let g:netrw_liststyle = 3
set keymodel=startsel,stopsel
set selectmode=key
set selection=exclusive
```

- `noerrorbells` 用于禁止错误提示音；`visualbell` 尝试以视觉提示替代声音，实际闪烁效果还与终端有关。
- `history=1000` 增加命令行历史记录数量；在 `:` 命令行中按方向键可回顾先前输入。
- `whichwrap` 允许某些左右移动操作跨越行边界，属于个人习惯设置。
- `netrw_liststyle=3` 让 `:Ex` 的文件浏览器默认采用树形列表。
- `keymodel`、`selectmode`、`selection` 调整选择行为，配合我的 `Ctrl+A` 映射更接近传统编辑器；不过终端是否把 `Shift+方向键` 传递给 Vim，仍取决于终端支持情况。

配置中还检查了 `~/.vim/pack/vendor/start/ctrlp.vim` 目录，意图在插件存在时调整运行时路径；这**不代表 CtrlP 一定已经安装或加载成功**。如果需要模糊搜索文件，应确认插件本身的安装状态与映射。

最后，原文件同时包含 `filetype plugin indent on` 与后面的 `filetype plugin on`。后者功能已被前者覆盖，整理配置时可以考虑删去重复项；不过在确认其他插件依赖之前，不必为了“精简”贸然重构整份配置。

## 9. 遇到问题时如何排查

| 现象 | 首先检查 |
| --- | --- |
| `gcc` 没反应 | `:verbose nmap gcc`，检查 vim-commentary 是否已加载 |
| 注释符号不对 | `:set filetype?`、`:set commentstring?` |
| 快捷键与想象不同 | `:verbose map`，定位映射来源 |
| Windows 无法粘贴 | `:echo has('clipboard')`，再检查 WSL 的 `clip.exe` / PowerShell |
| 配置文件出现 `E325` | 先确认是否还有另一个 Vim 进程在编辑文件 |
| TOML/CSS 粘贴后无法解析 | 查看具体错误行，检查自动补全映射和多余字符 |
| 改了 `.vimrc` 却没有生效 | 重启 Vim，或执行 `:source ~/.vimrc` |

需要在 Vim 内查阅某个命令的完整定义，可以使用内置帮助，例如 `:help text-objects`、`:help registers`、`:help :substitute`。它们比记忆模糊的网上快捷键表更可靠。

## 10. 我的常用命令速查表

最后把最常用的动作放在一起，日常编辑时可以直接查阅：

| 场景 | 命令 |
| --- | --- |
| 保存 / 退出 | `:w` / `:q` / `:wq` / `:q!` |
| 行首 / 行尾 | `^` / `$` |
| 文件开头 / 末尾 | `gg` / `G` |
| 删除 / 复制当前行 | `dd` / `yy` |
| 修改当前单词 | `ciw` |
| 粘贴 | `p` / `P` |
| 撤销 / 重做 | `u` / `Ctrl+r` |
| 全选 / 复制整个文件 | `ggVG` / `ggVGy` |
| 删除整个文件的内容 | `:%d` |
| 搜索 / 下一个结果 | `/keyword` / `n` |
| 全文件替换并逐个确认 | `:%s/old/new/gc` |
| 注释当前行（插件） | `gcc` |
| 注释整个文件（插件） | `ggVGgc` |
| WSL 复制选中内容到 Windows（个人配置） | Visual 下 `Space y` |
| WSL 从 Windows 粘贴（个人配置） | Normal 下 `Space p` |
| 分屏 / 切换分屏 | `:vsp` / `Ctrl+w` 再按 `h/j/k/l` |

## 结语

真正开始习惯 Vim 后，我反而不太追求“记住全部快捷键”。对于重复出现的操作，先思考它对应的是哪种**范围**：一个单词、一对括号、一行、一个段落，还是整个文件；再决定是删除、复制、修改还是注释。

配置也是一样。`.vimrc` 的价值不在于行数多，而在于把高频、稳定的工作流程固化下来。我的配置仍然有可以改进的地方，尤其是手写自动补全和 WSL 剪贴板兼容性；这些细节也会随着实际使用继续调整。

**参考资料**：[Vim 官方文档](https://www.vim.org/docs.php) · [Vim 在线帮助](https://vimhelp.org/) · [vim-commentary](https://github.com/tpope/vim-commentary)

