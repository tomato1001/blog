---
layout: page
title: gulp-browserify-watchify
categories: front-end gulp-browserify-watchfiy
date: 2015-06-21 18:00:00
---
# browserify watchify gulp集成

+ 安装依赖
	- npm install watchify browserify gulp vinyl-source-stream vinyl-buffer gulp-util gulp-sourcemaps \-\-save-dev
	- npm install lodash.assign \-\-save

``` js
'use strict';
var watchify = require('watchify');
var browserify = require('browserify');
var gulp = require('gulp');
var source = require('vinyl-source-stream');
var buffer = require('vinyl-buffer');
var gutil = require('gulp-util');
var sourcemaps = require('gulp-sourcemaps');
var assign = require('lodash.assign');


gulp.task('default', ['index', 'index2']);

gulp.task('index', function() {
	watchifyIndex('./src/index.js', 'bundle.js');
});

gulp.task('index2', function() {
	watchifyIndex('./src/index2.js', 'bundle2.js');
});

function watchifyIndex (input, output) {
	var customOpts = {
	  entries: input,
	  debug: true
	};
	var opts = assign({}, watchify.args, customOpts);
	var b = watchify(browserify(opts)); 
	bundle(b, output);
	// add transformations here
	// i.e. b.transform(coffeeify);

	b.on('update', function() {
		bundle(b, output);
	}); // on any dep update, runs the bundler

	b.on('log', gutil.log); // output build logs to terminal
}

function bundle(b, output) {
  return b.bundle()
    // log errors if they happen
    .on('error', gutil.log.bind(gutil, 'Browserify Error'))
    .pipe(source(output))
    // optional, remove if you don't need to buffer file contents
    .pipe(buffer())
    // optional, remove if you dont want sourcemaps
    .pipe(sourcemaps.init({loadMaps: true})) // loads map from browserify file
       // Add transformation tasks to the pipeline here.
    .pipe(sourcemaps.write('./')) // writes .map file
    .pipe(gulp.dest('./dist'));
}
```

运行`gulp index index2`或`gulp`之后，将自动检测index和index2文件的修改状态，当检测到修改之后,自动调用`browserify`打包输出到`dist`目录