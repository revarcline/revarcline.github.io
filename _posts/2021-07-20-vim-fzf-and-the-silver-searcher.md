[vim](https://www.vim.org) (or if you're like me, [neovim](https://neovim.io)) is a venerable gold standard of text editing, though nefarious for its learning curve. Perhaps less well-known to the brave few who booted it up for the first time and didn't know how to quit, is its incredible extensibility. Today I want to bring up one of those extensions that can make vim feel like a modern IDE: the [fzf plugin](https://github.com/junegunn/fzf.vim).
<!--more-->

[fzf](https://github.com/junegunn/fzf) is a speedy little command-line search tool that helpfully allows users to fuzzy-find files or entries from nearly any given text source. It includes a preview feature, as well as hotkeys to speed up navigation. You can easily install it from your system's package manager on just about any distro. After installing it, if you're using [vim-plug](https://github.com/junegunn/vim-plug) (by the same developer, Junegunn Choi), you can install the relevant plugins and point to your system executable by adding these lines to your `vimrc` in between `call plug#begin()` and `call plug#end()`:

```vim
Plug '/usr/bin/fzf'  "point to system binary
Plug 'junegunn/fzf'
Plug 'junegunn/fzf.vim'
```

If you'd prefer not to use the system binary or don't have admin privileges on your system, you can have plug install fzf locally in your vim plugins directory:

```vim
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }
Plug 'junegunn/fzf.vim'
```

Save your vimrc, then `:source %` and `:PlugInstall` to add fzf functionality to vim!

fzf is probably my favorite way to switch between files and buffers from a project root. All my fzf-related keybindings begin with the `<Leader>` key, which opens up a whole new layer of hotkey configuration in vim. I set my `<Leader>` key to space, but it defaults to `\`. Setting it to space is a little tricky, as you have to deactivate the original functionality like so:

```vim
" Set leader to space
nnoremap <SPACE> <Nop>
let mapleader = " "
```

Now that the Leader key is set, let's go through some of my go-to fzf bindings. First, here's a handy way to switch between all your open buffers by hitting `leader` then `b`:

```vim
" search between open buffers
nnoremap <Leader>b :Buffer<CR>
xnoremap <Leader>b :Buffer<CR>
```

Next, I set `leader ;` to pull up a fzf menu with all available commands in vim, which can be incredibly handy when you don't remember exactly what to type:

```vim
" search all :commands
nnoremap <Leader>; :Commands<CR>
xnoremap <Leader>; :Commands<CR>
```

This next one is probably my most common use-case of `vim-fzf`. Here you can fzf against all filenames from the project root to quickly find and open whatever it is you're looking for. `ctl-p` searches for all files, while `leader p` searches specifically within the project's git repo if available (allowing you to skip all those pesky `node-modules` and whatever else is in your `.gitignore`):

```vim
" search all files from root directory
nnoremap <C-p> <esc>:Files<CR>
xnoremap <C-p> :Files<CR>

" same, but only for files in git
nnoremap <Leader>p <esc>:GFiles<CR>
xnoremap <Leader>p :GFiles<CR>
```

I added some slight tweaks (from the project `README`) to make sure things look pretty. The first section adds an attractive syntax-highlighted preview, and the latter makes sure things appear more clearly in the floating buffer.

```vim
" FZF with preview for files
command! -bang -nargs=? -complete=dir Files
    \ call fzf#vim#files(<q-args>,
    \ fzf#vim#with_preview({'options': ['--layout=reverse', '--info=inline']}),
    \ <bang>0)

" Hide cmd buffer for fzf
autocmd! FileType fzf
autocmd  FileType fzf set laststatus=0 noshowmode noruler
  \| autocmd BufLeave <buffer> set laststatus=2 showmode ruler
```

Finally, let's talk [silver searcher](https://github.com/ggreer/the_silver_searcher), aka `ag`. Like fzf, you can install this from your system's package manager. `ag` is a lightning-fast text search that will actually dive into your files and return the source. As you might imagine, for a medium-sized project, this can be incredibly handy. It's already integrated with fzf, and here are my bindings to use it:

```vim
" call silver searcher
nnoremap <Leader>ag :Ag<CR>
xnoremap <Leader>ag :Ag<CR>

" search ag for word under cursor
nnoremap <Leader>k :Ag <C-R><C-W><CR>
```

The last line is especially useful. With `leader k`, I can easily search the entire project's code for the term under my cursor, and perform any follow-up subsearch I need with fzf.

Thanks to the extensibility of vim, you can get the sort of functionality you'd expect from a modern IDE like vscode, but with a fraction of the resource footprint and with a blazing-fast keyboard-based workflow. If you haven't tried it yet, it's always a great idea to start!
