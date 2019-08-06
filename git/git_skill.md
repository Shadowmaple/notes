# git命令
```
git log

git branch  bugFix      # 创建分支bugFix
git checkout bugFix     # 转到bugFix分支
git branch -b bugFix    # 创建并转到bugFix分支
git branch bugFix master^      # 在master分支的父节点上创建分支bugFix

git merge bugFix        # 将bugFix分支合并到当前分支
git rebase bugFix       # 把当前分支追加到bugFix分支，同时指向该分支
git rebase side bugFix  # 把bugFix分支追加到side分支

git checkout c1         # 将HEAD指向c1提交记录
git checkout master^    # 将HEAD指向master的父节点（上一个提交）
git checkout -f master HEAD~3       # 将master强制转到HEAD指向的前3级父节点

git reset HEAD^         # 撤销当前记录，回退到上一次提交，但这只是本地的操作，不便于远程合作
git revert c2           # 重新建立一次上次状态的提交，即远程回退到前一提交

git cherry-pick c2 c4 c5    # 将记录c2 c4 c5追加到当前分支上
git cherry-pick bugFix      # 将bugFix分支上的当前（一个）记录追加到当前分支上

git rebase -i HEAD~4        # 开启含前四个节点，交互式界面（文档），可用于调整节点顺序，删除等操作

git commit --amend          # 修改提交信息

# 标签标注的节点不能改变，只是作为彼状态的标注作用
git tag v1 c1           # 在c1节点上建立v1标签，默认则为HEAD节点
git describe <branch>   # 给出（某个分支）最近的标签

get pull == get fetch + git merge <...>

git checkout -b foo origin/master # 设定跟踪远程origin/master分支的是本地foo分支，会自动创建foo分支
git branch -u origin/master foo         # 同上

git push origin <place>
git push origin <source>:<place>
git push origin :<place>            # 会删除远程origin的<place>分支

git pull origin <place>             # 拉取远程的<place>分支或节点并与当前分支合并
git pull origin <source>:<place>
git pull origin :<place>            # 会在本地新建一个<place>分支
```
Rebase

优点:
Rebase 使你的提交树变得很干净, 所有的提交都在一条线上

缺点:
Rebase 修改了提交树的历史
比如, 提交 C1 可以被 rebase 到 C3 之后。这看起来 C1 中的工作是在 C3 之后进行的，但实际上是在 C3 之前。

