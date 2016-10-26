---
layout: "post"
title: "raphel"
date: "2016-06-22 16:50"
---

Raphael drag区域限制
思路: 先计算出容器允许的最大的x,y值,然后根据实际的x,y值进行判断并计算出x,y值

{% highlight js %}  
var Paper = function () {
    this.width = 480;
    this.height = 320;
    this.id = void 0; //    i
    this.offsetX = 0;
    this.offsetY = 0;
    this.watermark = void 0;
    this.paper = void 0;
    this.limitX = 0;
    this.limitY = 0;
};

Paper.prototype = {

    init: function (id, src, imageWidth, imageHeight) {
        this.id = id;
        if (imageWidth) {
            this.imageWidth = imageWidth;
            this.imageHeight = imageHeight;
        }
				// 计算出最大的x,y值
        this.limitX = this.width - this.imageWidth;
        this.limitY = this.height - this.imageHeight;
        this.paper = Raphael(document.getElementById("dragArea-" + this.id), this.width, this.height);
        $("#dragArea-" + this.id).data("paper", this);

        if (src != "" && src != null) {
            this.addWatermark(src, imageWidth, imageHeight);
        }
    },

    addWatermark: function (src, imageWidth, imageHeight) {

        if (src != null && src != "" && this.watermark != null) {
            this.watermark.remove();
        }
        var self = this;
        this.watermark = this.paper.image(src, this.offsetWidth, this.offsetHeight, imageWidth, imageHeight)
            .attr({stroke: "none", cursor: "pointer"});
        this.paper.set(this.watermark).drag(function (dx, dy) {
						// dx, dy为当前的偏移值,相对与当前点,可能会为负值
						
						// 超过最大的x,y值时,设置为最大值
            if (this.attr('x') > self.limitX || this.attr('y') > self.limitY ) {
                this.attr({x: this.ox + dx, y: this.oy + dy});
            } else {
                self.offsetX = Math.min(self.limitX, this.ox + dx);
                self.offsetY = Math.min(self.limitY, this.oy + dy);
                self.offsetX = Math.max(0, self.offsetX);
                self.offsetY = Math.max(0, self.offsetY);
                this.attr({x: self.offsetX, y: self.offsetY});
            }
            self.setValue(self);
        }, this.start);
    },

    start: function () {
        this.ox = this.attr("x");
        this.oy = this.attr("y");
    }
}

{% endhighlight %}