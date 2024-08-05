# Git cheat sheet and Note for Looking up

**use github tokens for push remote repository**
In this way no ssh settings needed.
```
# If remote origin url is already set
git remote set-url origin https://myusername:mygithubtoken@github.com/myusername/repository.git

# else then initialize directly with the github token
git remote add origin https://myusername:mygithubtoken@github.com/myusername/repository.git

```


## Git merge
1. Fast-forward branch merge
```
git checkout master
git merge hotfix 
```
```markdown
**Note**: `hotfix` is a branch branch direct right after the pointer of the master branch, so there is no confilicts that need to be solved.
```

2. delete branch because we need it no more.
```
git branch -d hotfix
```
3. Basic 3-way merge

 In this case, your development history has diverged from some older point. Because the commit on the branch you’re on isn’t a direct ancestor of the branch you’re merging in, Git has to do some work. In this case, Git does a simple three-way merge, using the two snapshots pointed to by the branch tips and the common ancestor of the two.

```
git checkout master
git merge iss53
```
Instead of just moving the branch pointer forward, Git creates a new snapshot that results from this three-way merge and automatically creates a new commit that points to it. This is referred to as a merge commit, and is special in that it has more than one parent.

4. Basic Merge Conflicts
```
git status
```
use this to check

Anything that has merge conflicts and hasn’t been resolved is listed as unmerged. Git adds standard conflict-resolution markers to the files that have conflicts, so you can open them manually and resolve those conflicts.

This resolution has a little of each section, and the `<<<<<<<`, `=======`, and `>>>>>>>` lines have been completely removed. After you’ve resolved each of these sections in each conflicted file, run git add on each file to mark it as resolved. Staging the file marks it as resolved in Git.



If you want to use a graphical tool to resolve these issues, you can run git mergetool, which fires up an appropriate visual merge tool and walks you through the conflicts:
```
git mergetool
```

After you exit the merge tool, Git asks you if the merge was successful. If you tell the script that it was, it stages the file to mark it as resolved for you. You can run git status again to verify that all conflicts have been resolved:
```
$ git status
On branch master
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:

    modified:   index.html
```

If you’re happy with that, and you verify that everything that had conflicts has been staged, you can type `git commit` to finalize the merge commit.


## Rebasing

However, there is another way to integrate brunches: you can take the patch of the change that was introduced in C4 and reapply it on top of C3 (C3 and C4 are commit on separate branches). In Git, this is called rebasing. With the rebase command, you can take all the changes that were committed on one branch and replay them on a different branch.

1. Basic Rebase
```
$ git checkout experiment
$ git rebase master # The master branch is that you want to keep
First, rewinding head to replay your work on top of it...
Applying: added staged command
```
This operation works by going to the common ancestor of the two branches (the one you’re on and the one you’re rebasing onto), getting the diff introduced by each commit of the branch you’re on, saving those diffs to temporary files, resetting the current branch to the same commit as the branch you are rebasing onto, and finally applying each change in turn.



At this point, you can go back to the master branch and do a fast-forward merge.
```
$ git checkout master
$ git merge experiment
```

There is no difference in the end product of the integration (Compared with a 3-way merging), but rebasing makes for a cleaner history. If you examine the log of a rebased branch, it looks like a linear history: it appears that all the work happened in series, even when it originally happened in parallel.

**Important Note**:

Often, you’ll do this to make sure your commits apply cleanly on a remote branch — perhaps in a project to which you’re trying to contribute but that you don’t maintain. In this case, you’d do your work in a branch and then rebase your work onto origin/master when you were ready to submit your patches to the main project. That way, the maintainer doesn’t have to do any integration work — just a fast-forward or a clean apply.