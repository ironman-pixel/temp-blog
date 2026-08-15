---
date: 2025-09-09
tags:
  - kanban
  - project
---
`ctrl + s` 로 저장 시 자동으로 `ruff --fix` 가 실행되도록 했다.

`.vscode/settings.json`

```json
{
  // Python 인터프리터 경로 (가상환경)
  "python.defaultInterpreterPath": ".venv/Scripts/python.exe",

  // Python 포맷터 설정
  "python.formatting.provider": "none",

  // Python 파일에 대한 설정
  "[python]": {
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.fixAll.ruff": "explicit",
      "source.organizeImports.ruff": "explicit"
    }
  },

  // Ruff 설정
  "ruff.lint.enable": true,
  "ruff.format.enable": true,
  "ruff.showNotifications": "onError",

  // 기본 에디터 설정
  "editor.rulers": [80],
  "files.trimTrailingWhitespace": true,
  "files.insertFinalNewline": true,

  // Python 관련 기본 설정
  "python.linting.enabled": false, // ruff를 사용하므로 기본 linting 비활성화
  "python.analysis.autoImportCompletions": true
}

```