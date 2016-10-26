---
layout: post
title: "sublime text3"
categories: sublime
date: 2015-12-31 14:29:38 +0800
---
sublime text是一个文本编辑器,通过丰富的插件及简单的配置即可将sublime text变为一个高效的编辑器及开发工具,以下主要介绍相关插件及配置.

# Package Control
Package Control是一个插件,用于管理sublime text的插件,如：安装，修改及删除等.

## 安装
访问[package control installation][pci],然后复制对应sublime text版本的代码,打开sublime text,使用<code>ctrl+`</code>打开console并将复制的代码粘贴到console,最后按回车并等待安装完成.

[pci]:  https://packagecontrol.io/installation

## 使用
安装完成后使用`ctrl+shift+p`即可打开package control的控制界面,在界面中几乎可以控制所有的插件功能.

## 配置
package control支持丰富的配置参数,可以通过这些参数改变默认的设置,比如: 设置代理等.通过以下方式打开设置文件  
	
	Preferences > Package Settings > Package Control > Settings – User

该文件的格式为json,直接指定需要设置的键值即可.支持的设置值可以参考[settings][pcs].下面是一个简单的设置配置

[pcs]:  https://packagecontrol.io/docs/settings

{% highlight  json%}
{
	"auto_upgrade": false,
	"bootstrapped": true,
	"http_proxy": "1.2.3.4:18888",
	"https_proxy": "1.2.3.4:18888",
	"submit_usage": false
}
{% endhighlight %}

# SublimeCodeIntel

代码智能提示及完成插件

# SideBarEnhancements

侧边栏增强,支持文件和目录操作.安装之后,鼠标右键点击文件或目录将弹出操作菜单.

# Emmet

强大的前端开发插件,你值得拥有.该插件依赖pyv8,pyV8下载比较慢,可以选择手动安装.将下载的压缩包解压到`~/.config/sublime-text-3/Installed Packages/PyV8/linux64-p3/`目录下.

# BracketHighlighter

括号高亮匹配

# Alignment

内容对齐

# ConvertToUTF8 

将文件内容转换为utf-8,当打开windows下的文本文件时,使用该插件可以避免乱码.

# 其它插件

	HTMLBeautify
	html5
	Saas
	jQuery
	DocBlockr
	JsFormat
	Color Highlighter