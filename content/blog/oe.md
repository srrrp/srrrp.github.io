+++
title = "OCaml Setup in Emacs"
date = 2023-05-17
+++


### Install OCaml

1. Install opam binary
2. Initialize opam
```
$ opam init
```
3. Create switch
```
$ eval $(opam env -switch=default)
```
4. Add this to `config.fish`
```
eval $(opam env)
```


### Install packages

- dune
- utop
- merlin
- ocaml-lsp-server

```
$ opam install dune utop merlin ocaml-lsp-server
```
	
	
### Setup Emacs

```el
(use-package tuareg
	:ensure t
	:mode (("\\.ocamlinit\\'" . tuareg-mode)))
	
(use-package merlin
	:ensure t
	:config
	(add-hook 'tuareg-mode-hook #'merlin-mode)
	(add-hook 'merlin-mode-hook #'company-mode))
	
(use-package merlin-eldoc
	:ensure t
	:hook ((tuareg-mode) . merlin-eldoc-setup))
	
(use-package flycheck-ocaml
	:ensure t
	:config
	(flycheck-ocaml-setup))
```


#### References

1. [Setting up Emacs for OCaml Development](https://batsov.com/articles/2022/08/23/setting-up-emacs-for-ocaml-development/)
2. [Tuareg](https://github.com/ocaml/tuareg)