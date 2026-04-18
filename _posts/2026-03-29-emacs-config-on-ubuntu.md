---
title: Linux용 Emacs 설치 환경설정
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - Emacs
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---

### .emacs 설정파일

```elisp
(defalias 'yes-or-no-p 'y-or-n-p)

(setq inhibit-startup-screen t)
(setq make-backup-files nil) ; stop creating backup~ files
(setq auto-save-default nil) ; stop creating #autosave# files

(setq-default c-basic-offset 8)

;; fontify code in code blocks
(setq org-src-fontify-natively t)

(load-theme 'tango-dark)

;; GUI에서 입력함
;; 한글 폰트 설정 (예: 나눔고딕코딩, 크기 14)
(set-fontset-font "fontset-default" '(#x1100 . #xffdc) '("NanumGothicCoding" . "unicode-bmp"))
(set-fontset-font "fontset-default" '(#x3130 . #x318f) '("NanumGothicCoding" . "unicode-bmp"))
(set-fontset-font "fontset-default" '(#xac00 . #xd7a3) '("NanumGothicCoding" . "unicode-bmp"))

(set-language-environment "Korean")
(prefer-coding-system 'utf-8)
(global-set-key (kbd "<S-kana>") 'toggle-input-method)
(global-set-key (kbd "<kana>") 'toggle-input-method)

(defun xftp (&optional frame)
"Return t if FRAME support XFT font backend."
(let ((xft-supported))
(mapc (lambda (x) (if (eq x 'xft) (setq xft-supported t)))
(frame-parameter frame 'font-backend))
xft-supported))
(when (xftp)
(let ((fontset "fontset-default"))
(set-default-font "Inconsolata")
(set-face-font 'default "Inconsolata")
(set-fontset-font fontset 'hangul
("NanumGothic" . "unicode-bmp"))
(set-face-attribute 'default nil
font fontset
:height 120)
))

;; ELPA
;; Package system
(require 'package)

(setq package-archives '(("gnu" . "https://elpa.gnu.org/packages/")
                         ("marmalade" . "https://marmalade-repo.org/package/")
                         ("melpa" . "https://melpa.org/packages/")))
(package-initialize)
;(package-refresh-contents)
;(setq auto-install-packages
;      '(color-theme bar-cursor htmlize flycheck flycheck-haskell
;		    haskell-mode sml-mode rust-mode fsharp-mode nasm-mode go-mode
;		    perl-mode web-mode )) ;;ffap-perl-module markdown-mode))
;(dolist (pkg auto-install-packages)
;  (unless (package-installed-p pkg)
;        (package-install pkg)))

(global-set-key (kbd "<f12>") 'org-todo-list)
(global-set-key (kbd "<f11>") 'org-agenda)
(global-set-key (kbd "C-c c") 'org-capture)

;(setq org-archive-mark-done nil)
;;(setq org-archive-location "~/Dropbox/orgzly/archive.org::* Archived Tasks")
(setq org-archive-location "~/Dropbox/orgzly/archive.org::")

;; C-x C-w : refile
(setq org-refile-targets '(
			   ("archive.org" :level . 1)
			  )
)

(global-set-key (kbd "<f12>") 'org-todo-list)

(org-babel-do-load-languages
 'org-babel-load-languages
 '((emacs-lisp . nil)
      (C . t)))

(setq browse-url-browser-function 'browse-url-generic
            browse-url-generic-program "google-chrome")
(custom-set-variables
 ;; custom-set-variables was added by Custom.
 ;; If you edit it by hand, you could mess it up, so be careful.
 ;; Your init file should contain only one such instance.
 ;; If there is more than one, they won't work right.
 '(package-selected-packages
   (quote
    (tango-2-theme tango-plus-theme tangonov-theme tangotango-theme))))
(custom-set-faces
 ;; custom-set-faces was added by Custom.
 ;; If you edit it by hand, you could mess it up, so be careful.
 ;; Your init file should contain only one such instance.
 ;; If there is more than one, they won't work right.
 )
```

## Related Posts
- [Windows용 Emacs 설치 환경설정]({% link _posts/2026-02-28-emacs-windows-setup.md %})
- [Orgmode 사용하는 기능 노트]({% link _posts/2026-03-04-orgmod-note.md %})
