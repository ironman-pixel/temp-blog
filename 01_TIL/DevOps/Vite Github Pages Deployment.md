---
date: 2025-06-16
tags:
  - kanban
  - project
  - "#githubactions"
  - "#vite"
---
# ❓ Information
* `username.github.io` project deployment
* using vite + react

---
# ❗ Relevant data
## 🎯 What Is The Objective
fixing console error from deployed github pages
## 📦 Information Resources
> [Vite official guide: static-deploy](https://ko.vite.dev/guide/static-deploy)


# 🔰 Content ->  

pushed a project to `startoverrox.github.io` repo. 
works on local. doesnt work from github page. 

**console error:**
`main.tsx:1 Failed to load module script: Expected a JavaScript-or-Wasm module script but the server responded with a MIME type of "application/octet-stream". Strict MIME type checking is enforced for module scripts per HTML spec.`

**trubleshoot:**
GitHub Pages doesn't compile TS. 
I need to manually bundle it (build it) and push the `dist/` folder.
Or, configure GitHub Actions to make CI/CD pipeline.

## 1️⃣ Intro 
* learning basics of vite build
* github actions CI/CD first try
## 2️⃣ Overview 
### How Vite handles static file (step by step)

#### 1. You used this import:
```
import GithubIcon from '@/assets/icons/GithubIcon.svg?react';
```
This is:
- A **module import**
- Handled by Vite
- Routed through the `vite-plugin-svgr` loader because of `?react`

#### 2. Vite sees this during build
It does **not** look for `/assets/icons/GithubIcon.svg` in your final HTML.
Instead, it:
- Loads `GithubIcon.svg` as a **React component**
- Transforms it to a `.js` module (e.g., an inline React component function)
- **Injects that code** into your final bundle in `dist/assets/`
So there’s **no actual SVG file copied** anymore — it's turned into React code.

#### 3. GitHub Actions builds your site
You have a GitHub Actions workflow that:
- Runs `vite build`
- Creates `dist/`
- Copies the transformed JS modules and bundles
- Publishes `dist/` to GitHub Pages
So **Github Pages is not serving SVGs directly — it’s serving JS** that **includes the inlined SVG-as-React-component**.
That’s why it works, even though the raw `.svg` file isn't directly served.

#### 4. You verified it works on GitHub Pages
Yes — because:
- The `import` made Vite bundle it    
- SVGR transformed it into React code    
- The final site **does not need** the raw SVG anymore — it has it compiled into JS

#### TL;DR
Your SVG becomes **React code (JS)** during build
That's why:
You **don’t see** the `.svg` file in `dist/` — it’s **compiled away**
You **do see** the icon on the live site — because it’s **rendered via JS
It works on GitHub Pages — because you’re serving a React app, not raw files**

### What is Github Pages 

#### GitHub Pages without GitHub Actions
GitHub Pages **just serves static files** from a specific location in your repo. No build, no CI/CD, just straight file hosting.

#### So When Is GitHub Actions Needed
Only when you need to:
- Build your project (e.g., TypeScript, SCSS, JSX → JS/CSS/HTML)
- Run tests or linters before deploy
- Deploy from a branch **other than the default
- Avoid committing build artifacts (`dist/`)

#### Where to put Github Actions configuration
under `.github/workflows/`

## 📃 Steps 
1. (static files should be under `src/assets`)
2. ( `dist` should be included in `.gitignore`)
3. (inside github repo settings, from pages, choose source as `GitHub Actions`. you may get to edit `static.yml`.  it will ask for you to `git push` the file.)
4. check if you got `.github/workflows/static.yml` in your project
5. add build steps inside workflow
