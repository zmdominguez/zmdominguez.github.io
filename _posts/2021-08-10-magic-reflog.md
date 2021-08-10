---
layout: post
title: "Harnessing the Power of Reflogs 🧙‍♀️"
tags:
    - git
---

A few weeks ago, I tweeted about a discovery that blew my mind:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">For today&#39;s edition of &quot;why didn&#39;t I look this up before&quot;, Git has a shorthand for going back to previously checked out locations. 🔀 Use `@{-N}`, where N-th branch/SHA. Ref: <a href="https://t.co/yHRbplXfxx">https://t.co/yHRbplXfxx</a> <br><br>Especially stoked about the shorthand `gco -` to go back to previous branch. 🧙‍♀️ <a href="https://t.co/4Sduv9puOZ">pic.twitter.com/4Sduv9puOZ</a></p>&mdash; Zarah Dominguez 🦉 (@zarahjutz) <a href="https://twitter.com/zarahjutz/status/1420606218981175302?ref_src=twsrc%5Etfw">July 29, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

In the tweet, I referenced the [git docs](https://git-scm.com/docs/git-checkout#Documentation/git-checkout.txt-ltbranchgt) which says:
> You can use the @{-N} syntax to refer to the N-th last branch/commit checked out using "git checkout" operation. You may also specify - which is synonymous to @{-1}.

That's cool and all, but what I realised is that most of the time, aside from the last checked out branch, I don't remember what the third to last or fifth to last branch was. Actually that's a lie. Sometimes I don't remember the last checked out branch too. :woman_facepalming:

All I remember is that I was in _that_ branch but I don't remember the name nor what number I should put in the braces. About 98% of the time, I ended up consulting the [`reflog`](https://git-scm.com/docs/git-reflog) to figure out this information.

The reflog enumerates anything that happens within git. It shows information about references, any commands that have been run, or where `HEAD` is pointing to, among others.
<center>
    <a href="https://imgur.com/aQ6YRPb"><img src="https://imgur.com/aQ6YRPb.png" title="Sample reflog output" /></a> <br />
    <small>A sample reflog output</small>
</center>

That's a _lot_ of information, but what we are looking for are lines similar to this one:
```zsh
1e8a561 (origin/main, main) HEAD@{10}: checkout: moving from main to task/lint-gradle
```

From here I can use the branch name and go back to that branch if I want to. Scanning all info spewed by reflog can be overwhelming; sometimes I'm just interested in finding out the branch name for checking things out and not in all the other details.

After some furious Googling and some trial and error, I ended up with this:
```zsh
git reflog | egrep -io "moving from ([^[:space:]]+)" | awk '{ print NR " - " $3 }' | head -n 5'
```

Let's break it down:
- `git reflog`: Without parameters like this one it, reflog will default to `show`
- `egrep -io "moving from ([^[:space:]]+)"`: Looks for the phrase "moving from " which is common to any entries for `checkout` commands (we can also just look for "checkout:" but I like living on the edge)
- `awk '{ print NR " - " $3 }'`: Prints out the line number (`NR`) and the third parameter in the line (the "from" branch name) separated by a dash
- `head -n 5`: limits the output to five lines

Running this command gives back:

<center>
    <a href="https://imgur.com/Yu5xa1Y"><img src="https://imgur.com/Yu5xa1Y.png" title="A list of last checked out branches" /></a> <br />
    <small>Convenience!</small>
</center>

The numbers on the left match the number I can use for checking out. For example, if I want to go back to `feature/package-rename`:
```zsh
gco @{-4}
```

Since I'm interested in using the numbers, I don't care if some branch names appear multiple times in the list. To remove duplicates, add this command after the `egrep`:
```zsh
awk ' !seen[$0]++'
```
:bangbang: Note that if you filter out duplicates the numbers on the left cannot be used for the `git checkout @{-N}` shorthand :bangbang:

---
Look, I'm an old woman and I like using the command line for git so please don't @ me with your fancy GUI suggestions. :older_woman:  

---

### References:
- [Git recent](https://gist.github.com/jordan-brough/48e2803c0ffa6dc2e0bd)  
- [How can I get a list of Git branches that I've recently checked out?](https://stackoverflow.com/questions/25095061/how-can-i-get-a-list-of-git-branches-that-ive-recently-checked-out)  
- [List your most recently-used branches using Git](http://ses4j.github.io/2020/04/01/git-alias-recent-branches/)
- [How to delete duplicate lines in a file without sorting it in Unix?](https://stackoverflow.com/questions/1444406/how-to-delete-duplicate-lines-in-a-file-without-sorting-it-in-unix/1444448#1444448)
