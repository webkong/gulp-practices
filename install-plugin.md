# 实践第一步：装插件 {#webkong}


我们使用工具来实现特定的自动化功能，那么我们的需求：

* 检查Javascript
* 编译Sass（或Less之类的）文件
* 合并Javascript
* 压缩并重命名合并后的Javascript

我们去gulp的[主站](http://gulpjs.com/plugins/)找这些插件，也可以去[npm的官网](https://www.npmjs.com/)进行查找

之后的命令以cnpm运行。


```
mkdir test;cd test
```
创建并进入test文件夹

```
cnpm install gulp-jshint gulp-sass gulp-concat gulp-uglify gulp-rename gulp-htmlmin gulp-imagemin gulp-util --save-dev 
```
安装完成后，可以运行 `cnpm init` 来创建一个package.json文件，来管理安装的依赖包。也可以在安装以来之前执行。

目前test文件夹的目录结构

```
.
├── gulpfile.js //gulp的配置文件
├── node_modules //npm安装的依赖包文件
└── package.json 
```
接下里具体配置gulp。

