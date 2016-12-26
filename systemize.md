# 实践第四部：整理和优化 {#webkong}

>首先梳理一下需求，并期待这样的目录结构。

自动化需求

* 编译sass/less、压缩、合并
* 编译、校验、压缩、合并JavaScript
* 压缩图片，可能情况下生成雪碧图
* 压缩html
* 给引用文件加hash
* 在本地启服务调试
* 只编译修改的文件，编译后自动刷新页面显示
* 文件统一按目录整理

期望的目录结构

```
├── config.json
├── dev
│   ├── css
│   ├── images
│   ├── index.html
│   ├── js
│   ├── result.html
│   ├── show.html
│   └── static
├── gulpfile.js
├── node_modules
├── package.json
├── public
│   ├── css
│   ├── images
│   ├── index.html
│   ├── js
│   ├── result.html
│   ├── rev-manifest.json
│   ├── show.html
│   └── static
└── src
    ├── css
    ├── html
    ├── images
    ├── js
    └── static

```

在代码开发阶段，可以用`gulp dev`命令还生成测试代码，上线过程可以用`gulp build`带压缩和优化代码到public目录。
还可以监视修改，自动更新修改文件到dev。


代码：

```
// 导入文件结构配置

var dir = require('./config.json');

//导入依赖

var gulp = require('gulp'),
    util = require('gulp-util'),
    jshint = require('gulp-jshint'),//校验js
    sass = require('gulp-sass'),// sass编译
    cleanCSS = require('gulp-clean-css'),// css压缩
    autoprefixer = require('gulp-autoprefixer'),//给css加前缀
    htmlmin = require('gulp-htmlmin'),// 压缩html
    imagemin = require('gulp-imagemin'),//图片压缩
    uglify = require('gulp-uglify'),//压缩js
    babel = require('gulp-babel'),//es6
    concat = require('gulp-concat'),//合并
    rename = require('gulp-rename'),//重命名
    sourcemaps = require('gulp-sourcemaps'),//生成sourcemap
    rev = require('gulp-rev'),//hash文件名
    revReplace = require('gulp-rev-replace'),//替换html中依赖文件
    clean = require('gulp-clean'),//删除文件或文件夹
    flatten = require('gulp-flatten'),//移动文件
    clone = require('gulp-clone'),//移动文件
    connect = require('gulp-connect'),//本地服务
    watch = require('gulp-watch');//自动监听

//clean

gulp.task('clean',function(){
    return gulp.src(dir.dev.root,{read:false})
        .pipe(clean());
});
gulp.task('cleanall',function(){
    return gulp.src([dir.dev.root,dir.public.root],{read:false})
        .pipe(clean());
});


//处理css 

gulp.task('acss', function() {
    return gulp.src(dir.src.css)
        .pipe(sourcemaps.init())
        .pipe(sass())
        .pipe(sourcemaps.write('./map/'))
        .pipe(gulp.dest(dir.dev.css))
        .pipe(connect.reload());
});

gulp.task('bcss', ['acss'] ,function() {
    util.log('autofixer and minify css');
    return gulp.src(dir.dev.css + '**/*.css')
        .pipe(autoprefixer({ //添加前缀
            browsers: ['last 2 versions'],
            cascade: false
        }))
        .pipe(cleanCSS())
        .pipe(gulp.dest(dir.public.css))
});

//处理js

gulp.task('ajs', function() {
    return gulp.src(dir.src.js + '*.js')
        .pipe(babel({
        	presets:['es2015']
        }))
        .pipe(concat('all.js'))
        .pipe(gulp.dest(dir.dev.js))
        .pipe(connect.reload());
});

gulp.task('bjs', ['ajs'],function(){
	return gulp.src(dir.dev.js+'**/*.js')
        .pipe(sourcemaps.init())
	  	.pipe(uglify())
        .pipe(sourcemaps.write('./map/'))
	  	.pipe(gulp.dest(dir.public.js));
});

//校验js
gulp.task('lint',function(){
    return gulp.src(dir.src.js + '/*.js')
        .pipe(jshint({progressive: true}))
        .pipe(jshint.reporter('default'));
});

//处理图片(一般只是move)

gulp.task('aimg',function(){
    return gulp.src(dir.src.image)
            .pipe(clone())
            .pipe(gulp.dest(dir.dev.image))
            .pipe(connect.reload());
});

gulp.task('bimg',function(){
    return gulp.src(dir.src.image)
            .pipe(imagemin())
            .pipe(gulp.dest(dir.public.image))
});

//移动静态资源
gulp.task('clone',function(){
    return gulp.src(dir.src.static+'**')
            .pipe(clone())
            .pipe(gulp.dest(dir.public.static))
            .pipe(gulp.dest(dir.dev.static))
            .pipe(connect.reload());
});

//html
gulp.task('ahtml',function(){
    gulp.src(dir.src.html)
      .pipe(clone())
      .pipe(gulp.dest(dir.dev.root))
      //.pipe(gulp.dest(dir.public.root))
});


gulp.task("revision", ["bcss", "bjs"], function(){
  return gulp.src([dir.dev.root+'**/*.css', dir.dev.root+'**/*.js'])
    .pipe(rev())
    .pipe(gulp.dest(dir.public.root))
    .pipe(rev.manifest())
    .pipe(gulp.dest(dir.public.root))
})

gulp.task("revreplace", ["revision"], function(){
  var manifest = gulp.src("./" + 'public' + "/rev-manifest.json");

  return gulp.src(dir.src.html)
    .pipe(revReplace({manifest: manifest}))
    .pipe(gulp.dest('public'))
});


//服务器

gulp.task('dev',['acss','ajs','aimg','clone','ahtml'],function(){
    util.log('------success dev------');
});

gulp.task('build',['bimg','revreplace'],function(){
    util.log('------success build------');
});

//监视器

gulp.task('watchdev',['dev'],function(){
    gulp.watch(dir.src.css, ['acss'])
    .on('change', function(event) {
      console.log('css-- ' + event.path + ' was ' + event.type + ', running tasks acss');
    });
    gulp.watch(dir.src.js, ['ajs'])
    .on('change', function(event) {
      console.log('js--- ' + event.path + ' was ' + event.type + ', running tasks ajs');
    });
    gulp.watch(dir.src.image, ['aimg'])
    .on('change', function(event) {
      console.log('images--- ' + event.path + ' was ' + event.type + ', running tasks aimg');
    });
    gulp.watch(dir.src.static, ['clone'])
    .on('change', function(event) {
      console.log('static\'s file--- ' + event.path + ' was ' + event.type + ', running tasks clone');
    });
    gulp.watch(dir.src.html, ['ahtml'])
    .on('change', function(event) {
      console.log('html--- ' + event.path + ' was ' + event.type + ', running tasks ahtml');
    });
});

//本地服务

gulp.task('serverDev',function(){
    connect.server({
        name: 'Dev',
        root: ['dev'],
        port: 9999,
        livereload: true
      });
});
gulp.task('serverPublic',function(){
    connect.server({
        name: 'Public',
        root: ['public'],
        port: 12121,
        livereload: true
      });
});


gulp.task('s',['serverDev','watchdev']);
gulp.task('sp',['serverPublic']);


```