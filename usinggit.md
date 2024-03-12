### Using Git tips and learning 
- Using git 
- You can create an account with github 
- how to clone a repo
```sh
# Using github client 
gh repo clone [name of the repo]

#Using git client 

git clone [Url of the Repo]

```
- Check if there a files to be committed 
```sh 
git status 
```
- if there are files that are changed and needs to be commmitted. 
```sh 
git add [Name of the file to add]
```
- Committing the file 
```sh 
# -m means to add the message to the commit with the message in "".
git commit -m "A message about the commit"
```
- Viewing the log of your commits or history

```sh
git log --all --decorate --oneline -graph
```
- Viewing the log of the commit with the graph 
```sh
git log --all --oneline --decorate --graph 
```

- Pushing to the git server 
```sh 
git push 
```
- Using the new files from the server 
```sh 
git pull 
``` 
- Using get fetch to get the file not pull
```sh 
git fetch
```
- Using the merge to merge the fetch file 
```sh 
git merge orgin 
```
- Adding the all the files and commit 
```sh 
git commit -a -m "message"
```
--------------------------------------------
- create a local directory then run the git init 
- git init created a new repo 
```sh
git init # this is in the new folder 
```
- Going back to a previous version 
```sh 
git rest --hard [git_id]
```
- Getting the commits of local repos 
```sh
git reflog
```
- Going back a point of time 
```sh
git rest --hard [git_id]
```
- Revert will remove what happen in a commit 
```sh
git revert [git_id] 
```

- Pushing to a remote repo 
```sh 
git remote add origin [URL_REMOTE_REPO]
git push -u orign main 
```
-------------------------------------------
### Using Branches with Git 
- Creating a Branches 
- Playing with the code in a branch 
- The -c : create a branch that is followed
```sh
git switch -c dev
```
- Switching to a branch 
```sh 
git switch main 
```
- Murging the branches 
```sh
git merge dev
```
------------------------------------------------
### Rebase Branchs 
- Using the rebase 
- Align the Branches 
```sh
git rebase
```

### Working with Conflics 
- Open the file conflic and fix it with checking and making changes. 
- Using squash 
```sh 
git rebase -i [git_id]
# using the squash 
push [somegit_id] comment
push [somegit_id] comment
squash [somegit_id] comment
squash [somegit_id] comment 
```




