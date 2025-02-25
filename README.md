# git-trix

## Config Settings
```
[fetch]
  prune = true
[push]
  autoSetupRemote = true
[pull]
  rebase = true
[rerere]
  enabled = true
  autoupdated = true

```

## Visual Studio: I can’t pull a branch! “branch does not exist”
This is an annoying thing where GIT is case-sensitive, but devops is not. I had branches created as “work/RayCapizzo” and then “work/raycapizzo”, and users who had pulled “RayCapizzo” could not pull “raycapizzo”.
THE FIX:
-	Switch to another branch (e.g. dev)
-	Delete the trouble branch (e.g. work/RayCapizzo)
-	In file-explorer, navigate to your .git/refs folder (repository/.git/refs/remotes/origin) and delete the problem folder (/work/RayCapizzo)
-	Do a fetch
-	Now you should be able to pull down the branch again.

## I need to rename my branch!
- If the branch hasn't been pushed yet:
  - `git branch -m <old-name> <new-name>`
- If the branch HAS been pushed:
  - `git branch -m <old_name> <new_name>`
  - `git push origin --delete <old_name>`
  - `git branch --unset-upstream <new_name>`
  - `git push origin <new_name>`
  - `git push origin -u <new_name>`
 

## I forgot to add something to my commit
Sometimes you do a feature, commit, then build, and the files change. Instead of undoing the last commit to add these changes or adding another noisy commit:
-	stage the files you want git add .
-	amend your last commit with the staged files git commit --amend
-	__!! NOTE__ this only works before pushing. What this really does is make a NEW commit with this + the old changes. Locally is okay, but pushing falls into the “rewrite history” category.

## I need to undo the last commit
BEFORE pushing:
- git reset HEAD~ – If you do this repeatedly, it will keep pulling the changes from the last commits into the “staging” area, until you get to 0/0. Then you can undo those changes, or just the ones you want. 
  - you can also do `git reset HEAD~n` to move back `n` commits
- Also helpful is: 
  - look in git history for the commit Id that you want to roll to
  - run git reset --hard {commit-id} – this will reset your local history to that commit.
- Another version:
  - Look in the incoming/outgoing in VS, right click and say “revert”
  - “soft” will keep the changes as staged/working-area changes, “hard” will undo them
__!Note!: using hard can delete your work! If that happens, see reflog__
## I accidentally reset/deleted/rebased/borked my work!
-	run git reflog
-	type “q” to exit
-	find the “head@{x}” that you want to roll back to
-	run git reset --hard "head@{x}", substituting the appropriate x.

## I need to reset my local branch
-	If somebody did some push -f, you will need to force your local branch to accept incoming changes.
-	git reset --hard origin/{your-branch} – you should end up at 0/0.

## I want to undo my uncommitted changes
-	git checkout . will undo your modifications
-	git clean -n does a dry-run of deleting untracked files (any that have been added)
-	git clean -f to actually delete the files

## Checkout Commit (detached head)
-	git checkout 757c47d4

## Stash
-	Stash all changes
  -	git stash
  -	Stash only unstaged items with git stash -k
  - use case: splitting work into different commits, but need to regenerate Swagger (stage the code to commit, stash the rest, rebuild, stage the Swagger changes, commit, un-stash)
- reapply the items
  - git stash pop
## worktree (multi tasking)
-	Create worktree – this creates another folder that shares refs with your main.
  -	You can open each solution in a separate VS window and work on each separately
  -	Branches/commits/fetch/pulls are shared (creating a branch in one shows in the other, pulling changes to ‘dev’ will pull for both instances, etc)
  -	You cannot checkout the same branch in both worktree folders
  -	`git worktree add -b <new branch> ./../new-folder <from branch>`
-	Show all worktrees: git worktree list
-	Remove worktree: git worktree remove new-folder (note: not the full path, just the folder-name)
-	Prune? git worktree prune
## I want to prune!
-	fetch and delete refs to branches that have been deleted on the remote
  -	git fetch --prune
  -	( do this automatically by setting git config --global fetch.prune true, then it will be done every fetch )
## What Changed?
-	git log – show recent commit messages
-	git log --raw – show all files that changed
-	git log --patch – show all changes
## When did THAT happen? ( search log )
-	git log -S "word" -- find the commits where this word happened (added/removed)
-	git log -S "word" -p -- show the diffs
-	git log -S "word" -p file -- create patch file

## I want to clean up my branches
-	prune branches and write these to a file:
  -	git remote prune origin && git branch -vv | grep ': gone]' | awk '{print $1}' > branches-to-delete.txt
  -	review the branches in the file, with the help of git branch -vv (in a separate window)
  - remove any branches that you want to keep from the file
  -	save, quit
  -	delete all branches in the file:
  -	while read r; do git branch -D $r; done<branches-to-delete.txt
## Rebasing, undoing commits:
You can change commits after you’ve pushed, but keep in mind that anything in the cloud is a shared work environment; if you’re “rewriting history” on something that somebody else has pulled, give them a heads up to ensure a smooth experience.
## Force Push: For changes marked with  , you’ll want to force your changes up into the remote, overwriting the history there. 
git push -f
## I want to reset my branch to whatever is in remote
If your branch has gotten into a weird state, you can undo everything you’ve done and reset to remote with
git reset --hard origin/[branch-name]
## I need to undo a rebase (done on your own computer):
Run git reflog to see which HEAD@{#} to roll back to
- git reset --hard "HEAD@{1}" – (quotes are required for for Windows)
## I want to ignore/combine/rename some earlier commits
- find the commit id of the earliest commit
- git rebase -i {commit id}
- Edit the text file:
- remove commits that you want to ignore
- read through directions to discover other possibilities
## I branched off the wrong branch
- If your have changes from {wrong-branch} that are ahead of {right-branch}, this gets more complicated…
- So far, the git rebase -i {commit id} has been a good choice – find the last “meaningful” commit that you want to go back to and use that as the commit id.
- Erase the lines you don’t want.
- finish rebasing
- push -f.
  - Probably git rebase --onto might be better..
  - DO NOT merge {wrong-branch} into your branch – that makes things much more complicated.
  - git rebase origin/{right-branch} – sometimes this is the right answer, and sometimes using --onto works better.. todo: figure out the difference.
  - THE (very informative) DIFFERENCE: https://stackoverflow.com/a/33948237 
  - git rebase --onto origin/{right-branch} {wrong-branch}
- Should get you into a pretty good state where your changes are appended to the current head of right-branch. 
## I want to undo changes AFTER COMMITTING:
  - Anything done AFTER pushing means you need to “rewrite history”, which is scary but doable.
  - If it’s your branch, and it’s unlikely that anyone else is working in it, it’s less scary.
  - If it’s a branch that people ARE working in, you must coordinate with them so they know what to expect when pulling. It’s confusing and they have the opportunity to undo what you’ve done. Yuck.
  - From the Git History, you can RESET --hard to the commit that you want to be on
  - Then git push -f to force your local branch to be the remote branch
## I need to undo the last merge
- These can be very confusing. git rebase -i HEAD~1 will give you the list of commits involved with the merge. You can delete the commits that were not yours.
- Sometimes there are merge-conflicts when rebasing. You must solve each of these and then git rebase --continue
It is often helpful in rebasing to do git status to see where you are and what your options are.
Common choices:
  - git rebase --abort
  - git rebase --continue
## I force-pushed and overwrote someone else’s changes!
  - I blindly sent git push -f on dev branch after not pulling for a while, and my latest HEAD pushed all subsequent commits out of the history tree. 
  - !! After running the force-push, do not close the terminal !!
  - You get a line that describes the commit change
-  + 000dfed25...c7816791f dev -> dev (forced update)
  - The left-side is the old commit-id
  - You can search for that commit in DevOps to find who it belonged to
  - Find someone who has that commit on their computer (test with git checkout 000dfed25 – if they have it, they’ll end up in a detached state. Afterwards, flip back to the branch, e.g. git checkout dev)
  - While on the branch, run git reset --hard 000dfed25 to point the local HEAD to the correct commit, then git push -f to the remote.
  - (Link: https://www.jvt.me/posts/2021/10/23/undo-force-push/ )

## I want to ignore a file just locally
- add your ignore rule to `.git/info/exclude`
  - https://docs.github.com/en/get-started/getting-started-with-git/ignoring-files#excluding-local-files-without-creating-a-gitignore-file
## .gitIgnore isn't working!
- `git rm -r --cached .` -- (for single file, `git rm --cached filename`)
- `git add .`
- `git commit -m "fixed gitignore"`

## other notes:
### Edit config file in editor
- git config --global --edit
### Find branch parent (alias):
- parent = "!git show-branch | grep '*' | grep -v \"$(git rev-parse --abbrev-ref HEAD)\" | head -n1 | sed 's/.*\\[\\(.*\\)\\].*/\\1/' | sed 's/[\\^~].*//' #"
### delete a branch: git branch -d <branch-name>
- use -D to ignore warnings
### find branches that have been merged to the current branch (e.g. dev)
- git branch --merged
### prune (detach branches that have been deleted upstream)
- git remote prune origin
- (“origin” is the remote – see your remotes with git remote -v
### Prune and delete detached branches: 
- [alias]
-   prune-branches = !git remote prune origin && git branch -vv | grep ': gone]' | awk '{print $1}' | xargs -r git branch -d
### Write “gone” branches to text file:
- git branch -vv | grep ': gone]' | awk '{print $1}' > branches-to-delete.txt
### git branch --format "%(refname:short) %(upstream)" | awk '{if (!$2) print $1;}'
- https://stackoverflow.com/a/31776247 
### Delete branches in a file:
- (to pipe to a file: add > branches-to-delete.txt to… most things)
- edit the file nano branches-to-delete.txt
- remove rows you DON'T want to delete
- save changes (ctrl x, y)
- delete those branches:
- `while read r; do git branch -D $r; done <branches-to-delete.txt`


## Helpful links:
- https://opensource.com/article/22/4/git-push
- https://devconnected.com/how-to-clean-up-git-branches/#Git_Remote_Prune 
- https://medium.com/tempus-ex/what-exactly-is-a-git-commit-1cac6abcf73c 
- https://opensource.com/article/21/4/git-worktree
- https://blog.gitbutler.com/how-git-core-devs-configure-git/
## Learning links:
- https://gist.github.com/qoomon/5dfcdf8eec66a051ecd85625518cfd13 ← organizing commits by “what they do”
- https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging ← tutorial for basic skills  
- https://www.freecodecamp.org/news/git-diff-and-patch/ ← What’s a Patch?
