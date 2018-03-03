---
layout: post
title: "That Thing About Commit Messages"
tags:
- git
---
Last week, I was giving feedback to someone about improving the commit messages they write. I was very taken aback by their response -- "it does not matter what the commit messages are". Now this is confusing for me, mainly because I know from _years_ of experience that having relevant commit messages is important.

Their argument was that change descriptions go into the pull request anyway, so they -- not just him, his **_whole team_** -- do not really care if the commit messages are vague.

I whole-heartedly disagree with this assessment. To illustrate my point, say we are looking for a commit that broke a feature.  What I usually do in this situation is to look at the whole commit history to look for anything obvious. It is not very easy when the history looks like this: 
<p style="text-align: center"><a href="{{ site.baseurl }}/assets/2017_info.png"><img src="{{ site.baseurl }}/assets/git_commits.png"></a></p>

😩😩😩😩 We can either go through each of the changed files in each of these commits. This might be too tedious, and the commit messages does not give us any clear idea if that might be the case.

I know this is a super-simplified example, but imagine combing through the history of a repo with five or eight or 20 or 200 contributors.

Going by their reasoning that all the details are in the pull request, I should go into Github and look at all the PRs from this time frame. How many would that be? And would you have to check out each of those branches?

`git bisect` would be helpful at this point. But if the commit messages were a bit more helpful, wouldn't we have at least have an educated guess of which commit we are looking for?

By all means, feel free to write a dissertation on your pull request descriptions. Just bear in mind that if you move from one service to another, you would need to move all your pull requests as well. Otherwise you lose all context of your project's history.

Logan Johnson [wrote so eloquently](https://medium.com/square-corner-blog/how-square-writes-commit-messages-8e92fcbf77c9) about Square and how they write commit messages. If you are in the "Well that is overkill camp", then maybe we can compromise by following this tip from [@dzaporozhets](https://twitter.com/dzaporozhets/status/870268536404533249): 
<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Best tip about writing a good commit message: <a href="https://t.co/7K4y7zUdy6">pic.twitter.com/7K4y7zUdy6</a></p>&mdash; Dmitriy Zaporozhets (@dzaporozhets) <a href="https://twitter.com/dzaporozhets/status/870268536404533249?ref_src=twsrc%5Etfw">June 1, 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
