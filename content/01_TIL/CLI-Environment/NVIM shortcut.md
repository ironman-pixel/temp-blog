---
date: 2025-08-01
tags:
  - til
  - dev-tools
  - nvim
---
# ❓ Information
* vim shortcut
* neovim shortcut

---

# 🔰 Content ->  
## vim
### copy to Clipboard
- To copy the current line, in command mode type:
```
"*yy
```

- To copy the whole file/buffer, in command mode, first go to the beginning via `gg`, then type
```
"*yG
```

---
Two sets of shortcut to remember: `"+yy` (copy line to clipboard) and `"+yy` (copy line to selection); `"+p` (paste from clipboard) and `"*p` (paste from selection). `"` is to select register which is vim's own internal register by default (the way `yy` and `p` would work without referencing **any** type of register).