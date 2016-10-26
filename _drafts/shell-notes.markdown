---
layout: "post"
title: "shell-notes"
date: "2016-06-22 16:47"
---

# debug

1. 使用命令行参数-x

	bash -x ./xxx

2. 在shell脚本中设置

{% highlight sh %}
#!/bin/sh
set -x
{% endhighlight %}

# 语法

* arithmetic

{% highlight sh %}
let a=1+1
let "a=1 + 1"

a=`expr 1 + 1`

a=$((1+1))
{% endhighlight %}

* case

{% highlight sh %}
v=$1
case $1 in
	start)
		echo "start";;
	stop)
		echo "stop";;
	*)
		echo "$1"
esac
{% endhighlight %}

* for

{% highlight sh %}
for ((a=1; a<=5; a++))
do
	echo "$a"
done
{% endhighlight %}

{% highlight sh %}
for i in $(seq 1 10)
do
	echo "$i"
done
{% endhighlight %}

{% highlight sh %}
for i in 1 2 3 4 5
do
	echo $i
done
{% endhighlight %}

{% highlight sh %}
# Can used in bash 3.x+

for i in {1..5}
do
	echo "3.x+, $i"
done
{% endhighlight %}

{% highlight sh %}
# can used in bash 4.0+
# {START..END..INCREMENT}

for i in {0..10..2}
do
	echo "$i"
done
{% endhighlight %}

# 其它

* 获得之前运行程序的pid  
  $!
  
# ftp

* 在ftp服务器上移动文件
	rename /orginal /target

* shell使用ftp命令上传文件

{% highlight sh %}
ftp -inv $publish_host << EOF
 user $publish_host_user $publish_host_password
 # $TARGET_PKG为本地文件
 put $TARGET_PKG /$PKG_NAME
 bye
EOF
{% endhighlight %}


打印结构树
	find	.	-print	|	sed	-e	's;[^/]*/;|____;g;s;____|;	|;g'
	

批量kill指定开始的进程

{% highlight sh %} 
#!/bin/sh

START=$1

if [[ -z "$1" ]];then
    echo "Please input start pid"
    exit 0;
fi

PID=

for ((i=0; i<10; i++))
do
   PID+="$((START + i)) "
done

kill $PID

{% endhighlight %}

获得脚本目录

{% highlight sh %} 
#!/bin/sh

# 获得当前执行目录
dirname "$(readlink -f '$0')"

#　获得脚本文件所在目录
DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"

echo $DIR

{% endhighlight %}

获得脚本文件所在目录

{% highlight sh %} 
#!/bin/sh

# resolve links - $0 may be a softlink
PRG="$0"

while [ -h "$PRG" ]; do
  ls=`ls -ld "$PRG"`
  link=`expr "$ls" : '.*-> \(.*\)$'`
  if expr "$link" : '/.*' > /dev/null; then
    PRG="$link"
  else
    PRG=`dirname "$PRG"`/"$link"
  fi
done

# Get standard environment variables
PRGDIR=`dirname "$PRG"`

# Only set CATALINA_HOME if not already set
DIR_HOME=`cd "$PRGDIR" >/dev/null; pwd`

echo $DIR_HOME

{% endhighlight %}