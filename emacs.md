基本操作速记：
C,代表CTRL键
M,代表Meta键，往往是ALT(部分键盘是ESC)

光标移动，形成肌肉记忆之前可以通过速记法记忆：

- C-n,下一行
- C-p,前一行
- C-f,向前移动一个字符
- C-b,向后退一个字符
- C-k,从光标位置到末尾删掉
- C-a,回到行首
- C-e,去到行尾
- M-<,回到编辑区域最开始的位置
- M->,去往编辑区组后的位置
- C-v,向下翻一屏
- M-v,向上翻一屏

自带文档：
C-h t;速记Help Tutorial
查看快捷键的含义
C-h k;速记Help Keybind
查看函数的定于以及快捷键的绑定
C-h f;速记Help Function

對外觀做點改變：
圖形化配置：
M-x customize
菜單欄
menu-bar-mode
工具欄：
tool-bar-mode
滾動條：
scroll-bar-mode

優勢：不用寫Elisp代碼
劣勢：
- 搜索
- 生成的配置代碼只能是單一的文件配置，顯得凌亂

配置文件：
創建文件 ~/.emacs並寫入以下內容：
```
(menu-bar-mode -1)
(tool-bar-mode -1)
(scroll-bar-mode -1)
```

配置文件位置，會按照以下順序去查找：
- ~/.emacs
- ~/.emacs.d
- ~/.config/emacs/init.el
第一個是一個單一的文件配置，第二個更符合工程化，第三個僅適用於>=27的版本

業界出名配置：
- spacemacs
- centaur emacs
- doom emacs
- Steve purcell
- redguidetoo(陳斌，一年成為emacs高手)
- ...

關閉啟動界面：
```
(setq inhibit-startup-screen t)
```

軟件源：
```
(setq package-archives '(("gnu"   . "http://mirrors.tuna.tsinghua.edu.cn/elpa/gnu/")
                         ("melpa" . "http://mirrors.tuna.tsinghua.edu.cn/elpa/melpa/")))
(package-initialize) ;; You might already have this line
```

Emacs擴展：
```
(setq package-check-signature nil);個別時候會出現簽名校驗失敗
(require 'package)
;;初始化包管理器
(unless (bound-and-true-p package--initialized)
   (package-initialize))
;;刷新軟件源索引
(unless package-archive-contents
	(package-refresh-contents))


(unless (package-install-p 'use-package);;除非已經安裝了
	(package-refresh-contents)
	(package-install 'use-package))
```

使用use-package管理擴展：

```
;;最簡單的格式
(require 'use-package)
(use-package restart-emacs)
;;常用的格式
(use-package smooth-scrolling
	     :ensure t ;是否一定要確保已安裝
	     :defer nil ;是否延遲加載
	     :init (setq smooth-scrolling-margin 2) ;初始化參數
	     :config (smooth-scrolling-mode t);基本配置參數
	     :bind ;快捷鍵的綁定
	     :hook ;hook模式的綁定

;;建議添加的配置
(eval-and-compile
	(setq use-package-always-ensure t) ;不用每個包都手動添加:ensure t 關鍵字
	(setq use-package-always-defer t)  ;默認都是延遲加載,不用每個包都手動天啊及:defer t
	(setq use-package-always-demand nil)
	(setq use-package-expand-minimally t)
	(setq use-package-verbose t)
```

更換主題:

(use-package gruvbox-theme
	     :init (load-theme 'gruvbox-dark-soft t))

配置好看點的mode line
(use-package smart-mode-line
	     :init
	     (setq sml/no-confirm-load-theme t)
	     (setq sml/theme 'respectful)
	     (sml/setup))

工程化管理配置：功能拆分：

文件加載與調用：

先設置加載的目標目錄到load-path中：
(add-to-list 'load-path (expand-file-name (concat user-emacs-directory "lisp")))

各個文件通過provide暴露對外調用的名稱
(provide 'init-ui)

然後在另一個文件中通過require調用：
require 'init-ui

操作系統的判斷：
(defconst *is-mac*(eq system-type 'darwin))
(defconst *is-linux*(eq system-type 'gnu/linux))
(defconst *is-windows*(or (eq system-type 'ms-dos)(eq system-type 'windows-nt))


通過修改字體解決windows上的emacs卡頓
(use-package emacs
	     :if (display-graphic-p)
	     :config
	     (if *is-windows*
	     (progn
	     (set-face-attribute 'default nil :font "Microsoft Yahei Mono 9")
	     (dolist (charset '(kana han symbol cjk-misc bopomofo))
	     	     (set-fontset-not (frame-parameter nil 'font)
		     		       charset (font-spec:family ""Microsoft Yahei Mono" :size 12))))
	     (set-face-attribute 'default nil :font "Source Code Pro for Powerline 11")))


中断命令输入

C-g

用y/n来代替yes/no

(defalias 'yes-or-no-p 'y-or-n-p)

显示行号：
通过设置display-line-numbers-type变量的值，来改变行号的类型

relative, 相对行号
visual, 可视化行号
t

行号配置
(use-package emacs
	     :config
	     (setq display-line-numbers-typ 'relative)
	     (global-display-line-numbers-mode t))

选中
方法一：使用鼠标拖动选中
方法二：默认使用C-CSP或者C-@进行开始位置标记
复制
M-w
剪切
C-w
粘贴
C-y (yank)

从光标位置到行尾的删除
C-k

删除当前行

可以使用扩展来实现crux，使用快捷命令

(use-package crux
	     :bind("C-c k" . crux-smart-kill-line))

一次性删除到前/后的第一个非空字符(删除连续的空白)
通过hungry-delete扩展功能实现
(use-package hungry-delete
	     :bind(("C-c DEL" . hungry-delete-backward))
	     :bind(("C-c d" . hungry-delete-forward)))

文本编辑之行、区域上下移动
通过插件drag-stuff来辅助实现
(use-package drag-stuff
	     :bind(("<M-up>" . drag-stuff-up)
	     	   ("<M-down>" . drag-stuff-down)))

搜索
当前位置到文件尾：C-s

当前位置到文件头：C-r

替换 M-%(替换時按下y確認替換，n跳過本處的替換，!全部替換)

強化搜索：
使用著名的ivy-counsel-swiper三劍客，就是為了強化搜索（同時優化了一系列的Minibuffer的操作）

