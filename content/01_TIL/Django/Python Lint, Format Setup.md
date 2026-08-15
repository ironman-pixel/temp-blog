---
date: 2025-06-28
tags:
  - kanban
  - project
---
# from VSC
>[orchestra: How to Install Ruff for Python on VS Code](https://www.getorchestra.io/guides/how-to-install-ruff-for-python-on-vs-code)

## Auto Linting
1. Install the Ruff Extension from vsc marketplace
2. add these to `settings.json` 
```
"python.linting.enabled": true,
"python.linting.ruffEnabled": true,
"python.linting.lintOnSave": true,
"python.linting.enabledWithoutWorkspace": true
```

## Auto Formatting
1. run `pip install black` from your env
2. add these to `settings.json`
```
"python.formatting.provider": "black",
"python.formatting.blackArgs": [
    "--line-length",
    "88"
],
"editor.formatOnSave": true
```


## to Format the Entire Project
1. run `black {dir}`
