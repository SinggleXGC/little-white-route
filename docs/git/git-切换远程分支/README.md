# git 切换远程分支

> 转载自[git切换到指定远程分支](https://blog.csdn.net/astonishqft/article/details/83029490)



1. 使用`git pull`命令拉取最新远程信息

   

2. 查看远程所有分支

   ```git
   git branch -a
   ```

   `git branch`不带参数，列出本地已经存在的分支，并在当前分支的前面用`*`标记。加上参数`-a`，可以查看所有分支列表，包括本地和远程。远程分支会用红色字体标记出来。

   ```
   $ git branch -a
   * development
     master
     remotes/origin/HEAD -> origin/master
     remotes/origin/development
     remotes/origin/master
     remotes/origin/release-1.0.0
   ```

   

3. 新建分支并切换到远程分支

   ```git
   git checkout -b release origin/release-1.0.0
   ```

   > git checkout -b 本地分支名 origin/远程分支名

   该命令可以将远程git仓库的指定分支拉取到本地，并在本地新建了release分支，并和指定的远程分支release-1.0.0关联了起来