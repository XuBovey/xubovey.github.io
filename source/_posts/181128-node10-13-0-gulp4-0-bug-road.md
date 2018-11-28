---
title: HEXO+NODE10.13+GULP4.0博客搭建过程中的坑
toc: true
date: 2018-11-28 18:05:06
categories: hexo,gulp
tags: hexo,gulp
---

## 前情
个人的静态日志网页有些日子没更新了。这两天想分享些东西上来。结果因为之前在七牛上的图床不让免费使用了，所以博客的头像也挂了，很是不爽。然后就有了下面的折腾。

## 尝鲜
奔着尝鲜的念头更新了node，直接到10.13.0版本，在之后的编写更新过程中无意执行了`npm audit fix --force`,结果把很多依赖给升级了，之后就悲剧了，本地可以执行`hexo server`，使用travis自动发布时却总是在gulp处出错，本地也试了确实压缩出问题，然后就是一顿不服，一顿尝试。最终在如下网站中获取帮助得以解决，为帮助其他踩坑者特写此篇。
<!---more--->
## 报错
```
assert.js:85
  throw new assert.AssertionError({
```
```
Uglifier complains with `Unexpected token: operator (>)` in 0.10.5
```

## 线索
```
Should be...

ES5 → https://www.npmjs.com/package/uglify-js → https://github.com/mishoo/UglifyJS2/blob/master/README.md
ES6+ (ES2015+) → https://www.npmjs.com/package/uglify-es → https://github.com/mishoo/UglifyJS2/blob/harmony/README.md
... but is currently ...

ES5 → https://www.npmjs.com/package/uglify-js → https://github.com/mishoo/UglifyJS2/blob/master/README.md
ES6+ (ES2015+) → https://www.npmjs.com/package/uglify-es → https://github.com/mishoo/UglifyJS2/blob/master/README.md
Cc: @isaacs ... is there a way to have npmjs.com read the harmony branch README.md correctly? Thanks for the visit in advance.
```

## 分析
1. 起初错误是gulpfile.js格式与gulp4.0不兼容，4.0的主要变更可[参考](http://web.jobbole.com/82992/)
2. 不支持ES6出现的`Unexpected token: operator (>)`错误

## 问题解决
### 不兼容问题解决
修改gulpfile.js修改后
```
var gulp = require('gulp');
var minifycss = require('gulp-minify-css');
var uglify = require('gulp-terser');
var htmlmin = require('gulp-htmlmin');
var htmlclean = require('gulp-htmlclean');
// 压缩css文件
gulp.task('minify-css', function() {
  return gulp.src('./public/**/*.css')
  .pipe(minifycss())
  .pipe(gulp.dest('./public'));
});
// 压缩html文件
gulp.task('minify-html', function() {
  return gulp.src('./public/**/*.html')
  .pipe(htmlclean())
  .pipe(htmlmin({
    removeComments: true,
    minifyJS: true,
    minifyCSS: true,
    minifyURLs: true,
  }))
  .pipe(gulp.dest('./public'))
});
// 压缩js文件
gulp.task('minify-js', function() {
  return gulp.src('./public/**/*.js')
  .pipe(uglify())
  .pipe(gulp.dest('./public'));
});
// 默认任务
// gulp.task('default', [
//   'minify-html','minify-css','minify-js'
// ]);
gulp.task('default', gulp.parallel(
  'minify-html','minify-css','minify-js'
));
```
主要将`var uglify = require('gulp-uglify');`变为`var uglify = require('gulp-terser');`  
默认任务的并行执行做了修改，由`gulp.parallel`实现。
为了使用gulp-terser还需要修改package.json文件中的`"gulp-uglify": "^3.0.1",`为`"gulp-terser": "^1.1.5",`

