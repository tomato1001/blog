---
layout: page
title: gulp-bower-sass开发配置
categories: ["gulp", "bower", "sass"]
date: 2015-07-02 19:35:52
---
# gulp bower sass前端开发配置

## bower
基于node、npm、git的web前端资源管理工具，简化并统一前端资源管理。

前端开发经常需要集成大量的第三方库或者组件，按照以前的集成方式，开发者需要到对应的网站上下载包，然后将这些包手动的集成到项目中。当第三方库的依赖比较少时，该方式还能应付自如。但是，当依赖越来越多，而且需要对每个依赖的版本进行维护时(比如，升级依赖库版本)，手动管理的弊端就体现出来了，需要重新下载包，然后复制、粘贴。

使用`bower`将每个第三个库作为依赖添加到`bower`的配置文件，从而让`bower`来对资源进行统一管理，安装和升级只需要敲一下命令就可以解放我们的双手，从而达到自动化管理的目标.

### 安装

``` sh
npm install -g bower
```

bower依赖***node***、***npm***及***git***，安装之前先确认这些工具是否已经安装

### 使用
创建***bower.json***

``` sh
bower init
```

***bower.json***文件将纪录所有的依赖及项目配置信息

### 安装包
通过bower安装的包默认在***bower_components/***目录下

``` sh
bower install <jquery>
```

### 安装并将包加入到bower.json文件中

``` sh 
bower install <jquery> --save
```
	
通过git名称、git url或者url进行安装

{% highlight sh %}
# 指定包名
bower install jquery
# github资源库名称
bower install desandro/masonry
# github url
bower install git://github.com/user/package.git
# url
bower install http://example.com/script.js
{% endhighlight %}

### 安装jquery、bootstrap-sass、font-awesome

``` sh
bower install --save jquery bootstrap-sass components-font-awesome
```

### 配置代理
`bower`的包都是从github中下载，通过代理可以加快下载速度。在~/.bowerrc中加入以下配置

``` js
{
	"proxy" : "http:xxxxxx:port"
}
```

## gulp

``` sh 
npm install gulp
```

## 集成

### 创建项目

``` sh
npm init
```

项目结构如下:
<div style="width: 200px;text-align: center;position: relative; left: 230px;">
	<img src="{{'/images/gulp-bower-sass-project-structual.png' | prepend: site.baseurl}}" alt="">
</div>
<br>	

### 安装依赖
	
``` sh 
npm install --save-dev gulp gulp-util gulp-sass gulp-sourcemaps gulp-watch underscore
```

### 创建`gulpfile.js`并加入以下代码
``` js
var gulp = require('gulp'),
	sass = require('gulp-sass'),
	sourcemaps = require('gulp-sourcemaps'),
	gutil = require('gulp-util'),
	_ = require('underscore');

var config = {
	bowerPath : './bower_components/',
	appPath : './app/'
};

gulp.task('default', ['css', 'js', 'fonts', 'watch']);

var includeSassPaths = ['src/scss'].concat(
		autoprefixString([
			'bootstrap-sass/assets/stylesheets',
			'components-font-awesome/scss'
		], config.bowerPath)
);

//
// {outputFolder : '', type: 'css', debug : false, src : ''}
function build (opts) {
	var stream = gulp.src(opts.src);
	if (opts.debug) {
		stream = stream.pipe(sourcemaps.init())
	}

	if (opts.type == 'css') {
		stream = stream.pipe(
			sass({
				includePaths : includeSassPaths
			}).on('error', sass.logError)
		);
	}

	if (opts.debug) {
		stream = stream.pipe(sourcemaps.write('./'))
	}

	stream.pipe(gulp.dest(opts.outputFolder, {cwd : config.appPath}));
};


var fonts = autoprefixString([
		'components-font-awesome/fonts/*',
		'bootstrap-sass/assets/fonts/bootstrap/*'
		], config.bowerPath);

gulp.task('fonts', function() {
	build({
		src : fonts,
		outputFolder: 'fonts'
	});
});

gulp.task('css', function() {
	build({
		type: 'css',
		debug: true,
		src : 'src/scss/**/*.scss',
		outputFolder: 'css'
	});
});

var jss = autoprefixString([
		'bootstrap-sass/assets/javascripts/bootstrap.js',
		'jquery/dist/jquery.js'
	], config.bowerPath);

gulp.task('js', function() {
	build({
		type: 'js',
		debug: true,
		src : jss,
		outputFolder: 'js'
	});
});

gulp.task('watch', function() {
	gulp.watch(['src/**/*'], function(e) {
		var s = new Date().getTime();
		if (e.path.lastIndexOf('.scss')) {
			build({
				type: 'css',
				debug: true,
				src : e.path,
				outputFolder: 'css'
			});
		} else if (e.path.lastIndexOf('.js')) {
			build({
				type: 'js',
				debug: true,
				src : e.path,
				outputFolder: 'js'				
			});
		}
		var e = new Date().getTime();
		gutil.log('Rebuild complete after ' + (e - s) + 'ms');
	});
});


function autoprefixString (v, prefix) {
	if (typeof v == 'string') {
		return prefix + v;
	}

	if (_.isArray(v)) {
		return _.map(v, function(value) {
			return prefix + value;
		});
	}
}	
```

### 运行
在项目根目录下运行`gulp`

* 自动编译`src/scss`下的文件并输出到`app/css`

* 将定义的`js`依赖文件(jss变量)复制到`app/js`

* 将字体文件复制到`app/fonts`

* 自动检测scss、js文件的修改并自定发布





