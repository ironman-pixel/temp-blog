---
date: 2025-06-20
tags:
  - cli
  - terminal
  - til
  - dev-tools
  - windows
  - python
---
# ❓ Information
```
$ conda env list 
bash: C:\ProgramData\Anaconda3\Scripts: Is a directory
```
fix this.

---

# 🔰 Content ->  

## 1️⃣ there is ~/.bashrc , ~/.bash_profile in Windows
* I didn't search for details on that
## 2️⃣ how to fix
* I added `unset -f conda` inside `~/.bash_profile` and it worked.
* not sure why
## 3️⃣ couldn't fix
- adding `unset -f conda` made `conda env list` work.
- but `conda activate aaa` won't work.
## 4️⃣ I'm done with this. I quit
