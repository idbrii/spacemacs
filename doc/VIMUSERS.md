This doc helps resolve:
https://github.com/syl20bnr/spacemacs/issues/773

Emacs works a lot like vim-unite: you can get a incrementally-filtered
list of possible actions. For example, typing `SPC ?` gives you a list
of all mappings (called keybindings in emacs) and you can type 'evil'
to see just the evil ones. Usually when you're presented with a list
of options, you'll get an autocompleting filtered list.

If you get in trouble, just keep hitting Esc to return to normal
mode. If you see `-- EMACS --` at the bottom of emacs, hit `C-z` to
return to evil mode.

IMPORTANT: Mappings below are defined using functions in the
Customising key bindings section. (Eventually, I'd like to get them
included in spacemacs/evil.)

# Differences

## Differences in defaults

Defaults that start with dotspacemacs can be changed in dotspacemacs/init. The variable is already defined, you just need to change the value.

Other settings need to be added to dotspacemacs/config as a line of lisp code.

Many settings can be toggled at run time with `SPC t <setting>`. Try typing `SPC t` and see the guide-key window on the right.

* Leader is Space

   dotspacemacs-leader-key "SPC"

* LocalLeader is comma

   dotspacemacs-major-mode-leader-key ","

* 'cursorline' is enabled

    (global-hl-line-mode -1)
    SPC t h

* Yanking modifies the clipboard

Spacemacs acts like `clipboard=unnamed` and every yank is put on the system clipboard. You can disable this behavior and still use the `+` register to yank to the clipboard:

  (setq x-select-enable-clipboard nil)

* 'wrap' is set, but 'linebreak' is not

In vim, if you use 'wrap' and 'linebreak', you get softwrapped text that breaks on word boundaries. Spacemacs gives you just softwrapped text.

## Differences in normal mode
* C-l scrolls buffer down instead of redrawing
* C-w C-q doesn't quit a buffer

    (evil-nmap "C-w q" 'evil-window-delete)

## Differences in visual mode
* \* searches for selection instead of word under cursor. This behavior is similar to [vim-visual-star-search](https://github.com/bronson/vim-visual-star-search). To get something close to the vanilla vim behavior (but it searches for partial words):

    (evil-vmap "*" 'evil-search-unbounded-word-forward)
    (evil-vmap "g *" 'evil-visualstar/begin-search-forward)

## Differences in insert mode
* `C-x C-f` does not do filename completion.

It invokes dired, but dired just jumps to the selected file. You can use `SPC f y` to yank the filename of the selected file after jumping to it, and then `SPC bb` to go back to your original file and paste.

* `C-u` does not clear the line before the cursor.

[This answer](http://stackoverflow.com/a/3888306/79125) demonstrates how to implement a kill-line function as a lambda.

    (evil-imap "C-u" '(lambda () (interactive) (kill-line 0)))

If you `inoremap <C-U> <C-G>u<C-U>` so that you can undo the deletion separately from what happens around it, then try this instead:

    (defun backward-kill-line ()
        "Kill from cursor to beginning of line.
            via http://stackoverflow.com/a/3888306/79125"
        (interactive)
        (undo-boundary) ;; Similar to vim's `C-g u`, this creates an undo point
        (kill-line 0))
    (evil-imap "C-u" 'backward-kill-line)


## Differences in search cmdline
Pressing `/` enters search cmdline like in vim.
* you cannot press Up/Down to navigate history.
* you cannot press C-f to go to a buffer of search history.
* C-w does not erase a word
* C-u does not clear before the cursor


## Helm Minibuffer
While cmdline (pressing :) still works like in vim, the emacs filter view (helm minibuffer?) does not:

* pressing tab does not autocomplete
 * instead you use Right Arrow
 * if you hit tab, you can get back with tab again
* C-w does not erase a word. You can fix with:

    (define-key helm-map (kbd "C-w") 'backward-kill-word)

* C-u does not clear before the cursor
* C-r does not work (inserting a word, file, etc under cursor)


# Common Vim Changes
There are some maps/settings that are so common that they're suggested by the vim help or implemented in [vim-sensible](https://github.com/tpope/vim-sensible). Here's how you do them in spacemacs.

* `nnoremap Y y$`

Make Y consistent with C and D by operating from cursor to end of line.

    (evil-nmap "Y" 'evil-copy-to-end-of-line)
  
* `nnoremap Q gw`

Use Q for formatting. To map to the `gq` equivalent (reposition cursor) instead of `gw` (format without moving cursor), use `'evil-fill-and-move`.

    (evil-nmap "Q" 'evil-fill)

* `nnoremap <C-q> <C-v>`

Use C-q to start block selection.

    (evil-nmap "C-q" 'evil-visual-block)

* `nnoremap cw dwi`

From :help cw

# Lingo

maps/mappings: keybindings. They're always listed with a space between each keystroke.

SPC: the space bar

RET: the enter key (CR)


# Basic orientation
## Using Emacs' help system

Run vimtutor:

    SPC h T

See keybindings:

    SPC ?

helm-apropos is similar to :help, but uses incremental search:

    SPC <F1>
    (hit RET or C-z to see help for selected item)

Use Emacs help:

    C-h C-h
    :help


## Running commands

Most vim commands are available. Some will look a little alien (like `:`), but ignore the extra text and keep going (it clears "default" after you start typing).

To open list of commands:

    SPC :

Now you can start typing (with spaces to break up tokens) and it will filter to matching commands.

## Switching buffers

`:ls` is different. Instead of just listing buffers, it creates a new buffer with a list of other buffers called buffer-menu. Press CR to open a buffer and q to quit.

Open (switch) to a buffer with helm:

    SPC b s


## Killing buffers
## Opening files

Helm is good for this, but I don't understand how to use/configure it.

    SPC h

## Saving files

    :w
    SPC f s

# Insert completion

# Creating the .spacemacs

Spacemacs will generate a .spacemacs for you:

    SPC : dotspacemacs/install RET

You need to move it to your dotfiles and symlink it yourself.

At any time, you can edit your spacemacs with:

    SPC f e d

# Customising key bindings

You map keys with evil-define-key. If you had surround mapped to a different key, you could use this to restore s to its default behavior and map `c` as your surround key (in visual mode only):

    (evil-define-key 'visual evil-surround-mode-map "s" 'evil-substitute)
    (evil-define-key 'visual evil-surround-mode-map "c" 'evil-surround-region)

Here's a set of consistent functions to make it easier to map keys:

    (defsubst evil-leader-nmap (key command)
      (evil-leader/set-key key command))

    (defsubst evil-map (mode key command)
      (define-key mode (kbd key) command))

    (defsubst evil-vmap (key command)
      (evil-map evil-visual-state-map key command))

    (defsubst evil-nmap (key command)
      (evil-map evil-normal-state-map key command)
      (evil-map evil-motion-state-map key command))

    (defsubst evil-imap (key command)
      (evil-map evil-insert-state-map key command))

[kbd and other info](http://emacslife.com/read-lisp-tweak-emacs/beginner-3-make-things-more-convenient.html#sec-1-2)

defsubst is like defun, but it makes the functions inline (since these are so small, it's worth inlining them). Put them in your dotspacemacs above your dotspacemacs/config function (not inside the function). Call them inside your dotspacemacs/config function like this:

    (evil-vmap "g *" 'evil-visualstar/begin-search-forward)

To create more advanced bindings that chain multiple commands together, you must create an interactive function:

    ;; A function to collect together three behaviors:
    ;;  * returning to normal mode (<Esc>)
    ;;  * removing search highlighting (:nohlsearch)
    ;;  * redrawing the display (<C-l>)
    (defun evil-david-clear-screen ()
      (interactive)
      (evil-normal-state)
      (evil-search-highlight-persist-remove-all)
      (redraw-display))

    ;; Remove the old keybinding so you don't accidentally use it outside of
    ;; evil mode.
    (global-unset-key (kbd "C-l"))
    ;; Equivalent to :nnoremap <C-l> <Esc>:nohlsearch<CR><C-l>
    (evil-nmap "C-l" 'evil-david-clear-screen)
    ;; Equivalent to :inoremap <C-l> <Esc>:nohlsearch<CR><C-l>
    (evil-imap "C-l" 'evil-david-clear-screen)

Functions called from a binding should always have (interactive) first. You can use helm-descbinds (`SPC ?`) to see existing bindings to determine what functions to call. Emacs bindings only bind to functions (not other key inputs) so they are never recursive and do not require a noremap equivalent. (Just don't write unguarded recursive functions.)



# Creating a first configuration layer
# Basic package config with use-package
