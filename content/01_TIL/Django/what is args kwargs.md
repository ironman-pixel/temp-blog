---
date: 2025-06-28
tags:
  - kanban
  - project
---
## 역할

- `*args`: 위치 인자 (tuple 로 받음)
- `**kwargs`: 키워드 인자 (dict 로 받음)

## 왜 쓰냐?

- View에서 상속받는 DRF 의 class들이 내부적으로 view 함수에 여러 인자를 넘길 수 있기 때문
- 확정성 확보 -> 상속 관계에서 부모 class가 전달하는 인자를 유실 없이 받기 위해 필수
