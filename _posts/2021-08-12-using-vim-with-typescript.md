Creating a good workflow in (neo)vim to work with typescript is a mostly painless process in the year of our lord 2021. You want to make sure you have a few important features: syntax highlighting, a linter to find and fix syntax errors, and autocompletion using [Language Server Protocol](https://en.wikipedia.org/wiki/Language_Server_Protocol). Let's dive in and show how to set all that up.
<!--more-->

For all plugins listed in this post, it is recommended that you use [vim-plug](https://github.com/junegunn/vim-plug) to install and manage them.

## Syntax Highlighting
Vim comes with syntax highlighters for a wide variety of popular formats by default, but that can always be improved. The gold standard for syntax highlighters is [vim-polyglot](https://github.com/sheerun/vim-polyglot), which contains over 600 different language packs. To install, add the following to the plugin section of your `vimrc`:
```vim
" extra language plugins
Plug 'sheerun/vim-polyglot'
```

If you don't want to install all of that and only want to add highlighting for typescript, simply add [yats](https://github.com/HerringtonDarkholme/yats.vim) with the following line:
```vim
" for just typescript (in lieu of vim-polyglot)
Plug 'HerringtonDarkholme/yats.vim'
```

Syntax highlighting is on by default in vim, but you can ensure it is always on by including this line in your `vimrc`:
```vim
syntax on
```

## Linting
The most popular linting tool for javascript and typescript is [eslint](https://eslint.org/), the latter using the [typescript-eslint](https://github.com/typescript-eslint/typescript-eslint) plugin. Additionally, I like to use [prettier](https://prettier.io/) to clean up my code and enforce particular standards. The easiest way to use these is to install `eslint` and `prettier` at the system level (either through `npm install -g` or your system package manager), or on a per-project basis with `npm`.

`eslint` requires per-project configuration using the `.eslintrc.js` file at your project root, and that can be easily generated by running the command `eslint --init` from your project root. Simply answer the relevant prompts about the nature of your project and you will get an `eslintrc` file with some sane defaults, and npm will install any relevant plugins (like for typescript, react, and/or vue). For more on configuirng eslint on your project, see the [official docs](https://eslint.org/docs/user-guide/configuring/configuration-files).

When using `prettier` with typescript, one important thing is to have it use single quotes instead of double. This is easily accomplished by adding the following to your `package.json`:
```json
  "prettier": {
    "singleQuote": true
  }
```

I also like to use [stylelint](https://stylelint.io/) along with `prettier` to check my css files. like the preceding tools, you can either install it system-wide or on your project.

In order to get these tools working within vim, I use [ALE](https://github.com/dense-analysis/ale), aka Asynchronous Lint Engine. To install ALE, add the following line to your `vimrc` plugin section:
```vim
" ALE for linting - requires linters as system or project packages
Plug 'dense-analysis/ale'
```

ALE will highlight errors, and even fix them on save if configured properly. To use ALE with `eslint`, `prettier`, and `stylelint` for javascript, typescript, HTML, and CSS, add the following to your configuration:
```vim
" ALE settings
" use ale as a deoplete source
let g:ale_completion_enabled = 1

" fix on save
let g:ale_fix_on_save = 1

" use eslint, prettier, and stylelint
let g:ale_fixers = {
      \    'javascript': ['eslint', 'prettier'],
      \    'typescript': ['eslint', 'prettier'],
      \    'css': ['stylelint', 'prettier'],
      \    'html': ['prettier']
      \}

" use eslint for ts and js
let g:ale_linters = {
      \    'typescript': [ 'eslint' ],
      \    'javascript': [ 'eslint' ]
      \}
```

For more information on configuring ALE, check the [README](https://github.com/dense-analysis/ale) on github.

## Language Server Protocol (LSP)
While vim does include an autocompletion engine by default, a number of plugins improve on it consideribly. My favorite is [deoplete](https://github.com/Shougo/deoplete.nvim). To install deoplete, add the following lines to your plugin section:
```vim
" Deoplete
Plug 'Shougo/deoplete.nvim', { 'do': ':UpdateRemotePlugins' }
" show completion results from syntax highliting file
Plug 'Shougo/neco-syntax'
" lsp support for deoplete
Plug 'lighttiger2505/deoplete-vim-lsp'
```

To use LSP, a few more plugins are necessary. [This handy plugin](https://github.com/mattn/vim-lsp-settings) by Yashuhiro Matsumoto makes LSP installation super easy. All you need to do after installing it is run the command `:LspInstallServer` while a file in the format you want to install the LSP server is open in vim. To install, add the following plugins:
```vim
" vim lsp support
Plug 'prabirshrestha/vim-lsp'
" easy lsp server installation
Plug 'mattn/vim-lsp-settings'
```

## Final vimrc review
So, after all that, our `.vimrc` should contain the following:
```vim
call plug#begin()

" for a complete library of syntax highlighters (recommended)
Plug 'sheerun/vim-polyglot'

" for just typescript (in lieu of vim-polyglot)
Plug 'HerringtonDarkholme/yats.vim'

" ALE for linting - requires linters as system or project packages
Plug 'dense-analysis/ale'

" Deoplete
Plug 'Shougo/deoplete.nvim', { 'do': ':UpdateRemotePlugins' }

" show completion results from syntax highliting file
Plug 'Shougo/neco-syntax'

" lsp suport
Plug 'prabirshrestha/vim-lsp'

" Deoplete LSP support
Plug 'lighttiger2505/deoplete-vim-lsp'

" Easily install relevant LSP for current filetype
Plug 'mattn/vim-lsp-settings'

call plug#end()

" enable syntax highlighting
syntax on

" Use deoplete.
let g:deoplete#enable_at_startup = 1

" Deoplete mappings (use c-j and c-k to navigate selection)
inoremap <expr> <C-j> pumvisible() ? "\<C-n>" : "\<C-j>"
inoremap <expr> <C-k> pumvisible() ? "\<C-p>" : "\<C-k>"

" disable lsp syntax highlighting (it's ugly)
let g:lsp_diagnostics_enabled = 0

" ALE settings
" use ale as a deoplete source
let g:ale_completion_enabled = 1

" fix on save
let g:ale_fix_on_save = 1

" use eslint, prettier, and stylelint
let g:ale_fixers = {
      \    'javascript': ['eslint', 'prettier'],
      \    'typescript': ['eslint', 'prettier'],
      \    'css': ['stylelint', 'prettier'],
      \    'html': ['prettier']
      \}

" use eslint for ts and js
let g:ale_linters = {
      \    'typescript': [ 'eslint' ],
      \    'javascript': [ 'eslint' ]
      \}

" helpers for toggling fix_on_save
command! ALEDisableFixers       let g:ale_fix_on_save=0
command! ALEEnableFixers        let g:ale_fix_on_save=1
command! ALEDisableFixersBuffer let b:ale_fix_on_save=0
command! ALEEnableFixersBuffer  let b:ale_fix_on_save=1

" use bracket bindings to navgiate ale specific errors/warnings
nmap <silent> [r <Plug>(ale_previous_wrap)
nmap <silent> ]r <Plug>(ale_next_wrap)

```