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


defsubst is like defun, but it makes the functions inline (since these are so small, it's worth inlining them). Put them in your dotspacemacs above your dotspacemacs/config function (not inside the function). Call them inside your dotspacemacs/config function like this:

    (evil-vmap "g *" 'evil-visualstar/begin-search-forward)


[kbd and other info](http://emacslife.com/read-lisp-tweak-emacs/beginner-3-make-things-more-convenient.html#sec-1-2)




# Creating a first configuration layer
# Basic package config with use-package
