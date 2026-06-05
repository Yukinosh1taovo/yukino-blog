+++
date = '2026-04-21T13:10:19+08:00'
draft = true
title = 'Linux'

+++

# Linux的使用

## 重设WSL中Linux系统用户的密码

以安装ubuntu22.04版本为例

在windows终端中输入`ubuntu2204 config --default-user root` 将wsl登录用户切换为root

在wsl系统中输入`passwd 用户名` 修改用户密码

再回到windows终端中输入`ubuntu2204 config --default-user yukino` 将wsl登录用户切换为原用户



或 管理员身份运行ps 输入`wsl.exe --user root`

 再输入`passwd root` 修改 root 密码

重置与用户密码与重置root密码类似





## 系统内置命令













## APT包管理器

更新软件清单 `sudo apt update`

更新系统所有软件 `sudo apt upgrade`

安装软件 `sudo apt install 软件名称`

卸载软件（保留配置文件） `sudo apt remove 软件名称 `

卸载软件（不保留配置文件） `sudo apt purge 软件名称`

清理不再需要的依赖包 `sudo apt autoremove`

搜索软件 `apt search 关键字`



## 性能监控htop

输入`htop`监控性能 按q退出监控



## Docker

查看当前**正在运行**的容器 `docker ps`  (加上-a 选项查看所有容器)

停止容器 `docker stop <容器id或名称>`

唤醒容器 `docker start <容器id或名称>`

删除容器 `docker rm <容器id或名称>` 或`docker rm -f <容器id或名称>` (强制删除 不用先停止再删除)

一键删除所有已经停止运行的容器 `docker container prune`

暴力清空：强行删除本地【所有】容器（无论是否在运行） `docker rm -f $(docker ps -aq)`



只执行 `docker rm` 只是删除了**运行实例（容器）**，下载下来的**基础镜像**（Image）依然会占用你的 WSL 虚拟硬盘空间



查看镜像列表和id `docker images`

彻底删除镜像 `docker rmi <镜像ID>`





