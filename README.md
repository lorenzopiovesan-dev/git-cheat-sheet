# GIT CHEAT SHEET

This is a "quick reminder" list of simple use cases on how to use git via command line that I've gathered and arranged, when coming across them while working and/or studying, throughout the years... Sooner or later I will update it with the many more use cases I had the luck, or unluck, to discover... Maybe even adding some information about this incredible version control system... but this is it for now :) 


BASIC
---
* `git status` --> display the state of the working directory and staging area
* `git ls-files` --> display the currently tracked files
* `git push remote-name local-branch-name` (e.g. git push origin develop) --> push local branch to the remote repository
  * `git push -u origin local-branch-name` --> same as above but the tracking for the remote branch is set and there are no need for extra arguments when pushing/pulling the branch (e.g. next push/pull --> git push/pull)
* `git mv fileName newFileName` --> rename fileName to newFileName and stages it (can be used to revert itself)
* `git rm fileName` --> delete fileName and stages it
* `git log` --> shows the commits history 
* `git log --oneline --graph --decorate --all`  (fancier git log)
* `git log commitIdStart...commitIdEnd` --> log all the commits between (and including) commitIdStart and commitIdEnd
* `git show commitId` --> shows the changes in the commitId

BACK-OUT
---
* (from staged, after git add) --> `git reset HEAD fileName`
* (reset fileName to last commit) --> `git checkout -- filename`

DELETE
---
* (from staged, after git status) --> from bash ``rm -rf fileName``
* (from committed, tracked) --> `git rm fileName`

FROM STAGED - what can we do
---
1. we can back-out
+ `git reset HEAD fileName` then
   + Or `git checkout -- fileName` to confirm the back-out
   + Or `git add fileName` to re-add
   + Or `git rm fileName` to remove
2. we can confirm what has been done
+ `git commit -am "commit_message"` to confirm the commit

ALIASES
---
We can create aliases (aka shortcut) to commands we often use

`git config --global alias.commandAliasName "git_command_without_git`

e.g.
```
git config --global alias.history "log --oneline --graph --decorate --all"
will create the git command/alias
================
git history
```

DIFFERENCES
---
* `git diff HEAD` --> shows the differences FOR TRACKED FILES ONLY between the current workspace state and the HEAD
* `git diff --staged HEAD` --> shows the differences between the current workspace state INCLUDING STAGED FILES and the HEAD
* `git diff -- fileName` --> shows differences for a specific fileName
* `git diff commitId HEAD` --> shows differences between a specific commitId and the HEAD
* `git diff commitId1 commitId2` --> shows differences between two specific commitIds
* `git diff HEAD HEAD~#` --> with #=1..N, shows differences between the HEAD and the #-th commit before
* `git diff localBranchName origin/remoteBranchName` --> shows differences between a local and a remote branch


BRANCHES
---
* `git branch` --> shows all local branches
* `git branch -a` --> shows all local and remotes branches
* `git branch branchName` --> create a local branch called branchName
* `git branch -m branchName newBranchName` --> rename the branch branchName to newBranchName
* `git branch -D branchName` --> delete branchName (it is not possible to delete a branch from within itself, first a checkout to a parent branch is needed)
* `git checkout -b branchName` --> automatically create the branch branchName and move in it

MERGE - steps
---
1. Move to the branch that will accept the merge e.g. `develop -> master ==> git checkout master`
2. Merge the branch
    + Without creating a merge commit `git merge branchName ==> develop -> master ==> git merge develop`
    + Creating a merge commit `git merge branchName ==> develop -> master ==> git merge --no-ff develop`
3. Resolve eventual conflicts
   ```
   The issue will be seen in the respective files in the following manner:
   
   <<< HEAD
   // code already present in the branch accepting the merge
   ===
   // code in the branch that is actively merged
   >>> branchName      
   
   The issues needs to be manually fixed with either a merging tool or normal text editor and then we need to commit the MERGE
   ```
4. Delete merged branch `develop -> master ==> git branch -D develop`

REBASE - steps
---
###### We perform a rebase in a branch when we want to incorporate in it changes happened in the parent branch.
    Performing a rebase will prevent, or at least reduce, the chances of having conflicts, when merging the branches.

1. Move to the branch we want to update with the rebase ```master -> develop ==> git checkout develop```
2. Perform the rebase on the branch updating with the latest commit from its parent ```git rebase parentName ==> master -> develop ==> git rebase master```
3. Resolve eventual conflict either
    + Or Aborting the rebase ```git rebase --abort```
    + Or fixing the issues:
    ```
    1. we manually fix the issues using either git diffmerge or Intellij editor (we don't commit yet)
    2. we restart the rebase with the fix --> git rebase --continue
    
    If succesfull will apply the changes and complete the rebase
    ```

REBASE from GIT PULL - steps
---
1. We update the local indexes with the latest remote version `git fetch remoteName branchName ==> git fetch origin develop` (it's a non-destructive operation)
2. We perform the remote rebase `git pull --rebase remoteName branchName ==> git pull --rebase origin develop`
3. We do all the commits we want
4. We push the changes made `git push remoteName branchName ==> git push origin develop`

SQUASH aka interactive Rebase - steps
---
###### Squash is an interactive rebase that lets us "squash" a certain number of commits altogether
0. we need to stage all the changes `git status --> git commit -am "message_for_commit`
1. `git rebase -i HEAD~#` with #=1..N --> we enter the interactive rebase to pick the commits to squash. We are telling git to consider the last # commits for the interactive rebase
   ```
   this will open the standard text editor and present something like this:
   --------------------
   
   pick c6dc8c7 completing the cheatsheet
   pick 829e51d adding SQUASH
   pick bd1620c adding further commit for squash purposes
   
   # Rebase 500b085..bd1620c onto 500b085 (3 commands)
   #
   # Commands:
   # p, pick = use commit
   # r, reword = use commit, but edit the commit message
   # e, edit = use commit, but stop for amending
   # s, squash = use commit, but meld into previous commit
   # f, fixup = like "squash", but discard this commit's log message
   # x, exec = run command (the rest of the line) using shell
   # d, drop = remove commit
   #
   # These lines can be re-ordered; they are executed from top to bottom.
   #
   # If you remove a line here THAT COMMIT WILL BE LOST.
   #
   # However, if you remove everything, the rebase will be aborted.
   #
   # Note that empty commits are commented out

   ```
2. we leave the first commit to ``pick`` (normally the bottom one is the last one) and we change all the other (BELOW) commits to ``squash`` --> we are telling to squash all the commits into the picked one
3. we save and exit and when prompted we add the new rebase message e.g. Squashed commits for XYZ reasons
4. Resolve eventual conflicts like in a normal rebase


STASH - steps
---
###### Stashes are really similar to branches, they will allow us to stash a workspace and/or stage to re-use it later when we are ready to commit it

0. We change what we need to temporarily change
1. we can check what's the current stage by performing `git status`
2. we stash the files we need (this will add a stash to a stack of Stashes):
   + only tracked files `git stash save "message_to_identify_the_stash"`
   + all the files tracked and untracked `git stash -u save "message_to_identify_the_stash"`
   ```
   at this point the branch is restored to the last commit and all the saves are stored away
   ```
3. we can perform all the changes and commits we want
4. If we want to recover and re-insert in the branch the amendments we saved in the stash
   ```
   git stash apply --> will load the last stash saved
   git stash apply stash@{#} --> with #=1..N will load the specified stash number #
   ``` 
5. We might have to resolve some conflicts like we would do with merge
6. Once finished we remove the stash we used
   ```
   git stash drop --> remove last saved stash
   or
   git stash drop stash@{#} --> with #=1..N will delete the specified stash number #
   ```
#### STASHES - useful commands
* `git stash` --> save current tracked files
* `git stash save "message_to_identify_the_stash"` --> save current tracked files with a message to identify it (useful for multiple stashes)
* `git stash -u` --> save current tracked and not tracked files
* `git stash save -u "message_to_identify_the_stash"` --> save current tracked and not tracked files with a message to identify it (useful for multiple stashes)
* `git stash apply` --> apply to the current branch the last saved stash
* `git stash apply stash@{#}` --> with #=1..N will load the specified stash number #
* `git stash list` --> shows the list of stashes saved
* `git stash show stash@{#}` --> with #=1..N will show the content of the specified stash number #
* `git stash drop` --> delete the last saved stash
* `git stash drop stash@{#}` --> with #=1..N will delete the the specified stash number #
* `git stash clear` --> will empty the Stack of stashes
* `git stash pop` --> automatically perform both `git stash apply --> git stash drop`

#### STASHES - what can we do with them
* We can apply them `git stash apply`
* We can drop them `git stash drop`
* We can create new branches off them
   1. we stash everything we need `git stash -u`
   2. we create a branch `git stash branch newBranchName` --> automatically perform `git chekout -b newBranchName --> git stash pop`


TAGS
---
###### Tags are labels to apply to commits and can be used in normal operations like commitIds
* Create TAGS
  * `git tag tagName` --> create a lightweight tag for the current commit
  * `git tag -a tagName commitId` --> create a lightweight tag for a specific commitId
  * `git tag -a tagName -m "tag_message"` --> create an Annotated tag, for the current commit, named tagName (normally like "v-1.0") that will include more details about the commit and a tag_message. Normally represents milestones.
  * `git tag -a tagName -m "tag_message" commitId` --> create an Annotated tag, for a specific commitId, named tagName (normally like "v-1.0") that will include more details about the commit and a tag_message. Normally represents milestones.
* Delete TAGS
  * `git tag --delete tagName` --> delete tagName (doesn't delete the commit)
* Update TAGS (moving tag to other commit)
  * Or `git commit --amend` --> let us amend the last commit description
  * Or `git tag --delete tagName --> git tag newTagName`
  * Or forcing the update `git tag -a tagNameToMove -m "optional_description" -f otherCommitId`
* retrieve INFO
  * `git tag --list` --> shows all the tags
  * `git show tagName` --> is the same of `git show commitIdTagPointsTo`
  * `git diff tagName1 tagName2` --> is the same of `git show commitIdTagPointsTo1 commitIdTagPointsTo2`

###### when pushing commits to a remote, tags needs to be handles separately
* `git push remoteName tagName ==> git push origin tagName` --> this will push to a remote repository the specified tag, and its commits
* `git push remoteName branchName --tags ==> git push origin develop --tags` --> this will push all the tags in a specific branchName to a remote repository
* `git push remoteName :tagName ==> git push origin :tagName` --> delete tagName from the remote repository

MOVING BACK AND FORTH IN TIME (with log and reflog) - steps
---
###### we can move back and forth in time by either using the CHECKOUT command (it will simply move the HEAD to another commitId, so we can start a branch to that position), or with the RESET command, and in this case we will lose the reference to the last commitId and the HEAD will then be the new one.
1. MOVE BACK IN TIME
   1. We find the commitId we want to move back to `git log --oneline --graph --decorate --all`
   2. We reset the current head to that specific commit `git reset pastCommitId` --> now the new head will be commitId (we won't be able to reset the head to the original one directly)
   * !!!WE MADE A MISTAKE!!!! we would have had to point to a later commit not this early one
2. MOVE FORTH IN TIME
   1. We look for the correct commitId in the logs of references (is going to be available for up to 60 days MAX) `git reflog`
      ```
      this will prompt a list like
      e.g. commitId HEAD@{#} command: ouput 
      ```
   2. We reset the head where needed `git reset commitIdFromLogs ==> git reset HEAD@{#}` --> with #=N..0

RESETS - types of
---
1. `SOFT ==> git reset --soft commitId` --> simply point the header to a new position, it is not destructive
2. `(DEFAULT) MIXED ==> git reset commitId` --> reset the Staging Area and then reset the head where we wanted to
3. `HARD ==> git reset --hard commitId` --> reset both Working and Staging area and then reset the head where we wanted to

CHERRY PICK
---
###### Cherry picking is a particular type of merge: If we have 2 branches and we want to apply an hotfix from a specific previous commit we cherry pick the commit
###### Cherry pick bring a specific commit down where we need it to go
1. Like any other merge can produce merge conflicts
2. Normally used for small changes (is a commit-specific small merge)
3. `git cherry-pick commitId` --> it will apply to the current branch we are in the commitId and merge it automatically









### Note (the important one)
I do know I am a bit rubbish for when it comes for me to explaining things... and I'm not even sure this list can actually be of any help to anyone, however, just in case, I want to apologise in advance for any mistake I might have made either grammatical/logical/etc... :)


### Note 2 (the not important one)
Rebasing is fantastic however, in my opinion, like anything else in this life it shouldn't be abused :D

### Note 3 (the universal)
The answer is 42
