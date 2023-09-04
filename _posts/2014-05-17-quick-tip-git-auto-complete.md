---
layout: post
title: 'Quick Tip: git Auto-complete'
date: '2014-05-17T04:22:00.000+10:00'
author: Zarah Dominguez
tags:
  - git
modified_time: '2014-05-17T04:22:04.941+10:00'
blogger_id: tag:blogger.com,1999:blog-8588450866181483028.post-2292886583832184790
blogger_orig_url: http://www.zdominguez.com/2014/05/quick-tip-git-auto-complete.html
---

When I started using git, it peeved me that there is no auto-complete. More so when you have to manually do a `git add` manually.

Thank heavens I found this little gem. Making auto-complete work for git:

In terminal:
```xml
curl https://raw.github.com/git/git/master/contrib/completion/git-completion.bash -o ~/.git-completion.bash
```

And then you'd need to "activate" it in your `.bash_profile`:
```shell
if [ -f ~/.git-completion.bash ]; then
  . ~/.git-completion.bash
fi
```