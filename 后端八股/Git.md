##### 基础操作
```
git clone  # 克隆仓库
git pull   # 拉取最新代码
git add    # 添加到暂存区
git commit # 提交
git push   # 推送
```

###### 分支操作
```
git branch    # 查看/创建分支
git checkout  # 切换分支
git switch    # 切换分支
git merge     # 合并分支
git rebase    # 变基
```
- mergre和rebase之间的区别？
	- merge：合并两个分支的历史记录，保留分支合并的轨迹
	- rebase：将一个分支的提交重新应用到另一个分支上，不保留分支合并的轨迹。这个操作会重新写历史

##### 查看/回退操作
```
git log        # 查看历史
git status     # 查看状态
git diff       # 查看差异
git reset      # 回退版本
```