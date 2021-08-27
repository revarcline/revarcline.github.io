Just wanted to share a handy little script to create a quick apt package search with [fzf](https://github.com/junegunn/fzf) in debian or ubuntu. Put the following in your shell rc file:

```bash
# apt with fzf
alias aptfzf="sudo true && apt list | sed 1d | cut -d/ -f1 | fzf -m --preview 'apt-cache show {1}' | xargs -ro sudo apt install"
alias aptrmfzf="sudo true && apt list --installed | sed 1d | cut -d/ -f1 | fzf -m --preview 'apt-cache show {1}' | xargs -ro sudo apt remove" 
```

Simply run `aptfzf` and you can live fuzzy-search packages to install, or `aptrmfzf` to search installed packages to remove.
