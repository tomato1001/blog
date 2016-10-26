---
layout: page
title: staruml-crack
categories: linux staruml-crack
date: 2016-01-20 16:56:05 +0800
---
# staruml crack

编辑staruml目录下www/license/node/LicenseManagerDomain.js,修改为以下:

{% highlight js %}
function validate(PK, name, product, licenseKey) {
	var pk, decrypted;
	return {
	    name: "wind",
	    product: "StarUML",
	    licenseType: "vip",
	    quantity: "wind",
	    licenseKey: "wind crack just for fun"
	};
}
{% endhighlight %}

打开staruml,输入上面的licenseKey
