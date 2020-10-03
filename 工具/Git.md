### 本地管理

git config --global user.email设置邮箱

git config --global user.name设置用户名，在进行推送的时候便可以查看到推送人

在工作目录下执行`git init`初始化本地工作区

- 添加文件

编辑好文件后，`git add xxx`将文件添加至暂存区，`git commit -m "说明"`提交至本地仓库

- 删除文件

`rm xxx`删除工作区文件，`git rm xxx`删除仓库文件（该步修改提交至暂存区），`git commit -m "说明"`提交本地仓库

- 版本回退

`git log/reflog`查看历史提交版本，通过`git reset --hard HEAD^`回退至倒数第二个版本，并还原工作目录，也可以将HEAD改为版本号前5位，git自动查找对应版本。注意，如果只想恢复某一版本中的某一个文件，使用`git checkout 版本号 xxx`，并手动commit

- 管理修改

`git status`查看目前状态，红字代表工作区修改，黄字代表暂存区修改。`git restore xxx`恢复工作区，`git restore --staged xxx`恢复暂存区。`git diff HEAD -- xxx`查看工作区与本地仓库文件最新版本区别。

- 工作区/暂存区/本地仓库

![image-20200811223122293](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200811223122293.png)

任何修改都是先修改暂存区，再利用commit提交至本地仓库

### 远程仓库

- 工作准备

执行`ssh-keygen -t rsa -C "youremail@example.com"`在user目录下生成公私钥，将公钥信息拷贝至github下add sshkey中，此后，针对该账号下的远程仓库的push/poll将会验证该身份信息。

- 添加仓库

`git remote add origin xxx`xxx代表远程仓库的ssh地址，`git remote`查看远程仓库

- 推送本地仓库至远程

`git push -u origin master`，-u代表将本地master与远程master关联（第一次关联仓库时使用）

- 拉取远程仓库至本地

`git clone xxx`xxx代表远程仓库的ssh地址

- 获取远程最新分支，`git fetch`，获取并融合 `git pull`

### 分支管理

- 创建分支 `git branch xxx`
- 查看分支 `git branch`
- 切换分支 `git switch xxx`
- 合并分支 `git merge xxx`
- 删除分支 `git branch -d xxx`
- 克隆其他分支完整commit `git reset --hard xxx`
- 获取其他分支commit的改变 `git cherry-pick xxx`
- 获取其他分支某个文件 `git checkout commit-id xxx`，需要手动commit

### 冲突管理

当执行merge操作时，有可能发生文件冲突，原因在于master分支和其他分支某个文件可能都做了修改，因此在执行merge操作时，会提示冲突的文件，并建议手动修复冲突，并重新提交该文件。`git merge --abort`取消本次合并

![image-20200811223108490](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200811223108490.png)

注意，只要两分支已经产生了演进，就会产生冲突，合并的时候无法使用fastforward模式，仅且仅当分支①是分支②的祖先结点时，才能快速合并，即不会产生冲突。

### 参考一篇文章

[实际项目中如何使用Git做分支管理](https://blog.csdn.net/ShuSheng0007/article/details/80791849)

