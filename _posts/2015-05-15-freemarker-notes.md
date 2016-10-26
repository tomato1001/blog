---
layout: post
title:  "freemarker notes"
date:   2015-05-15 22:23:00
categories: freemarker
---
### if else替代

> `value?string(1, 0)`
>
> + value为boolean值  
> + string中的值可以包含表达式如***string(value, 0)***,表达式不能使用***${}***

### null值处理

> `value!defaultValue or value?? or (object.value)!defaultValue or (object.value)??`
>
> + ***!*** 表达式后面是默认值,当***value***为null时，将使用***!***后的值  
> + ***??*** 表达式可以判断是否为null
> + `(value??)?string(value, 'v')`，如果value为null或不存在时将出现空异常。可以使用if else判断空值，或`(value??)?string(value!, 'v')`

### 直接输出表达式

> `${r"${exp}"}`

### 整形处理
freemarker默认根据国家区域来格式化显示数字，如:1000将显示为1,000,可以使用以下表达式不进行转换

> `number?c`

* 除法操作将会有小数点，可以使用***?int***转换为整形

### assign使用

	<#assign value="test" value2=object.value>
	<#assign value>test</#assign>
	<#assign value>${object.value}</#assign>

### function使用

	<#function method param1 param2>
	<#return (param1 - param2)>  
	</#function>

+ 调用函数***${method(2, 3)}***

### 枚举
调用枚举的name方法判断

> enum.name() == "TYPE1"

### Map
freemarker中的map只支持字符串作为key，且map与java的map不同。如果应用无法将key转换为string，可以使用`myMap?api.get(nonStringKey)`
获取指定key的值。`?api`默认关闭，需要手动开启。

> + 使用map[key]或map.key获取指定key的值
