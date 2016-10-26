---
layout: post
title: "requirejs"
categories: requirejs
date: 2015-09-06 16:23:51
---
Requirejs是一个javascript文件及AMD模块加载器。通过Requirejs可以让我们以依赖注入的方式来加载和管理模块。

# 实例

目录结构如下：

<div style="width: 133px;text-align: center;position: relative; left: 230px;">
	<img src="{{'/images/require-js-project-structual.png' | prepend: site.baseurl}}" alt="">
</div>
<br>

**js/common.js**

requirejs配置文件

{% highlight javascript %}
require.config({
	baseUrl: 'js/lib',
	paths: {
		app: '../app',
		jquery: 'jquery/jquery-1.11.2.min',
		'jquery.tipsy': 'jquery/tipsy/jquery.tipsy',
		'ymPrompt': 'prompt/ymPrompt',
		'underscore': 'underscore',
		'backbone': 'backbone/backbone',
		'marionette': 'backbone/backbone.marionette',
		'backbone-validation': 'backbone/backbone-validation'
	},

	// shim's key correspond to key of paths 
	shim: {
		'underscore': {
			exports: '_'
		},
		'backbone-validation': {
			deps: [
				'underscore',
				'jquery',
				'backbone'
			],
			exports: 'Backbone.Validation'
		},
		'jquery.tipsy': ['jquery'],
		'ymPrompt': {
			exports: 'ymPrompt',
			init: function() {
				this.ymPrompt.setDefaultCfg({closeBtn: false, maskAlpha: 0.75});
			}
		}
	}
});
{% endhighlight %}

> **paths**中各模块的路径相对于**baseUrl**
> shim用于集成非AMD模块编写的javascript

**js/page1.js**

主模块文件，用于data-main加载模块

{% highlight javascript %}
require(['./common'], function(common) {
	// execute below after common.js load complete

	require(['require', 'app/main1'], function(require) {
		var Main1 = require('app/main1');
		var m = new Main1();
		m.init();
	});
   
});
{% endhighlight %}

domReady是requirejs的插件，**domReady!**为requirejs插件读取语法，当dom文档加载完成后执行。

**js/app/main1.js**

{% highlight javascript %}
define([
	'domReady!',
	'ymPrompt',
	'jquery',
	'underscore', 
	'backbone',
	'marionette',
	'backbone-validation',
	'jquery.tipsy'
	], 
	function(doc, ymPrompt, $, _, Backbone, Marionette) {

		function Main () {
		}

		Main.prototype = {
			init : function() {
		        $("#example-1").tipsy();
		        $("#prompt").click(function() {
		            ymPrompt.succeedInfo({
		                message: 'Success'
		            });
		        });
				console.log($, _);
				console.log(Backbone.Wreqr);
			}
		}

		return Main;
});
{% endhighlight %}

**project.html**

{% highlight html %}
<!DOCTYPE html>
<html>
<head>
	<title>RequireJS</title>
	<link rel="stylesheet" type="text/css" href="js/lib/jquery/tipsy/tipsy.css">
	<link rel="stylesheet" type="text/css" href="js/lib/prompt/ymPrompt.css">
	<!-- data-main attribute tells require.js to load
             scripts/main.js after require.js loads. -->
	<script type="text/javascript" data-main="js/page1" src="js/lib/require.js"></script>
</head>
<body>
<h1>Tipsy</h1>
<p>
	<a id="example-1" href="#" original-title="Hello World">Hover me</a>
</p>
<h1>ymPrompt</h1>
<p>
	<a href="#" id="prompt">Prompt</a>
</p>
</body>
</html>
{% endhighlight %}

requirejs使用data-main加载主模块，主模块引用相关的所有子模块。


