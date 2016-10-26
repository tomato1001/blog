---
layout: post
title: gulp browserify babel react sass完整开发流程
---
# 简介

## gulp

基于task的自动化构建工具，功能与grunt类似，使用方式比grunt更简单。

## babel

es6语法库,以前的名字叫`es6to5`,看名字就能知道它的功能了.通过babel可以使用es6的语法编写javascript,然后将文件转换为es5语法,从而解决了兼容性的问题.

## react

facebook与Instagram开源的基于组件的方式创建ui的javascript库,属于MVC中的V.

# 安装

``` sh
npm install --save-dev browserify babelify browserify-shim gulp gulp-sass gulp-sourcemaps gulp-util gulp-webserver lodash.assign vinyl-buffer vinyl-source-stream watchify

npm install --save bootstrap font-awesome jquery react
```

# package.json

``` json
{
  "name": "bgbb",
  "version": "1.0.0",
  "description": "browserify-gulp-bower-babel integration",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "browser": {
    "jquery": "./node_modules/jquery/dist/jquery.js"
  },
  "browserify-shim": {
    "jquery": "global:jQuery"
  },
  "browserify": {
    "transform": [
      "browserify-shim",
      [
        "babelify",
        {
          "ignore": "/node_modules/",
          "extensions" : [".js", ".jsx"]
        }
      ]
    ]
  },
  "devDependencies": {
    "babelify": "^6.1.2",
    "browserify": "^10.2.4",
    "browserify-shim": "^3.8.9",
    "gulp": "^3.9.0",
    "gulp-sass": "^2.0.3",
    "gulp-sourcemaps": "^1.5.2",
    "gulp-util": "^3.0.6",
    "gulp-webserver": "^0.9.1",
    "lodash.assign": "^3.2.0",
    "vinyl-buffer": "^1.0.0",
    "vinyl-source-stream": "^1.1.0",
    "watchify": "^3.2.3"
  },
  "dependencies": {
    "bootstrap": "^3.3.5",
    "font-awesome": "^4.3.0",
    "jquery": "^2.1.4",
    "react": "^0.13.3"
  }
}
```

# gulpfile.js

``` js
var gulp = require('gulp'),
    gutil = require('gulp-util'),
    sourcemaps = require('gulp-sourcemaps'),
    sass = require('gulp-sass'),
    source = require('vinyl-source-stream'),
    buffer = require('vinyl-buffer'),
    browserify = require('browserify'),
    watchify = require('watchify'),
    babelify = require('babelify'),
    assign = require('lodash.assign'),
    webserver = require('gulp-webserver');

gulp.task('default', ['assets', 'sass', 'js']);

gulp.task('dev', ['watch', 'serve']);

gulp.task('build', ['default']);

gulp.task('watch', function() {
	watchifyJS('./src/app.js', 'app.js', true);
	gulp.watch('./src/**/*.scss', function(evt) {
		buildTimeStatistic(buildSass);
	});
});

gulp.task('serve', function() {
	gulp.src('./app')
	.pipe(webserver({
	  // livereload: true,
	  port : 9200,
	  fallback : 'index.html'
	}));	
});

gulp.task('js', function() {
    watchifyJS('./src/app.js', 'app.js');
});

gulp.task('assets', function() {
	// boostrap
	buildBootstrap();

	// jquery
    gulp.src('./node_modules/jquery/dist/jquery.js')
    	.pipe(sourcemaps.init({loadMaps : true}))
    	.pipe(sourcemaps.write('./'))
    	.pipe(gulp.dest('./app/lib/jquery'));

    // font-awesome
    gulp.src('./node_modules/font-awesome/css/font-awesome.css')
    	.pipe(sourcemaps.init({loadMaps: true}))
    	.pipe(sourcemaps.write('./'))
    	.pipe(gulp.dest('./app/lib/font-awesome/css'));
    gulp.src('./node_modules/font-awesome/fonts/*')
    	.pipe(gulp.dest('./app/lib/font-awesome/fonts'));
});

/**
 * Build and package bootstrap.
 */
function buildBootstrap () {
	var basePath = './node_modules/bootstrap/dist/',
		outFolder = './app/lib/bootstrap/';

    gulp.src([basePath + 'css/bootstrap.css', basePath + 'css/bootstrap-theme.css'])
    .pipe(sourcemaps.init({loadMaps : true}))
    .pipe(sourcemaps.write('./'))
    .pipe(gulp.dest(outFolder + 'css'));

    gulp.src(basePath + 'js/bootstrap.js', {base: basePath})
    	.pipe(gulp.dest(outFolder));

    gulp.src(basePath + 'fonts/*').pipe(gulp.dest(outFolder + 'fonts'));
}

gulp.task('sass', function() {
	buildSass();
});

function buildSass() {
    gulp.src('src/app.scss')
    	.pipe(sourcemaps.init())
    	.pipe(sass().on('error', sass.logError))
    	.pipe(sourcemaps.write('./'))
    	.pipe(gulp.dest('./app/css'));
}

function buildTimeStatistic (f) {
	var s = new Date().getTime();
	f(arguments[1]);
	var e = new Date().getTime();
	gutil.log('Rebuild complete after ' + (e - s) + 'ms');
}

function watchifyJS(input, out, enableWatchify) {
    var customOpts = {
        entries: input,
        debug: true
    };
    var opts = assign({}, watchify.args, customOpts);
    var b = enableWatchify ? watchify(browserify(opts)) : browserify(opts);
    bundle(b, out);
    if (enableWatchify) {
        b.on('update', function() {
            bundle(b, out);
        }); // on any dep update, runs the bundler		
    };
    b.on('log', gutil.log); // output build logs to terminal
}

function bundle(b, out) {
    return b.bundle()
        // log errors if they happen
        .on('error', gutil.log.bind(gutil, 'Browserify Error')).pipe(source(out))
        // optional, remove if you don't need to buffer file contents
        .pipe(buffer())
        // optional, remove if you dont want sourcemaps
        .pipe(sourcemaps.init({
            loadMaps: true
        })) // loads map from browserify file
        // Add transformation tasks to the pipeline here.
        .pipe(sourcemaps.write('./')) // writes .map file
        .pipe(gulp.dest('./app/js'));
}
```

# 使用

``` sh
gulp # 编译scss、js并将bootstrap、jquery、font-awesome编译到`app/lib`下

gulp build # 与gulp功能相同

gulp dev # 在执行默认的task上增加watch功能,支持自动编译修改文件
```



