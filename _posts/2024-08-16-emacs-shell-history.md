---
layout: post
title: Distinct persistent bash history files using emacs shell
author: Thomas E. Enebo
email: tom.enebo@gmail.com
---

# Introduction

Many people use bash and many people use emacs but not many people use
emacs shell.  I decided on this latest laptop refresh to address
some development ergonomics.  In this case, I tend to have different
emacs shell sessions in which I type different types of commands.
If I reboot or restart emacs it would be nice if the `jruby` shell has
my old JRuby command history and my `mri` shell has old C Ruby command
history.  No mixing of history between shells or complete destruction of
history (e.g.last shell closed wins at writing the file).

Talk about a post made for an obscure Venn diagram of users :smiley:  Yet,
here I am helping out the dozens of netizens who may want this.

# Bash Configuration

This is the extremely easy to lookup stuff.  I want a large history and I
want that history to always append.  For good measure let's weed out
duplicate commands to make running `history` show more useful
context (~/.bashrc):

```sh
export HISTSIZE=100000
export HISTFILESIZE=100000
export HISTCONTROL=ignoredups:erasedups
shopt -s histappend
```

These are sensible settings and I would think most people use these
settings if they have ever decided to configure how their shell history
works.

The piece of magic on the bash side is to set `HISTFILE` for each named
shell:

```sh
# If using emacs retain history for each named buffer so that future
# sessions will retain just that buffers history.
if [ ! -z "${EMACS_SHELL_NAME}" ]; then
  export HISTFILE=".bash_history_${EMACS_SHELL_NAME}"
fi
```

If we are starting a new shell and the environment variable `EMACS_SHELL_NAME` exists then it gets its own history file rather than using the default history file.  Also since we `shopt -s histappend` we will just keep building that history file up.  Neat...Except emacs does not set an `EMACS_SHELL_NAME` variable.  Let's figure that side out...

# Emacs Configuration

To enter shell mode you run `(shell)` which, by default, makes a shell run in a buffer called `*shell*`.  No one wants `*shell*` as a buffer name and so far as I can tell WE ALL make a similar interactive function (.emacs):

```lisp
(defun make-shell (buffer-name) "Create a shell buffer with specific name"
  (interactive "sShell Name: ")
  (let ((buffer (generate-new-buffer buffer-name)))
    (setenv "EMACS_SHELL_NAME" buffer-name)
    (shell buffer)))
```

This prompts for a buffer name, then makes it, and passes it as an optional parameter to the shell function.  We get a new named buffer with a shell running.

The one line I added to connect this all together is `(setenv "EMACS_SHELL_NAME" buffer-name)`.  With that line our newly created shells have the info they need to create the right `HISTFILE` in .bashrc.

# Conclusion

This ended up being simple to wire up which made me wonder why I
didn't do this like 30 years ago?  Probably because it was only a tiny bit
annoying and I didn't care enough to fix it.  Maybe it was just that I had
to wire this up in multiple files and it was more like making a feature
than configuring one.

One thing I do know is that I have wished many many
times that I still had an old command in my history only to realize it
was overwritten by some inconsequential shell history.

In any case if anyone ever searches for this...you're welcome.
