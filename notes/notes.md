# Notes

- [Notes](#notes)
  - [Intro](#intro)
    - [Key facts about Git](#key-facts-about-git)
  - [The first repo](#the-first-repo)
  - [Git internal representations](#git-internal-representations)
  - [Git config](#git-config)
  - [Git Branching](#git-branching)
    - [Merge](#merge)
    - [Rebase](#rebase)
    - [HEAD](#head)
      - [Reflog](#reflog)
    - [Cherry Pick](#cherry-pick)


## Intro

- `man`
- `man git-<op>`for the friendly manual

- repo
- `commit`: a point in time representing the project **in its entirey**, not just the differentials.
- 'staging area' or index: where you prepare the changes you wish to commit. When you commit, anything in the staging area will be commited. Unstaged items won't be committed.
- 'squash': to take several commits and turn it into one commit. Technically would be taking N commits and turning it into N-1 to 1 commit, but typically its N commits to 1 commit.
- 'work tree', 'working tree' or 'main working tree': your git repo. this is the set of files that represent your project. Your working tree is setup by `git init` or `git clone`. 
- untracked → staged → (commit) → tracked

### Key facts about Git

- git is an acyclic graph, meaning the following cannot exist
- In git, each commit is a node in the graph, and each pointer is the child to parent relationship. Meaning that you can have more than one parent, but you cannot have a cycle in the graph.
- If you delete *untracked* files, they are lost forever. Remember: commit early, commit often, you can always change history to make it one commit (squashing).

Cyclic vs acyclic graphs:

![Cyclic vs acyclic graphs](img/acyclic.png)

Git graph
![git graph](img/gitgraph.png)

- All git config keys are in the following shape: `<section>.<key>`.
- `--global` flag will ensure you set this key value for all future uses of git and repos
- `user.name` `user.email` are the key's used in creating a commit tied to you
- To add a key value, execute `git config --add --global <key> "<value>"`
- You can view any value of git config by executing `git config --get <key>`


## The first repo

(...)

- `git init`
- `git add <path-to-file | pattern>` will add zero or more files to *the index* (staging area)
- `git commit -m '<message>'` will commit what changes are present in *the index*.
- `git status` will describe the state of your git repo which will include tracked, staged and untracked changes.

## Git internal representations

- `git log` has many options to make viewing of history a pleasant experience.
  - `--no-decorate`, `--decorate[=short|full|auto|no]`. Print out the ref names of any commits that are shown. If short is specified, the ref name prefixes refs/heads/, refs/tags/ and refs/remotes/ will not be printed. If full is specified, the full ref name (including prefix) will be printed. If auto is specified, then if the output is going to a terminal, the ref names are shown as if short were given, otherwise no ref names are shown. The option --decorate is short-hand for --decorate=short. Default to configuration value of log.decorate if configured, otherwise, auto.
  - `--graph`. Draw a text-based graphical representation of the commit history on  the left hand side of the output.

`--decorate` is not really that useful, but it is when you export the log to a file. `git log --graph > out` will not print the `(HEAD -> main)` part of the log. It will when the command is `git log --graph --decorate > out`. 
![git log --graph --decorate > out](img/log-decorate.png)

The are ways to inspect files within the git's data store. `git cat-file -p <some-sha>` will echo out the contents of the sha. This can be a commit, a tree or a blob.

We can follow the trace of the SHAs to cat the content of a committed file:

![alt text](img/committedfileSHAs.png)

- tree: folder
- blob: file

## Git config

`git config --add --global <key> <value>`. Remember: every `key` is made up of two distinct parts, **section** and **keyname** (`section.keyname`). 

**To remove a section**: `git config --remove-section <section>`.

Problem: Let's create and add 3 values with the section 'fem' (frontend masters) and different keynames: dev is great, marc is ok, git would.

```bash
git config --add fem.dev "is great"
git config --add fem.marc "is ok"
git config --add fem.git "would"
```

**Listing out values**. Two options:

1. `--list`: where it will list out the entirety of the config. Example: `git config --list | grep fem`. This will show only those containing 'fem'.
2. `--get-regexp <regex>`: this takes a pattern and looks for all names matching. Example: `git config --get-regexp fem`. This will show the same values.

**Changing values**. We can add different values to the same key, so `git config --add fem.dev "is cool"` will not replace "is great" value, it just will add a new one.

```java
git config --add fem.dev "is cool"
git config --get-regexp fem
// It will show the 3 prior values and the new one.
git config --get fem.dev
// This just shows the latest one, "is amazing"
git config --get-all fem.dev
// This will show all fem.dev values

git config --unset-all fem.dev

```

We remove those values using `--unset`, but in cases where we have multiple value for one key, we should unset all at once, `git config --unset-all fem.dev`.

> [!important]
> **`--global` vs `--local`**
> 
> Remember if you're using `--global` to add these sections and keys. If you added with with `--global`, now you should remove them with it. `git config --global --unset-all fem.dev`.
> If you added them without this (or including `--local`, which is the same), you should delete them (or print them or whatever) in local.
> We can have the same keys and values both in local and in global.

Now let's **remove an entire section** using `--remove-section`.

**Question**. Let's make our default branch from `master` to `trunk`. The config setting is `init.defaultBranch`. Should this be a local or global change? It should be global if you want it to take place for ALL repos for ALL time on your WHOLE system. `git config --global init.defaultBranch trunk`. So if you start in a different folder a new git project, its main branch will be called 'trunk'. In case you just want to change the name for this repo, use `--local` (please).

## Git Branching

**You don't always want to be developing on the main branch**. Sometimes you need a feature that is developed off the main line, such that you can return to the main line, update the code, branch off, and perform some immediate needed fix.

This is where git branches come in. They are cheap, virtually free, to create. With our of understanding git internals this will become much more clear throughout this section

New branch: `git branch <name>`. This will just create it, git won't jump to the new branch.

We check our branches with `git branch`. 
Using `git log` we'll see that HEAD points to trunk, but there's also foo. Foo it's at the exact same place. **When we create a branch it branches off wherever you're at. It's pointing to the same commit**. 

**A branch POINTS to a commit, it is not a set of changes.**

![Git branch output](<img/git branch.png>)*Git branch output*.
![Output of git log](<img/git log branches.png>)*Output of git log*.

If we explore .git, we'll find our SHAs in .git/refs/head and the name of each branch. The branches just exists in this *heads* folder as just a SHA. Eventually a branch is just a SHA, and a SHA is the state of your repo. It's not a differential, that's why it's always able to do it and do it so fast.

![alt text](<img/foo in git refs.png>)

**Switching branches**. `git switch <branch-name>` or `git checkout <branch-name>`. It's the same.

Let's switch our branch to foo, create a new file (second.md), make two commits to it and check the state of our repo with `git log`.

![Current state of the repo](<img/state of repos after foo.png>)

This is the setup. A contains trunk and foo is at C.

```
    B - C foo
   /
  A       trunk
```

**Deleting branches**: `-d` or `-D`. 

### Merge

We have A, B and C, but here's what we desire now:

```
   B --- C    foo
 /
A --- D --- E trunk
```

So we switch to trunk and, edit README.md twice and commit each time with D and then E (`git switch trunk`, `echo "D" >> README.md`, `git add README.md`, `git commit -m "D"`...). Let's see what `git log --oneline` shows: 

![git log --oneline output](<img/git log --oneline commits.png>)

Now let's check foo with `git log --oneline foo`:

![git log --oneline foo output](<img/git log --oneline foo commits.png>)

> *Remember: A commit is a point in time with a specific state of the entire code base.*

**What's a merge?** *A merge is attempting to combine two histories together that have diverged at some point in the past*. There is a common commit point between the two, this is referred to as the best common ancestor ("merge base" is often the term used in the docs).

To put it simply, the first in common parent is the best common ancestor. git then merges the sets of commits onto the merge base and creates a new commit at the tip of the branch that is being merged on with all the changes combined into one commit.

**How to do it?** → `git merge <branchname>`. The branch you are on is the target branch and the branch you name in `<branchname>` will be the source branch. 

**Problem**. Given the following state, merge foo onto trunk. Since we want to keep foo and trunk in its current state for other problems later in this lesson. Please start by branching off of trunk. 

1. We create a new branch (trunk-merge-foo) and we switch to it: `git checkout -b trunk-merge-foo`.
2. `git merge foo`. We're asked to write a commit message.
3. `git log --oneline --graph --parents`. ![git log --oneline --graph --parents](<img/git log --oneline --graph --parents.png>). We can see that the merge commit has two parents. Its second ID (2bff...) comes from trunk. The third one (d031...) comes from foo. One is C and the other one is E.

Now, let's create the following git setup:

```git
              X - Y     bar
             /
A --- D --- E           trunk
```

1. `git checkout trunk`
2. `git checkout -b bar`
3. `echo "X" >> bar.md`, `git add .`, `git commit -m "X"`
4. `echo "Y" >> bar.md && git add . && git commit -m "Y"`
5. `git log --oneline --decorate --graph`
   - `* 321fcf0 (HEAD -> bar) Y`
   - `* e65eafb X`
   - `* 2bffa4f (trunk) E`
   - `* ec41551 D`
   - `* 6bba11c A`

Important to notice: In the prior example, foo and trunk both had diverged. B and C were in foo and D and E were in trunk. In this case, the best common ancestor is the tip (the last commit) of trunk, so only bar has diverged.

Let's merge trunk and bar, but not in a new branch.

1. `git checkout trunk`
2. `git merge bar`. Now there's no commit message. Why? Because of what we've just said. The best common ancestor was the tip of the branch you are merging onto, which means that it can just take the commits and just update the pointer. That's all, it doesn't have to do any sort of merging.It already works.You've already resolved any conflict.It can just simply fast forward the merge.

In the merge notes we see the following line: **`Fast-forward`**. That means we have a fast forward merge. `ff-merge` literally means you have a linear history with no divergence and you can just simply take the commits and update the pointer.

### Rebase

Rebase might be dangerous if it's not used well.

Rebase allows us to update underneath our set of changes. This is rewriting history in some sense, but it's allowing us to have what is currently the reality, then our changes as opposed to our changes that are taking.

```git
Our repo now:

   B --- C                foo
 /
A --- D --- E --- X --- Y trunk


Our repo later, using rebase.
Rebase will update foo to point to Y instead of A:

                           B --- C                foo
                         /
A --- D --- E --- X --- Y                         trunk
```

Rebase updates the commit where the branch originally points to. This also means that if we decide to merge foo into trunk we can do a ff-merge!

The basic steps of rebase is the following:

1. Execute `git rebase <targetbranch>`. I will refer to the current branch as `<currentbranch>` later on.
2. Checkout the latest commit on `<targetbranch>`, switching branches to the target branch.
3. Play one commit at a time from `<currentbranch>`. It's gonna take the branch that you were on, the current branch, and play the commits one at a time Time in order onto the target branch.
4. Once finished will update `<currentbranch>` to the current commit SHA. It will go back check out your current branch and update current branch to point to that latest commit.

**Problem**. Rebase foo with trunk. Please create a separate branch foo-rebase-trunk. This branch should be created off of foo. Once you have rebased foo-rebase-trunk with trunk check out the git logs with --graph and --decorate to see what has happened (--oneline can make it easier to read).

![rebasing](<img/first rebase.png>)

We see trunk, but also 2 commits that differed from trunk. After Y, history is linear.

We will see that trunk (which contains bar after our fast forward merge) is now the new base for foo-rebase-trunk. In other words, foo is no longer diverging from trunk. If we choose to merge foo into trunk we can do so via ff-merge (no merge commit).

- 👍 **Pros of rebase**. When using rebase you can have a clean history with no merge commits. If you are someone who uses git log a lot this can really help with searching.
- 👎 **Some cons**. It alters history of a branch. That means that if you already had foo on a remote git, you would have to force push it to the remote again. We will go over this more it shortly
- 🚧 **Some cautionary words**. NEVER CHANGE HISTORY OF A PUBLIC BRANCH. In other words, don't ever change history of trunk. But your own personal branch? I don't think it matters and i think having a nice clean history can be very beneficial if you use git to search for changes through time.

### HEAD

If we checkout to trunk and make a log, we'll see HEAD is pointing to trunk. If we do the same with foo, we'll see HEAD pointing to foo. And `cat .git/HEAD` will show something like `ref: refs/heads/foo`, because we've just switched to foo. So **HEAD is just a pointer to whatever we're using**.

#### Reflog

It allows us to see where HEAD has been. `git reflog`.

![git reflog output](<img/git reflog.png>)

`reflog` is just a pretty version of `.git/logs/HEAD` file:

![cat .git/logs/HEAD](<img/cat git logs HEAD.png>).

We can specify the number of lines we want to see: `git reflog -3`.

**Problem**. 1. Create a new branch off of trunk, call it baz. 2. Add one commit to baz. Do it in a new file baz.md. 3. Switch back to trunk and delete baz (git branch -D baz from earlier). 4. Can you bring back from the dead the commit sha of baz?
1. `git checkout trunk`, `git checkout -b baz`.
2. `echo "baz" >> baz.md && git add baz.md && git commit -m "baz"`.
3. `git checkout trunk`, `git branch -D baz`.
4. Now we'll try to recover it.
   1. `git reflog -5`. We'll see the SHA of the commit. ![git reflog -5](<img/git reflog baz.png>)
   2. We could rebuild the entire repo just from that SHA: `git cat-file -p dd85b51`. Just because we deleted the branch did not mean we deleted what you just changed,the files are in the computer. ![git cat-file -p dd85b51](<img/git cat-file -p dd85b51.png>)
   3. Now let's see what was inside of that file.
   4. `git cat-file -p d2d8e10a88b4e985003930d45c5c488abe712e6b`. This was the SHA of the tree. ![git cat-file -p d2d8e10a88b4e985003930d45c5c488abe712e6b](<img/git cat-file -p d2d8e10a88b4e985003930d45c5c488abe712e6b.png>)
   5. `git cat-file -p 76018072e09c5d31c8c6e3113b8aa0fe625195ca` will show us the content of baz.md (which was just 'baz'). The we could add it to another file, also called baz.md (`git cat-file -p 76018072e09c5d31c8c6e3113b8aa0fe625195ca > baz.md`)

All that would be easier just using `git merge dd85b51`. The problem with it is that we could have found many merging between those commit and trunk. If we merge it, we'll get all that history. Maybe we've just wanted one specific commit. In that case, we could have used Cherry Pick.

### Cherry Pick

`git cherry-pick`

> Given one or more existing commits, apply the change each one introduces, recording a new commit for each. This requires your working tree to be clean (no modifications from the HEAD commit).

So let's suppose we didn't use the previous method and that we didn't recover baz.md. Now we'll use cherry pick.

![git cherry-pick dd85b51](<img/cherry picking.png>)