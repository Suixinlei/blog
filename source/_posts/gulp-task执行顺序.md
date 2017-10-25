---
title: 用Promise说明gulp task执行顺序
date: 2017-10-25 11:19:43
tags:
---

前一段时间在忙组件库编译的事情，一直在用gulp做任务执行，因为每一个任务都是异步且耗时较长，于是就研究了下gulp的执行顺序。

```
const gulp = require('gulp');

// 传入一个回调函数，因此引擎可以知道何时它会被完成
gulp.task('one', function(cb) {
    // 做一些事 -- 异步的或者其他任何的事
    cb(err); // 如果 err 不是 null 和 undefined，流程会被结束掉，'two' 不会被执行
});

// 标注一个依赖，依赖的任务必须在这个任务开始之前被完成
gulp.task('two', ['one'], function() {
    // 现在任务 'one' 已经完成了
});

gulp.task('default', ['one', 'two']);
// 也可以这么写：gulp.task('default', ['two']);
```

对于上面的示例，要执行的是default, 其依赖的任务为'one', 'two'。那么实际的执行顺序是(参照Promise): `Promise.all([one, two]).then(default)`。也就是说'one' 和 'two' 是并发执行的。

那么问题来了，对于'two'来说，'one'是依赖，那么实际的运行结构则是`Promise.all([one]).then(two).then(default)`。这样就比较清楚了吧。那么下面再多一个小练习，看看最终答案和你想的是否一样呢？

```
const gulp = require('gulp');
const del = require('del');

gulp.task('clean', (cb) => {
  del.sync(['build', 'lib']);
  cb();
});

gulp.task('build1', ['clean'], (cb) => {
  // do some thing
  cb()
});

gulp.task('build2', ['clean'], (cb) => {
  // do some thing
  cb()
});

gulp.task('build3', ['clean'], (cb) => {
  // do some thing
  cb()
});

gulp.task('afterbuild', ['build1', 'build2', 'build3'], (cb) => {
  // do some thing
  cb();
})

gulp.task('build', ['clean', 'build1', 'build2', 'afterbuild']);
```

对的，结果就是 
  1. 'clean'
  2. 'build1', 'build2', 'build3'
  3. 'afterbuild'

而且 clean 只会执行一次， 因为gulp会自动去重哦~