---
date: 2025-09-05
tags:
  - package-manager
  - til
  - dev-tools
---
# ❓ Information
* precommit으로 ruff check 하는법

---

# 🔰 Content ->  

`.pre-commit-config.yml`

```yml
# See https://pre-commit.com for more information
# See https://pre-commit.com/hooks.html for more hooks
repos:
    - repo: https://github.com/pre-commit/pre-commit-hooks
      rev: v3.2.0
      hooks:
          - id: trailing-whitespace
          - id: end-of-file-fixer
          - id: check-yaml
          - id: check-added-large-files

    - repo: https://github.com/charliermarsh/ruff-pre-commit
      rev: v0.12.10
      hooks:
          - id: ruff-check
            args: [--fix]
          - id: ruff-format
```