---
date: 2025-12-09
tags:
  - cli
  - terminal
  - til
  - dev-tools
  - nvim
---
# ❓ Information
* nvim 사용시 md 파일에서 한글에 빨간줄이 뜸

---

# 🔰 Content ->  
/nvim/lua/config/options.lua
```lua
vim.opt.spelllang = "en,cjk" 
```

/nvim/lua/plugins/markview.lua
```
return {
  "OXY2DEV/markview.nvim",
  lazy = false, -- 즉시 로딩
  dependencies = {
    "nvim-treesitter/nvim-treesitter", -- Treesitter 의존성
    "nvim-treesitter/nvim-treesitter-textobjects",
    "nvim-treesitter/nvim-treesitter-context",
  },
  config = function()
    require("markview").setup({
      -- 여기에 옵션 설정 (아래 참조)
    })
  end,
}
```

