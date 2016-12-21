# 快速构建

开始构建Gulp环境。

> gulp是依托node来运行的，要提前安装好node。node的安装和使用请自行Google，包括npm的使用方式。

1.安装全局的gulp

```
npm install -g gulp //-g 是全局的意思，为--global的缩写 
```

2.如果项目的开发依赖安装

```
npm install --save-dev gulp //如果下载出现问题，自行谷歌cnpm
```

3.在项目根目录创建一个名为`gulpfile.js`的文件

```
var gulp = require('gulp');

gulp.task('default',function(){
    console.log('default')
});
```
4.运行gulp

```
gulp
```

这样就创建了一个gulp的管理项目。接下来使用它完成我们的需求。