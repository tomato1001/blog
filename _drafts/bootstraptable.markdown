---
layout: "post"
title: "bootstraptable"
date: "2016-06-22 15:41"
---

<div id="custom-toolbar">
    <button type="button" class="btn btn-primary" id="btn-add">
        <i class="glyphicon glyphicon-plus"></i>添加
    </button>
</div>
<table id="table-pagination"
       data-toolbar="#custom-toolbar"
       data-url="${request.contextPath}/live/whitelist/list"
       data-show-refresh="true"
       data-pagination="true"
       data-row-style="rowStyle"
       data-side-pagination="client">
    <thead>
    <tr>
        <th data-field="name" data-valign="middle" data-align="center" data-width="300" data-sortable="true">
            名称
        </th>
        <th data-field="ips" data-valign="middle" data-align="center" data-formatter="formatter.ips">
            IP
        </th>
        <th data-align="center" data-valign="middle" data-formatter="formatter.op" data-events="operateEvents" data-width="150">
            操作
        </th>
    </tr>
    </thead>
</table>

data-pagination指定是否分页, data-url指定服务端请求地址,data-side-pagination指定是在客户端(client)还是服务端(server)进行分页.
当为client时,返回的数据如下
[
    {
        "id": 0,
        "name": "Item 0",
        "price": "$0"
    },
    {
        "id": 1,
        "name": "Item 1",
        "price": "$1"
    }
]
为server时,返回的数据如下
{
    "total": 200,
    "rows": [
        {
            "id": 0,
            "name": "Item 0",
            "price": "$0"
        },
        {
            "id": 1,
            "name": "Item 1",
            "price": "$1"
        }
    ]
}

data-formatter指定格式化函数,用于自定义数据显示

var formatter = {
    op: function () {
        return "<a class='edit' href='javascript:void(0)'>编辑</a> <a class='remove' href='javascript:void(0)'>删除</a>";
    },
    ips: function (value) {
        var html = "";
        $.each(value, function (i, v) {
            html += (i == 0 ? '' : '<br>') + v;
        });
        return html;
    }
};

data-events指定操作列的事件,用于进行编辑,删除等操作

var handler = {
    edit: function (row) {},
    del: function (id) {}
};

window.operateEvents = {
    'click .edit': function (e, value, row, index) {
        handler.edit(row);
    },
    'click .remove': function (e, value, row) {
        handler.del(row.id);
    }
};
