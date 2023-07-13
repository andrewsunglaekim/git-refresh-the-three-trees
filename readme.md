# Git Refresh! The three trees of git
## Learning Objectives

- use `git show` to inspect a commit
- identify the unidriectional relationship of commits
- leverage git revisions to target commits
- describe the three trees of git
- use `git reset` to reset git history
- use `git reflog` to access reference logs to undo terrible things
- use `git revert` to "revert" a commit
- identify `git merge` creates a merge commit
- use `git rebase` to rewrite history onto a new branch's tip
- identify pros and cons of rebase vs merge
- prioritizing git workflows on teams
- other useful git things given time

## Framing

Today we'll be diving deeper into some of the git commands we use daily. With a solid understanding of these git concepts, we can start to leverage really powerful tools at git's disposal.

We'll try to dispel some of the fear caused by git commands by understanding exactly what happens during those commands.

It'll be impossible to cover the entirety of the git ecosystem, but we'll cover some useful tools for git users. As well as establish a foundation of knowledge for the taxonomy we use in git.

## project setup 

if you'd like to follow along:
```
$ mkdir temp
$ cd temp
$ git init
$ touch hello.txt
```

then run this bash script:

```
for ((i=1; i<=10; i++))
do
    # Write a line to the text file
    echo "This is commit number $i" >> hello.txt
    
    # Commit the changes using git
    git add commits.txt
    git commit -m "Commit number $i"
done
```

## Git show

`git show` shows us commit information and the git diff from the commit specified as the argument against it's direct parent(s)

It can be specified with sha or revision or without any argument

```
$ git show
$ git show c11c027f123        #random git sha
$ git show HEAD~3
```

Without an argument git show targets the `HEAD`. `HEAD` being the tip of the current branch the user is on.

## Level setting on commits

Commits are changes to a repository.  More specifically they point to the exact moment when the change occurs, what changes occurred, and who made the changes.

Not only that, commits are aware of where they come from. IE. which commit(s) is/are my parent(s).

> Note the (s) as commits can have multiple parents from a merge.

Git leverages this [linked list](https://en.wikipedia.org/wiki/Linked_list) data structure to define git logs and have histories of commits even when commits themselves are only aware of their direct parents.

## Git revisions and relative references

In Git, a revision refers to a specific state of a repository at a given point in time. 

#### commit hash

Each commit in Git has a unique identifier called a commit hash or commit ID. It is a long string of characters generated using a cryptographic hash function. You can use the commit hash to refer to a specific revision.

#### branches

Branches in Git are lightweight pointers to specific commits. They allow you to easily refer to a specific revision. For example, `master` or `feature/branch-name` can be used to identify a revision.

#### tags

Tags are named references to specific commits, often used to mark significant points in the project's history, such as releases or milestones. Tags provide a convenient way to refer to a specific revision using a meaningful name.

#### Relative references

Git provides various relative references to navigate the commit history.

- `~` or `^`: Specifies parent commits.

there are some other ones but you'll mostly just see `~` unless your working with the reference log or time in which case you may use `@{}`

## The three trees of git

When we think about git, we can boil down a good amount of it's commands with how they influence the three "trees" of git:

#### The working directory

This tree is in sync with the local filesystem and is representative of the immediate changes made to content in files and directories.

#### The staging index

This tree is tracking Working Directory changes, that have been promoted with git add, to be stored in the next commit.

#### The commit history

The repository, also known as the commit history, is where Git stores the complete history of your project. It includes all the commits, branches, tags, and other references. 

## Git reset

The following command could have disastrous effects on our HEAD's history, the branch HEAD is on. Be very careful when resetting. In most cases, it rewrites history, when it's not it's resetting your working directory. Either way we stand to lose some change we've made in one of our trees of git. 

`git reset` is a versatile tool for undoing changes. It is easiest to think about reset and its different flags in how it changes and modifies the three trees of git.

We will be leveraging git reset and differentiating between 3 different modes (`--soft`, `--mixed`, `--hard`). For each reset the 3rd tree(commit history) will always be reset during a git reset.

#### Starting with a soft reset

```
$ git reset --soft HEAD~1
```

When a soft reset is done, the third three(commit history) is set to `HEAD~1` (1 revision behind the head before the reset). The commit itself is removed but the diff from the commit now lives in the second tree(staging area). Nothing changes in the first tree(working directory) for a soft reset.

We might want to do a soft reset when we like the commits we've made, but we want to tack on a bit more to it

#### The mixed reset

```
$ git reset --mixed HEAD~1
# functionally equivalent
$ git reset HEAD~1
```

The mixed reset is the default mode for `git reset`

Just like in `--soft` the current HEAD will now reference what was formerly `HEAD~1`, but in addition the diff isn't staged. So the commit history is reset, the staging area is reset, but the working directory still exists as the original `HEAD`.

Do a mixed reset when we dont like the state of our current commit, and we dont want to eliminate the work from our working directory that lives in the current commit.

#### The hard reset

This is by far the most dangerous type of reset. We would only want to use a hard reset if we wanted to time travel to exactly the state of a commit.

```
$ git reset --hard HEAD~1
```

All three tree in this case would "reset" to the state of `HEAD~1` and that would be come the new `HEAD` This means we would lose all traces of the original `HEAD` commit and essentially not be able to access them ever again...or not!? 

#### git reflog

With all these dangerous things we can do in git, surely there must be some way for us to backtrack on disastrous commands. There is! Enter the reference log.

The default expiration time for reflog entries is 90 days. So it's good to fix any immediate mistakes with the reference log, but certainly isn't going to be a possible solution if we've waited too long.