---
layout: "post"
title: "jquery validator"
date: "2016-06-22 15:37"
---

# 自定义验证规则

$.validator.addMethod('validIP', function (value) {
    var split = value.split('.');
    if (split.length != 4)
        return false;

    for (var i = 0; i < split.length; i++) {
        var s = split[i];
        if (s.length == 0 || isNaN(s) || s < 0 || s > 255)
            return false;
    }
    return true;
}, 'ip地址格式错误');

使用方式
<input type="text" name="ip" data-rule-validIP="true" required/>

# validate

$("#form-").validate({
    errorElement: 'span',
    errorClass: 'error',
    focusInvalid: true,
    rules: {
        name: {
            required: true,
            maxlength: 50
        }
    },
    messages: {
        name: {
            required: "名称不能为空",
            maxlength: "名称不能超过50个字符"
        }
    },
    highlight: function (element) {
        $(element).closest('.input-group').addClass('has-error');
    },
    success: function (label) {
        if (label.parent()) {
            label.parent().siblings('.input-group').removeClass('has-error');
            label.remove();
        }
    },
    errorPlacement: function (error, element) {
        var $error = element.parent().parent().find('.error');
        if ($error) {
            $error.html(error);
        }
    },
    submitHandler: function (form) {
        // submit data
    }
});