---
title: How to squash and rebase in git
image:
imageMeta:
  attribution:
  attributionLink:
featured: true
authors: 
  - jen-weber
date: Mon Aug 23 2021 10:45:45 GMT-0400 (Eastern Daylight Time)
tags:
  - new
---

Did you ever need to do `git rebase --continue` over and over again just to get your own branch up to date? Chances are, you are on the sad path, and 99% of the time, there's a better strategy!

Rebasing is commonly a nightmare for newer devs or even experienced devs who don't use this workflow often, but it doesn't have to be! Here's how I walk people through their first squash and rebase. There are a ton of excellent articles and videos on this, but everyone does it a little differently. I like [this video by The Modern Coder](https://youtu.be/V5KrD7CmO4o) that shows part of the process in action. After a few times doing this process, it will feel tedious but straightforward. No nightmares here.

## Set your default text editor in git

You only need to do this once. Skip this if you already configured this on your computer. If you have not set your default editor for git, then some of the steps below would open in vim, which is difficult to use if you have zero practice with it.

I use VSCode, and so I used the one-line command in [this article by Carl Saunders](https://dev.to/deadlybyte/make-vs-code-your-default-git-editor-j6d).

## Steps to squash and rebase

Create a backup branch and push it up to GitHub/Gitlab/etc. This backup branch will have all your work nice and safe in case you need to start over.

```
git checkout my-branch-name
git branch backup-my-branch-name
git push origin backup-my-branch-name
```

Look up the commit id of the most recent commit that isn't yours, and copy it:

```
git log
```

Start the rebase, replacing the placeholder text below with the commit id you copied:

```
git rebase -i the-commit-id-you-looked-up-goes-here
```

Mark the commits you want to squash by putting "s" instead of "pick" in front of all the commits except the first one. The one at the top should still say "pick."
If there are a ton of conflicts, sometimes it's best to do some pair programming with the other dev(s) who made changes to these files. They can help you decide what to do.

Save and close, and wait for a new text editor prompt to pop up.

Enter your commit message.

Save and close.

Squashing should be complete. Check for any problems:

```
git status
```

Next, I'm getting ready to rebase. All my collaborators merge their work into a branch called `main`, so that's what I'm using in this example. I pull that branch first, just to make sure my local copy is up to date:

```
git checkout main
git pull origin main
```

Time to actually rebase. Check out your branch, and rebase!

```
git checkout my-branch-name
git rebase main
```

Fix any git conflicts just like you would if you were merging something.

Stage the files once you have resolved conflicts:

```
git add some-file.js another-file.html example.txt
```

Then, finish the rebase:

```
git rebase --contine
```

You overwrote git history, so you will need to force push your work up. Make sure you are on your own branch before running this! It will overwrite other people's work if you are on the wrong branch.

```
git push --force-with-lease origin my-branch-name
```

## If something goes wrong

No worries, you have a backup branch! Ask a colleague for help. This is a learning opportuinity. If you are all alone and you don't have anything else at all to work on, you could try the following to put things back to how they were. Be careful, don't lose your work!

Abort the rebase in progress:

```
git rebase --abort
```

Rename the messed up branch:

```
git branch -m messed-up-my-branch-name
```

Check out your backup branch. You may need to discard changes/new files with `git restore` before this command can be run.

```
git checkout backup-my-branch-name
```

Copy your backup, and give the copy the name you were using before:

```
git checkout -b my-branch-name
```

Now you are back to where you started, and can try again.

## Why squash before rebase?

Whenever you rebase, git compares every commit to the rebase target to see if there are conflicts. It makes you resolve them before you move on. Sometimes, this is useful - maybe you want to step through. But most of the time, fixing the conflict causes even more conflicts.

Instead, we want to squash all our work into just one commit. Then we resolve any conflicts _just once_, and then we are off to the races.

## When should I _not_ squash before rebase?

Every company/project has different norms about how they like to manage their project git history. If they tell you not to squash or rebase, then don't. Ask to pair program with someone a few times to get a sense of a nice workflow for their process.
