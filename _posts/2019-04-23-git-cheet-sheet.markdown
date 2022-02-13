---
title: Git CheetSheet
date: 2019-04-23
categories: Git
---

### Basic

Automatically stages everything and commit it:

```shell
git commit -a
```

Only staged files are to committed, others remain unstaged:

```shell
git commit -m 'message'
```

See the changes you have made in the working directory:

```shell
git diff
```

See the changes of the staged files:

```shell
git diff --staged
```

### Branch

* Create a new branch:

```shell
git branch <branchname>
```

* Switch to that branch:

```shell
git checkout <branchname>
```

* Merge the branch:

```shell
git merge <branchname>
```

* Delete branch, use `-D` to delete branch by ignoring changes:

```shell
git delete -d <branchname>
```

* Create a Branch to track remote branch:

```shell
git checkout -b serverfix origin/serverfix
```

* Use remote branch to replace local branch:

```shell
git fetch origin
git reset --hard origin/xxx
```

* Simply delete remote tracking branch(this will not delete the branch on the remote repo):

```shell
git branch -d -r origin/<remote branch name>
```

* Delete a branch remotely:

```shell
# delete branch "foo" in origin remote
git push origin --delete foo
```

### Revert

* Revert changes to working directory:

```shell
git checkout .
```

* Unstage the changes to the staging area:

```shell
git reset
```

* Remove untracked files:

```shell
git clean -f
```

* Remove untracked directories:

```shell
git clean -d
```

### Commits

* Change commit message before pushing to remote branch:

```shell
git commit --amend
```

* Add commits without commit record:

```shell
git commit --amend --no-edit
```

* Squash last X commits together:

```shell
git rebase -i <after-this-commit>
```

Then replace `pick` on the second and subsequent commits with `squash` and chanage the commit message. `after-this-commit` means the parent of the oldest commit you want to squash.

* Undo the last commit:

```shell
# any changes you made will be removed
git reset --hard HEAD~1
# or
git reset --hard <commitid>
```

```shell
# any changes you made will be kept in workspace
git reset --soft HEAD~1
# or
git reset --soft <commitid>
```

* Options
  * `--soft `: Staged area and working directory will not be changed;
  * `--mixed`: Default. Staged ahea will be changed but working directory will not;
  * `--hard`: Any changes to tracked files in the working tree since <commitid> are discarded;

* Undo the last commit you have pushed to remote repo:

```shell
# create new commit to revert last commit and push to the remote
git revert HEAD~1..HEAD
# or
git revert <commitid>
git push
```

* Delete local branch which does not exist in remote:

```shell
git remote prune origin
```

### Submodule

* Update all submodules:

```shell
git submodule foreach git pull
```

* Init and update submodules:

```shell
git clone --recursive /path/to/repos/foo.git
```

Equels to:

```shell
git clone /path/to/repos/foo.git
git submodule init
git submodule update
```



-End-
