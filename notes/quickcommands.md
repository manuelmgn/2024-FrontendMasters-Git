# Git Quick Commands

- `man`
- `man git-<op>`: for the friendly manual
- `git init`:  initialize repo
- `git config`
  - `git config --add <key> "<value>"`: add a key
    - `--global`
    - `--local` or just nothing
  - `git config --unset <key>`: unset a value
  - `git config --unset-all <key>`: unset all values for a key
  - `git config --get <key>`: view values of git config
  - `git config --remove-section <section>`: removes a section
  - `git config --list`: list out the entirety of the config
  - `git config --get-regexp <regex>`: takes a pattern and looks for all names matching
- `git add <path-to-file | pattern>`: add files to staging area
- `git commit -m '<message>'`: will commit what changes are present in staging area.
- `git status`: describe the state of your git repo
- `git log`: show history
  - `--no-decorate`: print out the ref names of any commits that are shown
  - `--decorate`
  - `--graph`: draw a text-based graphical representation of the commit history
  - `--oneline`
- `git cat-file -p <some-sha>`: echo out the contents of the sha
- `git branch <name>`:  create a branch
- `git branch`: check our branches
- `git branch -D <name>`: deletes a branch
- `git branch -a`: shows all existing branches
- `git switch <branch-name>` or `git checkout <branch-name>`: switch to other existing branch
- `git checkout -b <branch-name>`: creates a branch and jumps to it
- `git merge <branch-name>`: merges two branches
- `git rebase <target-branch>`: rebases two branches
- `git reflog`: shows where HEAD has been
- `git cherry-pick`: recover one commit
- `git remote add <name> <uri>`: adds a remote branch
- `git fetch`: fetch all the git state from the remote repository
- `git pull <remote> <branch>`:  fetch changes and merge them into the branch
- `git push`
- `git stash`: takes every change tracked by git and stores that result into the "stash"
  - `git stash -m "<message>"`
  - `git stash list` list of stashes
  - `git stash show [--index <index>]`: just show the changes or the diff
  - `git stash pop`  pop the latest stash
