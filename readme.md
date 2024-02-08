## Notes
Every time something is surrounded by `<>`, assume you should replace its content with something relative to your specific context. Factors such as the operating system, environment, and various other considerations can affect what works best for your specific case. 

For example:

```bash
# this:
git clone <repo-link>

# could be something like this:
git clone https://github.com/MarceloCFerraz/GitCheatSheet.git

# or something like this:
git clone git@github.com:MarceloCFerraz/GitCheatSheet.git
```

## Fix Last Commit Message

To fix the last commit message, you can use the command:

```bash
git commit --amend -m "message"
```

This will allow you to modify the message of the most recent commit.

## Logs of Applied Commits

If you want to see the logs of commits that have been applied, you can use the `git log` command. Remember, you can type `q` to quit logging.

## Logs of All Commits

For viewing logs of all commits, including pulls, resets, and rebases, you can use the `git reflog` command. Again, you can type `q` to quit.

## Messing with Staging Area
### Unstaged

- `git checkout -- <path-to-file-or-dir>` to discard all changes on a specific file or folder
- `git checkout --` to delete all changes on all files and folders at one
### Staged

- `git reset --`: remove everything from stage
- `git reset -- <path-to-file>`: remove only a specific file from stage
- `git reset --hard HEAD <path-to-file>` discard all changes on a specific file of folder
- `git reset --hard HEAD` discard all changes on all files and folders at once
- `git reset HEAD` + `git checkout --` to unstage first and only then discard changes. Prefer this way over previously mentioned alternatives.
- `git rm --cached <path-to-file>`: remove a specific file from stage and also deletes the file from your working directory
- `git restore --stage <path-to-file>`: remove a specific file from stage but don't delete it

Both `checkout --` and `reset --hard HEAD` replace the content of updated files with the contents of the same file on the last commit. The differences are:

- `reset` will be listed in `reflog`
- `reset` will point back to the last commit and if done multiple times - if there is nothing staged -, can keep going back and pointing to previous commits (untested)
- `checkout` will only replace contents without pointing anywhere, just gets the content and replace, but does not work on staged files.

## Check Differences Between Local and Remote

You can compare your local changes with the same file on the remote server using the `git diff` command. Here are the steps:

1. Fetch all changes from the remote repository:
```bash
git fetch origin
```
`git fetch` updates your local knowledge of the remote repository without making any changes to your local branches. This ensures that you're comparing your local changes to the most recent state of the remote branch. 

2. Run the `git diff` command to see the differences between your local branch and the remote branch:
```bash
git diff <local branch> origin/<remote branch>
```

Replace `<local branch>` and `<remote branch>` with your actual local and remote branch names. Also, make sure to replace `origin` with the name of your remote if it's different.

`git fetch` updates your local knowledge of the remote repository without making any changes to your local branches. This ensures that you're comparing your local changes to the most recent state of the remote branch. 

If you want to see the differences for a specific file, you can specify the file path at the end of the `git diff` command like this:
```bash
git diff <local branch> origin/<remote branch> -- <file path>
```
Just replace `<file path>` with the actual path of your file. This will show you the differences between your local version of the file and the version of the file on the remote branch. 

## Split Changes On The Same File Into Different Commits

For this, git uses a concept called `hunk`, which separates every modified piece of code into separate sections. Then we can decide which sections (hunks) will go into the next commit.

Use `git add -p`. If changes are close, git will list all hunks as a single unified hunk and will prompt you `Wish to stage this hunk?` (a bulk hunk) and go through next hunks individually and sub-sequentially if there are any.

If there is a file which had lines 10, 12, 30, 50, 51 and 53 modified, git will probably initially join close changes and go through them individually asking which we want to add to stage. For example: 

- `hunk 1` = lines 10 + 12
- `hunk 2` = line 30
- `hunk 3` = lines 50 + 51 + 53

Use option `s` to split hunk into smaller hunks then go through them individually and sub-sequentially. At `hunk 1`, split would create two more hunks:

- `new hunk 1` = line 10
- `new hunk 2` = line 12

Then we could either add hunks to stage by typing option `y` or ignore it with option `n`. After the last hunk the command will finish unless you choose another option besides `y` or `n` such as `j`, `J`, `g` or `/`.  Choose option `?` to read an explanation of each option (recommended). You can type `q`  and hit `Enter` to quit any time.

## Rollback Commits

To rollback commits, use:

```bash
git reset <mode> <reference>
```

 ``mode``: can be ``--soft``, ``--mixed``, `--merge` or ``--hard``.
 
- ``soft``: Does not touch the index file or the working tree at all, but resets the head to commit. This leaves all your changed files ``Changes to be committed``, as git status would put it.
- ``mixed`` (default): Resets the index but not the working tree (i.e., the changed files are preserved but not marked for commit) and reports what has not been updated. This is the default action4.
- ``hard``: Resets the index and working tree. Any changes to tracked files in the working tree since the selected commit are discarded.
- `merge`: Resets the index and updates the files in the working tree that are different between the specified commit and HEAD, but keeps those which are different between the index and working tree (i.e. which have changes which have not been added).

``reference``: is either a direct reference to a ``commit`` you want to reset to or a number to make HEAD index rollback. 

- `commit`: Using a direct reference will reset the current branch to the specified commit. 
- `HEAD~<n>`: Using a index number will make HEAD do `n` rollbacks and stop there.

Either reference can update the index (resetting it to the tree of where it stops) and the working tree depending on the mode

For example Imagine you have 3 commits and their hashes are:

```bash
123 (newest)
1234
12345 (oldest)
```

If you want to roll back to `12345`, meaning you want to uncommit both `123` and `1234`, then use `git reset --soft 12345`. Every file updated by those commits will be staged again but changes won't be lost. 

## Rollback a Single Commit

You might also want to revert only changes introduced by a single commit, instead of reverting every commit that came after it. Let's use the same example as we did before, imagine you have 3 commits and their hashes are:

```bash
123 (newest)
1234
12345 (oldest)
```

Instead of rolling back until `12345`, now I only need to rollback `1234`. To do so, we can use `git revert`.

``git revert`` creates a new commit that undoes the changes introduced by a specified commit. This means that if a file was created in the commit you're reverting, that file will be deleted in the new commit. If a file was deleted in the commit you're reverting, that file will be restored in the new commit. 

However, it won't delete or restore files in your working directory immediately. Instead, it stages these changes and waits for you to commit them. 

Here's an example of how we can use it **to keep files and updates from the commit**:

```bash
# revert, but do not commit yet
git revert -n 1234
# clean all the changes from the index
git reset
# now just add the file you want to keep the changes
git add <file>
git commit -m "reverting changes but saving <file>"
```

If you don't want to keep anything from the commit, just use:

```bash
git revert --no-edit 1234
```

This will create a new commit with a default message reverting everything done at `commit`.

**Tip**: Before rolling back with any of these methods, use `git log` to save all commits in a text editor. This helps to revert the rollback. You can use `git log` or `git reflog` to get `<commit-hash>`, but prefer `git reflog` to get `n`.

## Revert Rollback

To revert a rollback, use:

```bash
git cherry-pick <commit-hash>
```

- `<commit-hash>`: hash of the commit to recover.

Note: If you have uncommitted changes in your local repo, either commit or `stash` changes them before running `cherry-pick`. Check out how to stash below.

After running `cherry-pick`, stage changes with `git add` and commit changes again. If you rolled back more than one commit, do this for every commit you wish to recover individually. 

## Stash Changes

Stash basically get your changes on the working tree and save them for later. That's useful when you have to pull or push and don't want to commit those changes yet for whatever reason.

To stash everything, use:

```bash
git stash
```

You can see a list of stashes you did with:

```bash
git stash list
```

To put stashed changes back to the working tree, use:

```bash
git stash apply
```


These commands are general, meaning they will stash everything and apply everything from and to your working tree. However, you can also be specific on what you want to stash. To stash a specific file, you can use the following command:

```bash
git stash push <path-to-file>
```

You can also stash multiple files by specifying each file path:

```bash
git stash push <path-to-file1> <path-to-file2>
```

And if you want to stash all the files in a specific folder, you can do so by specifying the folder path¹:

```bash
git stash push <path-to-folder>
```

Note: `git stash` will stash the changes and revert the files back to your last commit. To recover what you stashed, apply your stash again. 

You can also specify what stash you want to apply. To do so, first use `git stash list` and then use:

```bash
git stash apply stash@{n}
```

`n` is the number of the stash on the list.

You can clear your stash list with


```bash
git stash clear
```

## Remove a Commit From Git Logs

Doing so demands rewriting the whole git history, so beware of that before proceeding. First, find the hash of the commit to be removed with `reflog` or `log`, then you can use this command to remove it from the history:

```bash
git rebase -i <commit-hash>~<n>
```

This command will open an interactive window using your configured default editor for you to individually choose what to do to the commit. To remove it, mark it as `drop` in the interactive window and `start rebase`.

Again, this will rewrite all your history, so at least make a copy of your repo with a clean `git clone --mirror <repo-link>` somewhere else. This will download your repo in a different format. It basically download mostly refs, so prefer to use it as a back up.

Regretted it? Then do this:
1. `git reflog`
2. Get the commit hash prior to your `rebase`/`reset`
3. `git reset --hard <commit-hash>`
4. `git push origin <branch> -f`
5. `git log` to check 

Haven't regretted it? Then, after rebasing, double-check with log/ reflog, but note that since garbage collector didn't clean the mess yet, you'll still see everything. 

So, run this:

```bash
# this will clean the local history
git reflog expire --expire=now --all && git gc --prune=now --aggressive
# this will force changes to the remote repo (last chance to make a backup)
git push origin -f
```

Note that this will force a rewrite of your repo, so be careful pushing something like this to remote

## Remove Unwanted Files and Content From Git History
This is a faster alternative to `git-filter-branch`, but a lot faster. Works for removing large files (or any file really) and/or sensitive content from files such as passwords.
1. Download and Install Java JDK 8+
2. Configure your JAVA_HOME environment variable if you haven't yet
	- go to the folder where you installed Java JDK
	- go to bin folder
	- copy the path to that folder
	- on windows, open PowerShell and type 

```powershell
[Environment]::SetEnvironmentVariable("JAVA_HOME", "<paste the path to bin folder here>", "Machine")
```

And then: 

```powershell
[Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path", "Machine") + ";" + $env:JAVA_HOME, "Machine")
```
You can also replace `Machine` with `User` and then only you will have access to Java

3. Download [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/)
4. Do a clean clone of your repo with `git clone --mirror <link to your repo> [OPTIONAL] <specify output dir>`. This will download a bare repo, which doesn't contain your files but all your working history
5. Run `java --jar <bfg jar file> --delete-files [<files or file extension>] <your bare repo dir>`
6. Run `cd <your bare repo dir>`
7. Run `git reflog expire --expire=now --all && git gc --prune=now --aggressive`
8. Then you can do `git push`. This will update all refs on your remote server, so only do this if it is really mandatory

Let's use an example:

My repo had some example xml files that i wanted to remove, because no one should download any of that. They were used locally in a test scenario and shouldn't be on the repo in the first place.
Let's suppose all those files had a `ExampleDoc` prefix, all of them were `.xml` and let's also say my repo was hosted in the theoretical remote address `github.com/Marcelo/ExampleXml.git`. Now we can follow the steps above.
1. Set up JAVA
2. Downloaded bfg do `%USERPROFILE%/source/`
3. Opened a terminal there
4. Clone repo with `git clone --mirror github.com/Marcelo/ExampleXml.git`
5. I also zipped `ExampleXml` folder before doing anything just to have a backup
6. Then ran `java --jar bfg.jar --delete-files ExampleDoc*.xml ExampleXml`
	- `*` is part of a regular expression. It basically says anything in between `ExampleDoc` and `.xml` can be ignored to provide a correct match.
7. Then ran `git reflog expire --expire=now --all && git gc --prune=now --aggressive` followed by
8. `git push`
	- you might need to use flag `-f` to force push
That's it. All git history modified to not include any example xml files. You can do the same to remove passwords and other private stuff from your git history. Although you might have already removed them in new commits, someone with your git history can also get contents of any file modified since the beginning of your repository. Always prefer using environment variables for things like passwords and secret keys and remember to also add any `env` folder or file to your `.gitignore` file ;)
