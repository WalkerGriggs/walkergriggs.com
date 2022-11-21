+++
title = "A Year with Emacs"
author = ["Walker Griggs"]
date = 2017-01-05
draft = false
creator = "Emacs 28.1 (Org mode 9.5.2 + ox-hugo)"
weight = 2004
+++

<span class="underline">It is important to preface that everything in this article is opinion and based off (roughly) a year of heavy Emacs usage. It is also important to know that this article will be updated along side my configuration and tastes. So without further ado...</span>

We all know Emacs is an immensely powerful beast. We also know how easy it is to venture down a rabbit hole of elisp and never surface. I liken it to a carpenter replacing a door. After removing the old door, he notices the hinges are askew. He removes the hinges only to notice rot in the door frame. By the time he replaces the frame, he notices a slight difference in shade between the new frame and old moldings... The learning curve for Emacs is wonderfully circular. That being said, I would like to take a moment and explain my configuration in moderate detail.

Before I get too technical, I should probably explain my fascination and reservation with Emacs. Brief background: I was forced into using Emacs when the only other editor on the lab machines was Gedit (and Vi, but we'll forget about that for now). In all honestly, it was quite a hassle. I began compiling a minimal init.el out of necessity. Linum, flyspell, you name it. It was certainly a gradual transition from cushy Atom, but, after a long while, it became an addiction. It wasn't until I discovered a keyboard designed with Emacs in mind (Atreus) did I see Emacs (and the devoted community) in all of its glory.

As for my reservations...

> The learning curve is far too steep. My time is best spent elsewhere.

WRONG. The weeks of struggling with Meta keys and Emacs pinkie pays off. Trust me. My workflow has increased substantially, and I feel extraordinarily comfortable in my configuration. Granted, emacs is truly a lifestyle. Embrace it.

> It's a bloated editor packed with legacy functionality. The startup time is just too long!

MYTH. You think Emacs is too heavy for you system? Try running Eclipse and Chrome simultaneously and then get back to me. As long as your config file is optimized (cough cough 'use-package'), the startup time won't be longer than a couple of seconds. Granted, on a system with limited resources, Vi may be a better option. Which brings me to my biggest qualm. Vi is an editor. Emacs is an editor AND IDE. When remoting into a server, I'm not about to Xforward a fully functional Emacs when bandwidth and memory are scarce. For that reason, I keep a modest .vimrc on hand for some quick cli editing.


## Configuration {#configuration}


### melpa and use-package {#melpa-and-use-package}

Melpa is a very common package manager for Emacs. I try not to rely on it, though it certainly comes in handy. The simple (and recommended) solution...

```lisp
;; Melpa
(require 'package)
(setq package-enable-at-startup nil)
(add-to-list 'package-archives
  '("melpa" . "https://melpa.org/packages/"))
(add-to-list 'package-archives
  '("melpa-stable" . "http://stable.melpa.org/packages/"))
```

Now it wasn't until a friend picked through my config when I learned about 'use-package'. UP is a wonderful macro written by John Wiegley that declares and isolates packages in your config. Each package can then be initialized, configured, and bound independently. This is a must use...

```lisp
;; Bootstrap 'use-package'
(unless (package-installed-p 'use-package)
    (package-refresh-contents)
    (package-install 'use-package))
(setq use-package-verbose t)
```


### tabs / whitespace {#tabs-whitespace}

The next few go hand in hand: tabs and whitespace. I'd like to reiterate, these are simply opinions. Feel free to disagree, but I cannot stand tabs in my code. Tab size varies across environments but a space will ALWAYS be one column. Case closed. That being said, tab functionality is quite nice, so I've turned indent-tabs-mode to nil. Simply...

```lisp
(setq-default indent-tabs-mode nil)
(setq-default tab-width 2)
```

The next is an acquired taste: whitespace-mode. Ever since I properly configured my whitespace (invisibles) to be tastefully visible, I've grown to appreciate the subtly clean code. Trailing whitespace / unnecessary new lines have since disappeared.

```lisp
;; Whitespace
(use-package whitespace
    :bind (("C-c C-w" . whitespace-mode))
    :init
    (dolist (hook '(prog-mode-hook text-mode-hook conf-mode-hook))
        (add-hook hook #'whitespace-mode))
    :config
    (add-hook 'prog-mode-hook 'whitespace-mode)
    (global-whitespace-mode t) ;; Whitespace ON.
    (setq whitespace-global-modes '(not org-mode))
    (setq whitespace-line-column 80) ;; Set indent limit.
    (setq whitespace-display-mappings
    '(
        (space-mark 32 [183] [46])
        (newline-mark 10 [172 10])
        (tab-mark 9 [9655 9] [92 9]))))
```

Here, I've remapped the display for the space, newline, and tab to suit my taste. Whitespace is shown on pretty much every mode except org (where it really is never needed). Other than that, lines over 80 columns are highlighted. Simple and lovely.


### helm {#helm}

Helm is a package that I never knew I needed, until I started using it. It's described as an incremental completion and selection narrowing framework. Essentially, it gives me proper control over buffers, files, and commands similar to Smex (with a Neotree feel). Helm, however, is capably of out of order regex matching which is surprisingly uncommon.

Here, I've remapped the helm key bindings to reflect standard C-x C-f / tab-complete functionality.

```lisp
;; Helm
(use-package helm
    :ensure t
    :bind
    (("M-x" . helm-M-x)
    ("C-x C-f" . helm-find-files))
    :config
    (setq helm-split-window-in-side-p        t  ;; opens helm inside window
          helm-move-to-line-cycle-in-source  t
          helm-autoresize-min-height         20
          helm-autoresize-max-height         40
          helm-scroll-amount                 8)
    (define-key helm-map (kbd "<tab>") 'helm-execute-persistent-action)
    (define-key helm-map (kbd "C-z") 'helm-select-action)
    (setq helm-mode-fuzzy-match t))
```


### org {#org}

Org-mode might be one of the most expansive and powerful features of emacs. It is perfect for daily organization, notes, etc. Recently, I've adopted the org-clock, which can time tasks and generate useful reports. I may not be a freelancer who charges by the hour, but it certainly keeps me on track and focused.

```lisp
;; Org
(use-package org
    :ensure t
    :mode (("\\.org$" . org-mode))
    :bind (("C-c C-x C-i" . org-clock-in)
           ("C-c C-x C-o" . org-clock-out)
           ("C-c C-x C-j" . org-clock-goto)
           ("C-c C-x C-r" . org-clock-report))
    :config
    (progn
        (define-key org-mode-map "\M-q" 'toggle-truncate-lines)
        (setq org-directory "~/org")
        (setq org-clock-persist t)
        (setq org-clock-mode-line-total 'current)))
```

While these snippets are not my configuration in it's entirety, the full file is not a hulking mass. It can be found at in my [dotfiles repo](https://github.com/WalkerGriggs/DotFiles/blob/master/.emacs). Feel free to take and modify what you need. If you have anything to contribute, feel free to shoot me an em