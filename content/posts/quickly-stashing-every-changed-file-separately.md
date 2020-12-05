+++
date = 2020-12-05T05:27:00Z
draft = true
title = "Quickly stashing every changed file separately"

+++
In [my last foray](https://usrme.xyz/posts/quickly-generating-patch-files-for-every-changed-file/) into not using Git entirely correctly I dabbled with generating patch files from changed files as a way to quickly store the state within a repository and being able to excise that state piece by piece elsewhere.

Looking back now it was a bit ham-handed and in all honesty done more for the sake of breaking my blog's dry spell. But this time! Oh, this time there even may be _some_ value in what I'm about bring forth.

***

[Git-stash](https://www.git-scm.com/docs/git-stash) is the _de facto_ cool way to store state within a repository without branching or committing, and ever the _ingénieur_ I have devised of a way to automatically stash every file separately. This comes with an added bonus of being able to tell which stash relates to which file.

Before getting to the meat of the matter I should mention that I am aware of the `-p|--patch` option to `git stash [push]`, which allows you to interactively select hunks, but I do not see this as solving the same "problem" of quickly dumping the state, moving on, and being able to restore one's mental context when coming back without too much work.

You ready? Don't blink:

`git status -s | cut -d " " -f 3 | xargs -I {} git stash push {} -m "{}"`

Here's what everything does:

* show the working tree status in the [short-format](https://git-scm.com/docs/git-status#_short_format) with `git status -s`:

```
 M Dockerfile
 M Dockerfile.az
```

* cut the third field using space as the separator with  `cut -d " " -f 3` leaving just the filename:

```
Dockerfile
Dockerfile.az
```