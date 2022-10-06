# Git Learning

```shell
git remote add origin git@github.com:xsjlmzs/Rep.git # 添加远程仓库

git add . # 添加当前目录下所有文件
git reset [file]# 取消file的add
git status # 查看git 状态
git fetch # 获取remote最新的记录（不会更改本地分支）
git merge bugFix # 将bugFix分支和当前分支合并
git pull # 等同于 git pull + git merge 拉取remote分支并与本地分支合并
git push -u origin main# 推送本地分支main到远程origin仓库并与远程分支关联起来
# 从2021年11月开始，新项目github默认的主分支从master 变成了main，而在2021年之前创建的项目(老项目)，主分支仍使用master。
git remote set-url origin <remote-url> # 修改远程仓库地址
git show <commit-id> # 查看commit的内容
git reset --soft HEAD^ # 取消上一次commit操作，修改仍然在
git reset . # 取消add
git stash # 暂时存储修改
git stash list # 列出所有stash
git stash pop <stash_id> # 恢复stash的修改 

```

`HEAD` 总是指向当前分支上最近一次提交记录。

```shell
git clone -b dev xxx # clone指定分支dev到本地
```



## Git Setting

```shell
git config --global core.editor vim # 更改git的默认编辑器为vim
git config --global user.name xsjlmzs # 设置全局用户名
git config --global user.email 1498020741@qq.com # 设置全局email
ssh-keygen -t rsa -C 1498020741@qq.com # 生成ssh-key
```

```shell
git reset <file> # 取消add文件
git submodule add <url> <path> # 添加指定的url仓库作为submodule，其路径存在path里
git push origin hsj:hsj # 将本地分支hsj推送到远程分支hsj去，若没有则创建
```

