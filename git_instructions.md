---
title: git_instructions
date: 2024-07-23
type: general
tags: " #code/shell #github "
description: Code for backing up this folder to github.
---



```zsh
cd /Users/brendancasey/Dropbox/obsidian_notes/3_resources/obsidian/templates
git init 
git remote add origin https://github.com/bgcasey/obsidian_templates.git
git add . 
git commit -m "Initial commit" 
git push -u origin main
git pull origin main
```