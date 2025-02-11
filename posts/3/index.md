---
author: ["Marcelo Borges"]
title: "Supercharging My Clipboard with OSC52 Escape Sequence"
date: "2025-02-11"
summary: "Learn how to seamlessly sync your remote clipboard with your local system clipboard using OSC52 escape sequences."
tags: ["Neovim", "clipboard", "OSC52", "SSH", "terminal", "productivity", "CLI"]
categories: ["Tech", "Neovim", "Linux"]
series: ["Neovim Tips", "CLI", "Tips"]
ShowToc: true
TocOpen: true
---

## Introduction

My daily workflow involves connecting over SSH and using Neovim, but I struggled with clipboard integration.
I wanted a way to **copy text inside Neovim and sync it with my local clipboard** seamlessly.

However, copying and pasting between remote and local machines isn’t always straightforward.
Thankfully, **OSC52 (Operating System Command 52)** came to the rescue!

In this post, I'll introduce my **[`copy`](https://github.com/jmarcelomb/.dotfiles/blob/main/scripts/copy)** script that leverages OSC52,
explain how OSC52 works, and show how to integrate it into Neovim for a smooth experience.

---

## What is OSC52?

**OSC52** is a terminal escape sequence that allows you to copy text to your system clipboard from a remote session.
It works by encoding the copied text in **Base64** and sending it through a special terminal command,
which supported terminal emulators (such as
[Alacritty](https://github.com/alacritty/alacritty),
[Ghostty](https://github.com/ghostty-org/ghostty),
[Kitty](https://github.com/kovidgoyal/kitty),
[WezTerm](https://github.com/wezterm/wezterm))
detect and copy to your local clipboard.

This means that even when working on a remote machine over SSH,
you can copy text directly to your **local** clipboard without needing tools like `xclip`, `wl-copy`, or `xsel`.

---

## The Copy and Paste Scripts

To make clipboard management effortless, I created two simple scripts:
[`copy`](https://github.com/jmarcelomb/.dotfiles/blob/main/scripts/copy) and
[`paste`](https://github.com/jmarcelomb/.dotfiles/blob/main/scripts/paste).
You can find them in my **[.dotfiles](https://github.com/jmarcelomb/.dotfiles)** repository.

### Copy Script

The [`copy`](https://github.com/jmarcelomb/.dotfiles/blob/main/scripts/copy) script encodes the input in Base64
and sends it using OSC52. Here’s a simplified working version:

```bash
#!/bin/sh

printf '\033]52;c;%s\a' "$(cat | base64 | tr -d '\n')"
```

Usage:

```bash
echo "Hello, clipboard!" | copy
```

This sends **"Hello, clipboard!"** to your system clipboard, even over SSH! 😊!

### Paste Script

I also attempted to create a paste script, but due to terminal limitations,
I could only prompt for input when executing the paste script.

Example use case:

```bash
❯ echo "$(paste)" | grep "42"
📋 Enter input and press Enter (Ctrl+D to finish):
We can insert text, or paste, in a
multi-line format, and it will be print to stdout.
42 and when we want to exit, we just do
Ctrl+D
✅ Echoing the input to stdout!
42 and when we want to exit, we just do
```

---

## Integrating OSC52 with Neovim

To enable seamless copying inside **Neovim**, we need to:

1. Set the clipboard to `unnamedplus`
2. Use an **autocommand** to copy text using OSC52 whenever we yank (`yy`, `d`, `x`, etc.)

### Configuration for Neovim

Add the following to your `init.lua` or equivalent configuration file:

```lua
vim.o.clipboard = "unnamedplus"

vim.api.nvim_create_autocmd("TextYankPost", {
  callback = function()
    vim.highlight.on_yank()
    local copy_to_unnamedplus = require("vim.ui.clipboard.osc52").copy("+")
    copy_to_unnamedplus(vim.v.event.regcontents)
    local copy_to_unnamed = require("vim.ui.clipboard.osc52").copy("*")
    copy_to_unnamed(vim.v.event.regcontents)
  end,
})
```

### Explanation
- **`vim.o.clipboard = "unnamedplus"`** ensures that Neovim's default clipboard operations work with the system clipboard.
- **`TextYankPost`** event triggers whenever you copy (yank) text in Neovim.
- **`vim.highlight.on_yank()`** provides a visual cue that text was copied.
- **`vim.ui.clipboard.osc52.copy("+")`** and **`vim.ui.clipboard.osc52.copy("*")`** copy the yanked text to both `+` (system clipboard) and `*` (primary selection).

With this setup, anything you **yank** inside Neovim is automatically available in your system clipboard, even over SSH!

---

## Conclusion

These simple scripts and Neovim integration massively improved my workflow,
allowing seamless clipboard access over SSH.

I hope they help you as well! 😊

Give them a try, and let me know how they work for you. 🚀
