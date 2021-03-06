# Emacs builtin modes 功能介绍

`Emacs`自带的`mode`功能也比较强大，而一般初学者（比如我）使用`Emacs`时间较短，对
它自身强大的`mode`不了解而错失一些可以提高生产力的工具。

## winner-mode

`winner-mode`是一个全局的`minor mode`，它的主要功能是记录窗体的变动。例如当前有
2 个窗口，然后你关了一个，这时可以通过`winner-undo`来恢复。还可以再`winner-redo`
来撤销刚才的`undo`.

它默认按键绑定为:

<kbd>C-c <Left></kbd> `winner-undo`

<kbd>C-c <Right></kbd> `winner-redo`

如果不想它绑定在<kbd>C-c</kbd>前缀按键上，可以通过

``` elisp
(setq winner-dont-bind-my-keys nil)
```

来禁止。

建议配置:

```elisp
(use-package winner-mode
  :ensure nil
  :hook (after-init . winner-mode))
```

同时，它也可以应用在`ediff`上，恢复由`ediff`导致的窗体变动。

```elisp
(use-package ediff
  :ensure nil
  :hook (ediff-quit . winner-undo)
```

## saveplace

`saveplace`记录了上次打开文件时光标停留在第几行、第几列。如果不想每次打开文件都
要再次跳转到上次编辑的位置，这个`mode`可以轻松地应对这种情况。

建议配置:

```elisp
(use-package saveplace
  :ensure nil
  :hook (after-init . save-place-mode))
```

## hl-line

高亮当前行。

```elisp
(use-package hl-line
  :ensure nil
  :hook (after-init . global-hl-line-mode))
```

## hideshow

隐藏、显示结构化数据，如`{ }`里的内容。对于单函数较长的情况比较有用。

建议配置：

```elisp
(use-package hideshow
  :ensure nil
  :diminish hs-minor-mode
  :hook (prog-mode . hs-minor-mode))
```

`hideshow`的默认按键前缀为<kbd>C-c @</kbd>，这里放一个默认的按键与经过
`evil-mode`的版本的对比表格:

| 功能               | 原生                   | `evil-mode`   |
|--------------------|------------------------|---------------|
| `hs-hide-block`    | <kbd>C-c @ C-h</kbd>   | <kbd>zc</kbd> |
| `hs-show-block`    | <kbd>C-c @ C-s</kbd>   | <kbd>zo</kbd> |
| `hs-hide-all`      | <kbd>C-c @ C-M-h</kbd> | <kbd>zm</kbd> |
| `hs-show-all`      | <kbd>C-c @ C-M-s</kbd> | <kbd>zr</kbd> |
| `hs-hide-level`    | <kbd>C-c @ C-l</kbd>   | 无            |
| `hs-toggle-hiding` | <kbd>C-c @ C-c</kbd>   | <kbd>za</kbd> |

一些类似`hideshow`的插件

- [origami](https://github.com/gregsexton/origami.el)
- [folding](https://www.emacswiki.org/emacs/folding.el)
- [yafolding.el](https://github.com/zenozeng/yafolding.el)

其中`origami`有`lsp`支持[lsp-origami](https://github.com/emacs-lsp/lsp-origami)

### hideshow 扩展: 显示被折叠的代码行数

默认情况下`hideshow`对于显示的代码是以`...` `overlay`的形式显示的，而且
`hideshow`给予了自定义的能力，通过设置`hs-set-up-overlay`变量即可。

``` elisp
;; 这里额外启用了 :box t 属性使得提示更加明显
(defconst hideshow-folded-face '((t (:inherit 'font-lock-comment-face :box t))))

(defun hideshow-folded-overlay-fn (ov)
    (when (eq 'code (overlay-get ov 'hs))
      (let* ((nlines (count-lines (overlay-start ov) (overlay-end ov)))
             (info (format " ... #%d " nlines)))
        (overlay-put ov 'display (propertize info 'face hideshow-folded-face)))))

(setq hs-set-up-overlay 'hideshow-folded-overlay-fn)
```

附效果图:

![before-fold](https://emacs-china.org/uploads/default/original/2X/c/c204b95093febf0f2455b17e1c98c3c5d7858a13.png)
![after-fold](https://emacs-china.org/uploads/default/original/2X/a/a88653d0d1f48d0e23d1814e46ba998390fc61da.png)

## whitespace

显示空白字符，如`\t` `\f` `\v` 空格等等。

可以配置在`prog-mode`，`markdown-mode`和`conf-mode`下，显示行尾的空白字符。

```elisp
(use-package whitespace
  :ensure nil
  :hook ((prog-mode markdown-mode conf-mode) . whitespace-mode)
  :config
  (setq whitespace-style '(face trailing)))
```

当然，仅显示行尾空白字符也可以简单地设置`show-trailing-whitespace`为`t`来开启。

[kinono](https://emacs-china.org/u/kinono) 分享的配置:

```elisp
(use-package whitespace
  :ensure nil
  :hook (after-init . global-whitespace-mode) ;; 注意，这里是全局打开
  :config
  ;; Don't use different background for tabs.
  (face-spec-set 'whitespace-tab
                 '((t :background unspecified)))
  ;; Only use background and underline for long lines, so we can still have
  ;; syntax highlight.

  ;; For some reason use face-defface-spec as spec-type doesn't work.  My guess
  ;; is it's due to the variables with the same name as the faces in
  ;; whitespace.el.  Anyway, we have to manually set some attribute to
  ;; unspecified here.
  (face-spec-set 'whitespace-line
                 '((((background light))
                    :background "#d8d8d8" :foreground unspecified
                    :underline t :weight unspecified)
                   (t
                    :background "#404040" :foreground unspecified
                    :underline t :weight unspecified)))

  ;; Use softer visual cue for space before tabs.
  (face-spec-set 'whitespace-space-before-tab
                 '((((background light))
                    :background "#d8d8d8" :foreground "#de4da1")
                   (t
                    :inherit warning
                    :background "#404040" :foreground "#ee6aa7")))

  (setq
   whitespace-line-column nil
   whitespace-style
   '(face             ; visualize things below:
     empty            ; empty lines at beginning/end of buffer
     lines-tail       ; lines go beyond `fill-column'
     space-before-tab ; spaces before tab
     trailing         ; trailing blanks
     tabs             ; tabs (show by face)
     tab-mark         ; tabs (show by symbol)
     )))
```

比较好的是能指示过长的行，这样都不需要装那种显示一条竖线的插件了。

![效果图](https://emacs-china.org/uploads/default/optimized/2X/f/f3820ca342843118043eb220ce9cf77d00805f7b_2_690x297.png)

## so-long

有时候会打开一些文件，这些文件里的某一行特别长，而`Emacs`没有针对这种情况做特殊
处理，会导致整个界面卡死。现在它来了！

直接全局启用:

```elisp
(use-package so-long
  :ensure nil
  :config (global-so-long-mode 1))
```

当打开一个具有长行的文件时，它会自动检测并将一些可能导致严重性能的`mode`关闭，
如`font-lock` (`syntax highlight`)。

注：`Emacs` 27+ 自带

## glasses

当遇到驼峰式的变量时，如`CamelCasesName`，但是你比较喜欢`GNU`式的命名方式（使用
下划线），那么你可以开启`glasses-mode`。它只会让`CamelCasesName`**显示**成
`Camel_Cases_Name`而不会对原文件做出修改。

不过，大写字母加下划线的组合有点奇怪。

## subword

由[kinono](https://emacs-china.org/u/kinono)分享。

`subword`可以处理`CamelCasesName`这种驼峰式的单词，<kbd>M-f</kbd>
(`forward-word`) 后，光标会依次停在大写的词上。

```elisp
(use-package subword
  :ensure nil
  :hook (after-init . global-subword-mode))
```

如果不想全局打开，也可以只利用`subword-forward`等移动命令。

此外，`subword`包还提供了一个模式叫做`superword-mode`。在这个模式下，
`this_is_a_symbol`被认为是一个单词。 <kbd>M-f</kbd> (`forward-word`) 可以直接跳
过。

## follow-mode

如果你的屏幕很宽，但是实际显示的条目的宽度无法利用这宽屏幕，那么`follow-mode`可
以帮助你。一个典型的使用案例是，再打开一个窗口，然后对当前`buffer`开启
`follow-mode`，这样之后另一个窗口显示的内容会是当前窗口的后续。例如，一个文件有
100行，当前`buffer`只能显示10行，那么另一个窗口将会显示下面10行。如果嫌窗口数还
是太少，可以继续增多。

![follow-mode](https://emacs-china.org/uploads/default/original/2X/b/b6f11e53b620049c4534a92bd7f22e8f08a15483.png)

## delsel

由[Kermit95](https://emacs-china.org/u/Kermit95)分享。

选中文本后，直接输入就可以，省去了删除操作。这在其他文本编辑器里都是标配，建议打开。

```elisp
(use-package delsel
  :ensure nil
  :hook (after-init . delete-selection-mode))
```

## parenthesis

高亮显示配对的`( )` `[ ]` `{ }` 括号，比较实用，建议打开。

```elisp
(use-package paren
  :ensure nil
  :hook (after-init . show-paren-mode)
  :config
  (setq show-paren-when-point-inside-paren t
        show-paren-when-point-in-periphery t))
```

## simple

在`modeline`里显示行号、列号以及当前文件的总字符数。

```elisp
(use-package simple
  :ensure nil
  :hook (after-init . (lambda ()
                         (line-number-mode)
                         (column-number-mode)
                         (size-indication-mode))))
```

## autorevert

有时候`Emacs`里打开的文件可能被外部修改，启用`autorevert`的话可以自动更新对应的
`buffer`.

```elisp
(use-package autorevert
  :ensure nil
  :hook (after-init . global-auto-revert-mode))
```

## isearch

本身`Emacs`自带的`isearch`已经足够强大，稍加修改就可以增加实用性。

例如[`anzu`](https://github.com/emacsorphanage/anzu)的显示匹配个数的功能就已经原
生支持了。通过

```elisp
(setq isearch-lazy-count t
      lazy-count-prefix-format "%s/%s ")
```

来显示如 `10/100` 这种状态。

比较恼人的一点是，在搜索中删除字符会回退搜索结果，而不是停在当前位置将最后一个搜
索字符删除。这里可以通过`remap isearch-delete-char`来实现。

此外，还可以将搜索结果保持在高亮状态以方便肉眼识别。这个是通过设置
`lazy-highlight-cleanup`为`nil`实现的。去除高亮状态需要人工`M-x`调用
`lazy-highlight-cleanup`。

```elisp
(use-package isearch
  :ensure nil
  :bind (:map isearch-mode-map
         ([remap isearch-delete-char] . isearch-del-char))
  :custom
  (isearch-lazy-count t)
  (lazy-count-prefix-format "%s/%s ")
  (lazy-highlight-cleanup nil))
```

注：`isearch-lazy-count`和`lazy-count-prefix-format`需要`Emacs` 27+

## tempo

[`tempo`](https://www.emacswiki.org/emacs/TempoMode)可以算是`yasnippet`的祖先，
`skeleton`算是它的爷爷。由于`tempo`里可以使用`elisp`函数，灵活性非常大。

实际上在写代码的时候，想插入一个**LICENSE**头是个比较常用的需求，它也可以通过其
他方式如`auto-insert`在打开文件时就自动插入。在这里，我们使用`tempo`来实现。

目前比较推荐的方式是采用`SPDX`的格式，而不是直接把`license`内容写入代码文件中。
采用`SPDX`格式可以有效的减少文件大小，不会喧宾夺主占用大量代码行数。

一个典型的`license`头是这样:

``` cpp
// Copyright 2017 - 2018 ccls Authors
// SPDX-License-Identifier: Apache-2.0
```

所以我们可以仿照着这个格式来写一个`tempo`的`template`.

``` elisp
;; 完整的列表非常长，可以访问 https://spdx.org/licenses/ 获得
(defconst license-spdx-identifiers
  '(Apache-1.0 Apache-2.0 MIT))

(tempo-define-template "license"
  '(comment-start
    (format "Copyright %s - present %s Authors"
            (format-time-string "%Y")
            (if (featurep 'projectile)
                (progn
                  (require 'projectile)
                  (projectile-project-name))
              "Unknown"))
    comment-end > n>
    comment-start
    "SPDX-License-Identifier: " (completing-read "License: "
                                                 license-spdx-identifiers)
    comment-end > n>)
  'license
  "Insert a SPDX license.")
```

`tempo`内的`>`表示的是缩进，`n`表示的是插入一个换行，其他的部分就是一个普通的
`elisp`函数了。

这样定义了这个`template`之类，会生成一个叫`tempo-template-license`的函数。因此我
们可以直接调用它来插入`license`头部。

此外还可以结合`abbrev-mode`来自动替换，如果想在`elisp-mode`下直接替代，可以通过
`define-abbrev`来实现：

```elisp
(define-abbrev emacs-lisp-mode-abbrev-table ";license" "" 'tempo-template-license)
```

这里只需要在`elisp-mode`下开启`abbrev-mode`，然后输入`;license `就会实现自动替换
（注意，最后要有一个空格）。

## align

听说有些写`java`的朋友特别喜欢将变量的`=`对齐，即原来的代码是这样的:

```java
private int magicNumber = 0xdeadbeef;
private double PI = 3.14159265358939723846264;
```

选中它们，然后调用`align-regexp`，给定`=`作为它的参数，就会将上述代码的`=`部分对
齐了。

```java
private int magicNumber = 0xdeadbeef;
private double PI       = 3.14159265358939723846264;
```

其他`align`相关的函数功能还有待开发。

## make isearch behave more like searching in browser

在浏览器里，我们只需要按<kbd>C-f</kbd>，然后敲入所要搜索的字符串。之后只要按回车
就可以不断地向下搜索。如果我们需要向上搜索，那么需要点击一下向上的箭头。

现在我们在`isearch`里模拟这种情况，还是使用<kbd>C-s</kbd>来调用`isearch`。但是之
后的`repeat`操作是交给了回车。

首先，我们先定义一下变量来保存当前搜索的方向。

```elisp
(defvar my/isearch--direction nil)
```

然后使得`isearch-mode-map`下的<kbd>C-s</kbd>可以告诉我们当前是在向下搜索；同理，
使得`isearch-mode-map`下的<kbd>C-r</kbd>告诉我们是在向上搜索。

```elisp
(define-advice isearch-repeat-forward (:after (_))
  (setq-local my/isearch--direction 'forward))
(define-advice isearch-repeat-backward (:after (_))
  (setq-local my/isearch--direction 'backward))
```

这里偷懒，采用了`advise`的方式。如果不想侵入，可以自己在上层包装一下对应的命令。

然后在`isearch-mode-map`下的回车操作就是根据`my/isearch--direction`来搜索了。就
是如此简单。

```elisp
(defun my/isearch-repeat (&optional arg)
  (interactive "P")
  (isearch-repeat my/isearch--direction arg))
```

当然在按`Esc`键的时候表明搜索已经结束了，此时应该重置当前的方式:

```elisp
(define-advice isearch-exit (:after nil)
  (setq-local my/isearch--direction nil))
```

完整代码见下方:

```elisp
(use-package isearch
  :ensure nil
  :bind (:map isearch-mode-map
         ([return] . my/isearch-repeat)
         ([escape] . isearch-exit))
  :config
  (defvar my/isearch--direction nil)
  (define-advice isearch-exit (:after nil)
    (setq-local my/isearch--direction nil))
  (define-advice isearch-repeat-forward (:after (_))
    (setq-local my/isearch--direction 'forward))
  (define-advice isearch-repeat-backward (:after (_))
    (setq-local my/isearch--direction 'backward))
  )
```

## dired

`dired`是一个用于`directory`浏览的`mode`，功能非常丰富。因此这里介绍的东西肯定不
能完全覆盖，会慢慢完善之。

### 在 dired 中用外部程序打开对应文件

在`dired-mode-map`中，也是可以执行`shell`命令的。与之相关的命令有

- `dired-do-shell-command`, 默认绑定在<kbd>!</kbd>
- `dired-smart-shell-command`，默认绑定在<kbd>M-!</kbd>
- `async-shell-command`，默认绑定在<kbd>M-&</kbd>

其中，通过配置`dired-guess-shell-alist-user`可以令`dired-do-shell-command`有一个
比较好的默认命令。例如，我这是样配置的:

``` elisp
(setq dired-guess-shell-alist-user `((,(rx "."
                                           (or
                                            ;; Videos
                                            "mp4" "avi" "mkv" "flv" "ogv" "mov"
                                            ;; Music
                                            "wav" "mp3" "flac"
                                            ;; Images
                                            "jpg" "jpeg" "png" "gif" "xpm" "svg" "bmp"
                                            ;; Docs
                                            "pdf" "md" "djvu" "ps" "eps")
                                           string-end)
                                      ,(cond ((eq system-type 'gnu/linux) "xdg-open")
                                             ((eq system-type 'darwin) "open")
                                             ((eq system-type 'windows-nt) "start")
                                             (t "")))))
```

这里考虑了多个平台下的差异。如`linux`平台下会使用`xdg-open`来打开对应的这些文件
(通过`mimeinfo`来配置，见`~/.config/mimeapps.list`)。但是它有一个缺点，会阻塞当
前的`Emacs`进程，所以仅适用于临时查看的需求。

`dired-smart-shell-command`与`dired-do-shell-command`类似，也会阻塞当前`Emacs`进
程。

`async-shell-command`则不会阻塞当前`Emacs`，唯一的缺点可能是会多弹出个`buffer`吧。
如果对`async-shell-command`的结果不是很感兴趣，可能通过`shackle`等类似的工具把忽
略对应的`buffer`。

如果使用的是`Emacs` **28**的话，并且已经设置了

``` elisp
(setq browse-url-handlers '(("\\`file:" . browse-url-default-browser)))
```

可以直接在`dired`里按`W` (`browse-url-of-dired-file`), 这会直接用外部程序打开。
当然，它不会阻塞`Emacs`。

### 隐藏、显示 以`.`开头的文件

`dired`显示文件时使用的`ls`命令参数是由`dired-listing-switches`来控制的，它的默
认值是`-al`。如果不想要显示以`.`开头的文件，那么通过<kbd>C-u s</kbd> (`s`为
`dired-sort-toggle-or-edit`)来重新设置`dired-listing-switches`。

如果只是想简单地隐藏当前目录下以`.`开头的文件，那么可以通过将满足`^\\.`正则的行
删除就行（真实文件并没有删除，只是删除它的显示）。注意到`dired-do-print`命令基本
不怎么使用，于是可以利用`advice`来覆盖它，实现我们自己的`dotfiles-toggle`。

``` elisp
;; 修改自 https://www.emacswiki.org/emacs/DiredOmitMode
(define-advice dired-do-print (:override (&optional _))
    "Show/hide dotfiles."
    (interactive)
    (if (or (not (boundp 'dired-dotfiles-show-p)) dired-dotfiles-show-p)
        (progn
          (setq-local dired-dotfiles-show-p nil)
          (dired-mark-files-regexp "^\\.")
          (dired-do-kill-lines))
      (revert-buffer)
      (setq-local dired-dotfiles-show-p t)))
```

这样只要按一下<kbd>P</kbd>就可以达到隐藏、显示的切换了。

如果不想自己写`elisp`，这里也有一个现成的包 https://github.com/mattiasb/dired-hide-dotfiles

## ispell

`ispell`全称是`interactive` `spell`检查器，它支持`ispell`, `aspell`和`hunspell`，
以下以`hunspell`为例。

``` elisp
;; 这里使用的是 en_US 字典，需要使用包管理安装对应的字典，类似的名字可能 hunspell-en_US
(setq ispell-dictionary "en_US"
      ispell-program-name "hunspell"
      ispell-personal-dictionary (expand-file-name "hunspell_dict.txt" user-emacs-directory))
```

这样就可以通过调用`ispell-word`来看一个单词是否正确了。如果是`evil`用户，这个函
数已经被绑定至<kbd>z=</kbd>上了。 \w/

## calendar

`Emacs`里日历可以拿来干什么呢？

第一个作用自然是看日期的，最起码得让今天醒目得吧？于是选择了在
`calendar-today-visible-hook`上加上`calendar-mark-today`。默认今天的日期是有下划
线的，如果不喜欢也可以自己修改`calendar-today-marker`。

第二个作用自然是看节日的，为了更加更本地化一点，可以设置一些自己想关注的节日。我
是这样设置的:

把较本土的节日放在了`holiday-local-holidays`里,

``` elisp
;; 分别是妇女节、植树节、劳动节、青年节、儿童节、教师节、国庆节、程序员节、双11
(setq holiday-local-holidays `((holiday-fixed 3 8  "Women's Day")
                               (holiday-fixed 3 12 "Arbor Day")
                               ,@(cl-loop for i from 1 to 3
                                          collect `(holiday-fixed 5 ,i "International Workers' Day"))
                               (holiday-fixed 5 4  "Chinese Youth Day")
                               (holiday-fixed 6 1  "Children's Day")
                               (holiday-fixed 9 10 "Teachers' Day")
                               ,@(cl-loop for i from 1 to 7
                                          collect `(holiday-fixed 10 ,i "National Day"))
                               (holiday-fixed 10 24 "Programmers' Day")
                               (holiday-fixed 11 11 "Singles' Day")))
```

再把其他没在默认日历里的放进`holiday-other-holidays`里,

``` elisp
;; 分别是世界地球日、世界读书日、俄罗斯的那个程序员节
(setq holiday-other-holidays '((holiday-fixed 4 22 "Earth Day")
                               (holiday-fixed 4 23 "World Book Day")
                               (holiday-sexp '(if (or (zerop (% year 400))
                                                      (and (% year 100) (zerop (% year 4))))
                                                  (list 9 12 year)
                                                (list 9 13 year))
                                             "World Programmers' Day")))
```

然后再开启`calendar`内置的中国节日支持:

``` elisp
(setq calendar-chinese-all-holidays-flag t)
```

这样就可以获得一个不错的日历体验了。如果自己还有农历节日需求的话，可以使用
`holiday-chinese`来定义。如

``` elisp
;; 元宵节
(setq holiday-oriental-holidays '((holiday-chinese 1 15 "Lantern Festival")))
```

当然元宵节已经默认被定义了，只需开启`calendar-chinese-all-holidays-flag`。

如果这还不够，还有[cal-china-x](https://github.com/xwl/cal-china-x)。

第三个功能也可以在`calendar`界面添加日记，默认的日记从功能上来说自然是不如
`org-mode`加持的丰富。请确保`org-agenda-diary-file`的值不是`'diary-file`，然后在
`calendar-mode-map`下调用`org-agenda-diary-entry`即可选择插入日记。

附图:

![org-agenda-add-entry-to-org-agenda-diary-file](https://emacs-china.org/uploads/default/original/2X/a/a2dbd0a45689694308f31218030d6f95cecb7d93.png)

![org-agenda-diary-file](https://emacs-china.org/uploads/default/original/2X/d/d8fd2964e8999f56e88c55b07043c804d02be37d.png)

需要注意，它默认不会自动保存`org-agenda-diary-file`。如果不喜欢这一点，可以利用
`advice`来修正一下。

``` elisp
(defun org-agenda-add-entry-with-save (_type text &optional _d1 _d2)
  ;; `org-agenda-add-entry-to-org-agenda-diary-file'里认为如果用户没有输入有效的
  ;; 内容，会弹出对应 buffer 让用户人工输入。
  (when (string-match "\\S-" text)
    (with-current-buffer (find-file-noselect org-agenda-diary-file)
      (save-buffer))))

(advice-add #'org-agenda-add-entry-to-org-agenda-diary-file :after #'org-agenda-add-entry-with-save)
```

我觉得这样子设置之后，可以轻度取代
[org-journal](https://github.com/bastibe/org-journal)了?
