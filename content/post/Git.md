+++
date = '2026-01-02T13:14:30+08:00'
draft = false
title = 'Git的使用入门'

+++

# Git

### 安装配置

首先安装好git

安装好后设置好用户名和电子邮箱

```bash
git config --global user.name <username>
git config --global user.email <email>
```

查看帮助`git help [command]`

---



然后配置好github 的ssh

> ` $ ssh-keygen -t rsa -C <email>` 
>
> 然后将用户文件夹（Linux 为 `~` 文件夹，Windows 为 %USERPROFILE% 文件夹）内的 `ssh` 文件夹下的公钥（`id_rsa.pub`）上传到 GitHub 的“个人设置->SSH keys”中

---



### 从零建库并托管到Github

1. 初始化本地仓库 ` git init`
2. 在 GitHub 上创建一个新的仓库 
3. 将本地仓库与 GitHub 远程仓库关联  ` git remote add origin <Github repo URL>`
4. 将本地提交推送到Github远程仓库(~~先同步一下避免出现问题~~)  ` git push -u origin master`
5. 在本地修改后添加更改到Git暂存区  ` git add [options] <file>`
6. 提交文件到Git仓库 ` git commit -m "message"` 
7. 将本地提交推送到 GitHub 远程仓库  ` git push -u origin main`



### git常用命令



初始化仓库 ` git init`

添加更改到暂存区 ` git add`   **使用 `git add .` 添加所有文件**

提交文件到仓库 ` git commit -m"<message>"`

更改最近一次提交信息 ` git commit --amend -m "message"`

将本地 Git 仓库提交到远程 Git 仓库 

- `git push <url>`
- ` git push origin` 
- `git push -u origin <branchname>`

拉取远程仓库 `url` 的 `<branch>` 分支的更改，并 merge 到当前分支上`git pull <url> <branch> `

若不指定 `<branch>`，则为远程仓库的默认分支

---

查看提交历史 ` git log`

查看当前仓库状态 ` git status`

查看分支 ` git branch`

查看目前所储存的所有远程仓库链接 `git remote`

- `git remote get-url <name>`：查看 `<name>` 所代表的链接
- `git remote set-url <name>`：重新设置 `<name>` 所代表的链接
- `git remote remove <name>`：删除该链接

---

将分支 `<branch>` 的提交合并到本分支 `git merge <branch>`：

在暂存区中删除一个文件 `git rm <file>`

---

比较文件差异  ` git diff`

- `git diff <filename>`：比较当前目录中文件与暂存区中文件的差异（若不指定文件名则显示所有有差异的文件的差异）
- `git diff <commit> <filename>`：比较当前目录中文件与某个历史提交的差异
- `git diff <commit> <commit> <filename>`：比较两个历史提交中某文件的差异

---

撤销操作  ` git reset`

- `git reset <commit>`：将 Git 仓库的 `HEAD` 回退到某版本，清空暂存区，但是不修改目录中的文件
- `git reset <commit> <filename>`：将暂存区中内容回退到某版本，但不修改 Git 仓库的 `HEAD` 和目录中的文件
- `git commit <commit> --hard`：除了会将 Git 仓库回退到某版本，还会把目录内的所有文件都改为该版本的内容

---

创建与切换分支

- `git branch <name>`：创建一个新的分支，并命名为 `<name>`
- `git checkout <name>`：切换到名称为 `<name>` 的分支
- `git checkout -b <name>`：创建分支 `<name>` 并切换到该分支。相当于 `git branch <name> && git checkout <name>`

变基 `git rebase <branch>`：让当前分支以 `<branch>` 为基进行变基

---













 







  













