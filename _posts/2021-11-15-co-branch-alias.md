---
layout: post
title: "Lazy dev: Indexed Branch Switching 🌳"
tags:
    - git
---

Back in August, I [wrote about making an alias](https://zarah.dev/2021/08/10/magic-reflog.html) for finding the five most recent branches I have checked out by filtering out `git reflog` entries.

Here's how the output looks like for the alias:
```shell
➜  sdk_sandbox git:(feature/app-shortcuts) ✗ grefb  
1 - main
2 - feature/clickable-spans
3 - main
4 - task/update-gradle
5 - task/update-to-mdc
```

The alias has been really useful and I have been using it a lot, but the command to _actually_ switch branches is a bit difficult to type.

```shell
➜  sdk_sandbox git:(feature/app-shortcuts) ✗ gco @{-3}
Switched to branch 'main'
Your branch is up to date with 'origin/main'.
```

So today I added a new alias that will put in the command for me:

```shell
alias gcobr='() { gco @{-$1} ;}'
```

This means I can just enter the index of the branch I want to go back to without having to worry about the syntax :tada: :
```shell
➜  sdk_sandbox git:(feature/app-shortcuts) ✗ gcobr 3
Switched to branch 'main'
Your branch is up to date with 'origin/main'.
```

