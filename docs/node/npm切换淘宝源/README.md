# npm切换淘宝源

npm默认下载用的是国外源，我们可以切换成淘宝源。



1. 安装cnpm

   ```
   npm i -g cnpm --registry=https://registry.npm.taobao.org
   ```

2. 使用cnpm下载依赖

   原先我们下载是这样子的

   ```
   npm install -g commitizen
   ```

   使用淘宝源，我们这样下载

   ```
   cnpm install -g commitizen
   ```



我们也可以临时使用淘宝镜像下载，而不需要下载cnpm

```
npm i -g commitizen --registry https://registry.npm.taobao.org
```

