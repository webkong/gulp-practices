# 实践第二步：使用和配置 {#webkong}


在gulpfile.js文件中进行配置

1.引入gulp

```
var gulp = require('gulp');
```

2.引入组件

```
···
var jshint = require('gulp-jshint');
var sass = require('gulp-sass');
var concat = require('gulp-concat');
var uglify = require('gulp-uglify');
var rename = require('gulp-rename');

···
```

3.创建task

强烈建议学习[官方的API](https://github.com/gulpjs/gulp/tree/master/docs)或者[中文API](http://www.gulpjs.com.cn/docs/api/)或者本文中的[API一章](/gulp-api.md)。

```js
// 检查脚本
gulp.task('lint', function() {
    gulp.src('./js/*.js')
        .pipe(jshint())
        .pipe(jshint.reporter('default'));
});

// 编译Sass
gulp.task('sass', function() {
    gulp.src('./scss/*.scss')
        .pipe(sass())
        .pipe(gulp.dest('./css'));
});

// 合并，压缩文件
gulp.task('jsmin', function() {
    gulp.src('./js/*.js') //通过
        .pipe(concat('all.js'))
        .pipe(gulp.dest('./dist'))
        .pipe(rename('all.min.js'))
        .pipe(uglify())
        .pipe(gulp.dest('./dist'));
});

//监视器 监视文件的变化进行task

gulp.task('watch',function(){
    var watcher = gulp.watch('src/assets/css/**',['sass']);
    var watcher2 = gulp.watch('src/assets/js/**',['jsmin']);

    watcher.on('change',function(event){
        console.log('File ' + event.path + ' was ' + event.type + ', running tasks...');
    });
    watcher2.on('change',function(event){
        console.log('File ' + event.path + ' was ' + event.type + ', running tasks...');
    });
});
```

> task 的写法和node的stream没有区别，都是通过pipe来链接和传递。
>
> 通过gulp.src来把文件变成stream，然后通过pipe在各处理程序（插件）中处理，最后通过gulp.dest来存储到指定位置。

目前就是整个gulp在基础使用中需要的配置。 但是这并不能满足我的所有需求，比如我要用ES6语法、或者在文件后面加hash后缀来避免缓存等。

下面进一步实践。

