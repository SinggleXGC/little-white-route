# git新建远程分支

1. 新建一个本地分支

   ```git
   git checkout -b release
   ```

2. 将新建的本地分支push到远程服务器，就可以完成远程分支的建立

   ```git
   git push origin release:release
   ```

3. 查看远程仓库，就会发现release分支已经建立了