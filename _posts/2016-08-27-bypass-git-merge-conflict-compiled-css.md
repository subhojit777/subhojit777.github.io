---
title: Bypass Git merge conflicts in compiled CSS
description: Bypass Git merge conflicts in compiled CSS using Git attributes
date: '2016-08-27'
author: Subhojit Paul
#tags:
# - git
# - gitattributes
---

After reading this blog you will learn, how you can automate the resolving of merge conflicts for certain files during `git rebase`.

## The Problem
I was working on a project where we were using Sass for styling. We were maintaining multiple branches for multiple features. I often had to rebase branches. And obviously, there were merge conflicts. I use [Meld](http://meldmerge.org) to resolve conflicts. There were conflicts in the compiled CSS as well. For every merge conflicts Meld, opens up and asks you to resolve them. Similarly, it used to open up for the compiled CSS as well. You may think :confused: - well.. ok.. you can just close the Meld without doing anything and recompile the CSS. Well, that is okay unless you don't want to automate that. My **problem** was - the CSS was also minified, and since it is minified, Meld used to take ~10 seconds, to open  up the file in three panes. And there's more - we had the compiled CSS `styles.css` *and* the map file as well. So, ~10 * 2 = ~20 seconds, I had to  wait for ~20 seconds just to close the Meld and recompile the CSS.

Earlier in the project, there were fewer conflicts while rebasing, so I usually waited for that 20 seconds. But things became more painful when we used to rebase two branches with longer history. It used to take me around half hour to completely rebase one branch to another, while the "actual" conflict fix took me hardly 2 minutes.

So, I decided - why should I waste my time, let the Computer do this thing for me.

## The Solution
People in IRC `#git` channel helped me out with the problem. They said [**`.gitattributes`**](https://git-scm.com/docs/gitattributes) is the solution. I googled and found many blogs and StackOverflow links. I liked this [StackOverflow question](http://stackoverflow.com/questions/928646/how-do-i-tell-git-to-always-select-my-local-version-for-conflicted-merges-on-a-s). The answer states the solution in detail. It tells how you can make use of drivers to resolve merge conflicts. We will be creating a custom merge driver.

**The definition of the driver:**

Inside `.git/config` file:

```
[merge "ours"]
	name = Keep ours merge
	driver = true
```

Or you can just execute the following commands:

```sh
git config merge.ours.name "Keep ours merge"
git config merge.ours.driver true
```

Create `.gitattributes` file with the following content:

```
*.css merge=ours
*.css.map merge=ours
```

If you already have the `.gitattributes` file, you just append the above content in the file.

**Explanation:**

From this [StackOverflow answer](http://stackoverflow.com/a/3052118/1233922):

> A rebase switches `ours` (current branch before rebase starts) and `theirs` (the upstream branch on top you want to rebase).

Since, I am using GUI mergetool Meld, therefore, in a GUI mergetool context:

> - local references the partially rebased commits: "ours" (the upstream branch)
> - remote refers to the incoming changes: "theirs" - the current branch before the rebase

So what we are doing is, we are keeping the local version (`ours`) of the conflicting CSS and map files. That's it. You can do more complex stuff, you just have mention that in the `merge.*.driver`.

The next time there is a merge conflict in the CSS or in the map file, it won't open Meld, it will keep the local version of the file. After the successful rebase, if you still don't trust the merged CSS, you can recompile it again and [`squash`](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History#Squashing-Commits) the new commit with any desired commit.
