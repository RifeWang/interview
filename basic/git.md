# GIT

工作区 > 暂存区 > git 仓库

- `git merge` / `git rebase`
- `git reset` / `git revert`
- `git pull` / `git fetch` (`git pull` = `git fetch` + `git merge`)
- `git reflog`: 查看所有分支的所有操作记录

- `git init`
- `git add <filename | .>` : 提交到暂存区
- `git rm --cached <filename>` : 从暂存区中移除，保留工作区
- `git rm <filename>` : 从暂存区以及工作区中删除文件
- `git commit -m <message>`
- `git diff --cached` : 不带 --cached 只能看到已经提交的改变
- `git commit -a -m <msg>` : 一步完成 add & commit ，但不能针对新文件
- `git commit --amend` : 修改提交

- `git stash` : 将当前工作区未提交的部分临时存储
- `git stash list` :
- `git stash pop` :
- `git stash drop` :


- `git branch -f master HEAD~3` : 将 master 分支强制指向 HEAD 的前三个提交

- `git cherry-pick <commit>...` : 将 commit 内容复制到当前分支上
- `git rebase <branch> <branch target，默认当前所在分支>` : 将 branch 变基到目标分支


history:
- `git log` : 查看提交记录，只能查看当前提交之前的记录（commit id, author, date, commit message）
- `git log -p` : 查看提交记录每一步的完整 diff
- `git log --stat --summary` : 概览提交记录

- `git reset --hard <commit id>` : HEAD 回退到指定的 commit id
- `git reflog` : 可以查看完整的历史操作

- `git remote add origin <url>` : 关联远程库


branch:
- `git branch` : 查看所有分支
- `git branch <new branch name>` : 创建一个新的分支
- `git checkout <branch name>` : 切换到目标分支
- `git checkout -b <new branch>` : 创建并切换到目标分支
- `git branch -d <branch name>` : 删除目标分支，但会确认目标分支内容已存在于当前分支
- `git branch -D <branch name>` : 强制删除目标分支
- `git branch <branch name> [commit]` : 创建分支并指向特定的提交


合并分支:
- `git merge <branch name>` : 将目标分支内容合并到当前分支，没有冲突则直接完成
- `git diff` : 如果有冲突，则查看分支不同的内容

<<<<<<< HEAD 和 ======= 之间的是我们当前分支的内容，为 ours
======= 和 >>>>>>> 之间的为 theirs
- `git checkout --ours <conflict-file-name>` : 采用当前内容，然后执行 `git add` , `--continue` 或者 `--skip`


强制覆盖本地:
- `git fetch --all`
- `git reset --hard origin/master`
- `git pull`

