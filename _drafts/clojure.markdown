---
title: clojure
layout: post
---

# leiningen

创建项目
	lein new app <appname>

重新加载
	(require 'my.namespace :reload-all)
	(clojure.tools.namespace.repl/refresh)


grammer

显示namespace
	*ns*

数据结构

[1 2 3]  ; A vector (can access items by index).
[1 "2" "3"] 
{:a 1 :b 2} ; A hashmap (or just "map", for short).
#{:a :b :c}        ; A set (unordered, and contains no duplicates).
'(1 2 3)           ; A list (linked-list)

require

	reload
		(require '[execrise.structure] :reload)
		(require '(execrise structure) :reload)

集合
Some of the Clojure abstractions are:

	Collection (Lists, vectors, maps, and sets are all collections.)
	Sequential (Lists and vectors are ordered collections.)
	Associative (Hashmaps associate keys with values. Vectors associate numeric indices with values.)
	Indexed (Vectors, for example, can be quickly indexed into.)		
In the docs for the various functions, you'll often see that they take, for example, a "coll". This means that the particular function will work on any of the collections.

与java交互

调用构造函数创建实例

	(new java.lang.String "New")
	(java.lang.String. "new")
	(.toUpperCase "sssss") #调用实例方法
	