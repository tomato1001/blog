---
layout: page
title: flyway
categories: flyway
---

# 安装及使用

1. 下载并解压flyway到任一文件夹

2. 修改conf/flyway.conf文件

{% highlight sh %}
flyway.url=jdbc:mysql://127.0.0.1/sampledb?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true

flyway.driver=com.mysql.jdbc.Driver

flyway.user=root

flyway.password=root
{% endhighlight %}

3. 将以下sql文件放入到sql目录下

V1__Create_person_table.sql

{% highlight sql %}
create table PERSON (
    ID int not null,
    NAME varchar(100) not null
);
{% endhighlight %}

V1.1__Add_people.sql

{% highlight sql %}
insert into PERSON (ID, NAME) values (1, 'Axel');
insert into PERSON (ID, NAME) values (2, 'Mr. Foo');
insert into PERSON (ID, NAME) values (3, 'Ms. Bar');
{% endhighlight %}

4. 登录mysql并创建sampledb数据库

5. 在flyway目录下执行

{% highlight sh %}
./flyway migrate
{% endhighlight %}

执行命令后,flyway将自动对sql目录下的文件进行排序并执行,且将信息记录到schema_version表中

# 相关命令

查看当前版本信息

	./flyway info

修复
当之前的脚本执行失败时,可以先将错误修复,然后执行以下命令进行修复

	./flyway repair

集成已有的数据库

	./flyway baseline -baselineVersion=1 -baselineDescription="Base line version"