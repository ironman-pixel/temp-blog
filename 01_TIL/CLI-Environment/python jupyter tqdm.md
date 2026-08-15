---
date: 2025-06-26
tags:
  - cli
  - terminal
  - til
  - dev-tools
  - python
---
# ❓ Information
* to get tqdm work from jupyter:
  you will need to set a few things

---
# ❗ Relevant data
## 📦 Information Resources
[wikidocs: IPyWidgets: Jupyter 환경에서의 인터랙티브 위젯](https://wikidocs.net/229722)

# 🔰 Content ->  
## 1️⃣ 문제 상황
* ipython 파일에서 중첩 tqdm 이 잘 렌더되지 않았다.
## 2️⃣ 해결
* 다음 설정이 더 필요하다.
```
$ pip install ipywidgets
$ jupyter nbextension enable --py widgetsnbextension --sys-prefix
```
- 그리고 파일을 껐다 켜면 될것이다.