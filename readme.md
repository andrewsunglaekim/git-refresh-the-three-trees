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
- prioritizing communication and git workflows on teams
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
    git add hello.txt
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

## Git Bisect

There's been lots of times where we implement a feature, we think it's solid and nothing can go wrong with it. Then we continue on the project, 4 or 5 more features later, the initial feature we implemented breaks. We have no idea which feature let alone a commit that broke the code. It would be really difficult to pin point the exact commit that breaks the code. Enter git bisect.

Through git logs we can see that commit histories are linear and "sorted" in terms of sequence of changes.

Git leverages binary search and user input to help identify bad commits through git bisect. Let's see how it works.

In order to "bisect" our code. We start with a simple command:

```
$ git bisect start
```

This will enter our git environment into "BISECTING" mode. If at any point we want to leave, run `$ git bisect reset`.

The next thing that bisect expects is 2 commit sha's. One sha where we know our code base is "correct" and another sha where we know our code base has a bug. Example:

```
$ git bisect good bc08081
$ git bisect bad b6a0692
```

> The arguments to the following commands are git sha's of a commit history. If our bad commit is at the HEAD, then we can run $ git bisect bad without an argument.

It'll start at the midway point. It will checkout to the commit that lies as close to the middle between the good and bad commits. It then expects a response between:

- `$ git bisect good`
- `$ git bisect bad`

If we inspected our code and it's bug free, we would use the `good` command.

If instead our code had a bug, we would use the `bad` command

Depending on the choice we make, git will move the HEAD to the next mid point commit.

If we choose `bad`, it will move HEAD to the midpoint commit in the first half

If we choose `good`, it will move HEAD to the midpoint commit in the second half.

Git repeats this until it identifies the first commit where the code went wrong. Unless we never tell it `bad`.

This is yet another reason why small purposeful semantic commits can be really helpful for code maintainability. If we're writing code that needs to compile, this is a great reason to make sure to only commit code that compiles.

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

The default expiration time for reflog entries is 90 days. So it's good to fix any immediate mistakes with the reference log, but certainly isn't going to be a possible solution if we've waited too long. It also is local to a developers machine.

References, the namesake of reflog more commonly known as branches, point to a commit. Quite literally, there is a file with the branch's name in the .git directory that has a commit sha of the the commit it is pointing to.

Git tracks these branches in a place that isn't ... the branches. It's the `reflog`. We can think of the `reflog` as a chronological history of everything we've done in our local repo. Reference logs record when the tips of branches and other references were updated in the local repository. Reflogs are useful in various Git commands, to specify the value of a reference for a commit that isn't able to be accessed through git log.

The most generic reflog is simply:

```
$ git reflog
```

What we'll see is the default reference log for the `HEAD`. Any change `HEAD` makes, `reflog` will list it out. Each line contains:

- abbreviated commit sha
- a git revision
- action type(eg. commit, reset, checkout, etc.)
- commit message or description

## Git revert

Another way to "change history" is by not changing a commit but instead adding a new commit using `git revert`. `git revert` takes a specified commit and rolls back the changes made from that commit. Then has us stage changes to continue the revert. git revert simply creates a new commit that is the opposite of an existing commit.:

```
$ git revert <some-git-sha>

# might look something like this:
$ git revert 67f68c
```

You'll likely have some conflicts that you'll have to parse through, stage them and then continue with:

```
$ git add <files to be staged>
$ git revert --continue
```

> don't commit the changes!

Eventually once all the conflicts are complete your default text editor will load to provide one or more commit messages.

## Git merge

When working with teams on a project, there will often be times where we need to pull changes from a remote repository. We have two main ways to integrate changes from the upstream. 

One is `git merge`.

```
$ git merge feature-branch
```

This command would merge `feature-branch` into `HEAD` resulting in a net new commit.

If each of the branches being merge change the same lines in the same file, this will result in a merge conflict. Resolve the conflict then stage and commit the changes to complete the merge

## Git rebase

Similar to when we merge, `git rebase` is another way we can grab changes from the upstream. Merge is always a forward moving change record, an additional commit. Alternatively, rebase "rewrites" history. `git rebase` reapplies commits on top of another base tip.

`git rebase` finds the common ancestor between the branch that we want to rebase and the branch we're rebasing on to. It then "removes" the commits of the rebasing branch until that common ancestor. Then it moves the branch's(the one that's rebasing) tip to the tip of the branch we are rebasing on to. Then it reapplies the same commits that were removed on that new tip.

> The reapplied commits are actually brand new commits with different pointers (git sha's).

To rebase, simply run:

```
$ git rebase <branch-rebasing-onto> <branch-being-rebased>

# OR if you are on the branch that you want to rebase on a new tip
$ git rebase <branch-rebasing-onto>
```

If it goes smoothly, we won't even have to do anything. All our changes will be intact on the new base and we can continue to work. If it doesn't go smoothly ...

#### Merge conflicts during a rebase

Just like with merges, git's rebase doesn't know what to do if the two branches edit the same line in the same file. It will create a merge conflict in a rebase as well.

We'll fix the conflict like we normally would in a merge.

Stage the changes of that merge,

but DO NOT commit. Instead:

```
$ git rebase --continue
```

If at any point we want to opt out of the rebase entirely, we can always run:

```
$ git rebase --abort
```

We'll have to do this for potentially every commit that is being replayed.

Throughout this whole process, we have never made a commit documenting that we were integrating changes from the upstream. It is as if we had written off of this new base the whole time. Rebasing reduces the noise of merge commits in your git history.

As nice as rebasing is, there are some serious dangers to it. Remember, we are rewriting history. Any time we rewrite history, we should be cautious.

The golden rule of rebasing is:

<strong>Don't rebase shared or public branches</strong>

## Merge vs Rebase

So, which is better? Depends on who we ask. There are many differing opinions on this. One perspective is that the repository's commit history is a record of what actually happened. That is to say, if we are grabbing changes from upstream(pulling), version control should know about this merge happening and document it always.

Another perspective is that we want to catalog code changes not meta data about how we are versioning.

If we find ourselves asking if we should be rebasing or merging, the answer lies in understanding how rebasing and merging effect git history and then determining a course of action.

In general the way to get the best of both worlds is to rebase local changes we’ve made but haven’t shared yet before we push them in order to clean up history. Then merge in rebased feature branches into our master branch or whichever branch is our "clean" one.

## Communication

One of the most important tools in git team workflows is not any one command or git interface. It is communication. There aren't any tools or commands that can reduce the amount of merge conflicts. The only thing that can help on that end is good team communication.

If we can have conversations with our team members about which features are being implemented or refactored, we can reduce the amount and complexity of merge conflicts.

