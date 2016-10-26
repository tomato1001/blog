---
layout: page
title: highcharts-notes
categories: highcharts
date: 2016-04-01 12:31:11 +0800
---


## 隐藏legend
在series的子对象中设置showInLegend: false

{% highlight js %}
series: [{
    name: '2016/03/18',
    data: [29.9, 71.5, 106.4, 129.2, 144.0],
    color: '#57C2F4'
    showInLegend: false
},
{% endhighlight %}

## 自定义tooltip

{% highlight js %}
tooltip: {
    formatter: function() {
        console.log(this);
        return this.x;
    }
}
{% endhighlight %}

## 折线图

### 删除legend图标中间的符号

在load事件中将中间的符号删除

{% highlight js %}
chart: {
    renderTo: this._id,
    events: {
        load: function() {
            that.$el.find(".highcharts-legend-item").find("path:last").remove()
        }
    }
}
{% endhighlight %}

### 取消legend的事件

{% highlight js %}
plotOptions: {
    series: {
        marker: {
            radius: 3
        }
    },

    line: {
        events: {
            legendItemClick: function() {
                return false;
            }
        }
    }
}
{% endhighlight %}

### 矩形网格

{% highlight js %}
xAxis: {
    tickInterval:1,
    tickmarkPlacement: 'on',
    gridLineWidth: 1,
    categories: ["20160323", "20160324", "20160325", "20160326", "20160327"],
    lineColor: '#428bca',
    lineWidth: 2,
    tickPosition: 'inside'
},

yAxis: {
    title: {
        text: ''
    },
    lineWidth: 2,
    tickWidth: 1,
    lineColor: '#428bca'
},
{% endhighlight %}


## 饼状图

### legend hover显示tooltip

{% highlight js %}
this.$el.find('.highcharts-legend text').each(function(index, element) {
    $(element).hover(function() {
        that._chart.tooltip.refresh(that._chart.series[0].data[index]);
    }, function() {
        that._chart.tooltip.hide();
    });
});
{% endhighlight %}

### 取消legend的事件

{% highlight js %}
plotOptions: {
    pie: {
        allowPointSelect: true,
        cursor: 'pointer',
        dataLabels: {
            enabled: false
        },
        showInLegend: true,
        point: {
            events: {
                legendItemClick: function (e) {
                    e.preventDefault();
                }
            }
        }

    }
}
{% endhighlight %}

### slice

{% highlight js %}
this._chart.series[0].data[index].slice(sliced);
{% endhighlight %}

### 显示tooltip

{% highlight js %}
function(index, show) {
            if (this._chart) {
                var d = this._chart.series[0].data[index];
                if (show) {
                    d.setState('hover');
                    this._chart.tooltip.refresh(d);
                } else {
                    d.setState();
                    this._chart.tooltip.hide();
                }
            }
            return this;
 }
{% endhighlight %}

# zoomType设置为x时，xy被同时放大bug

指定yAxis的min和max值即可解决该bug

{% highlight js %}
yAxis: {
    min: 0,
    max: Math.round(data.max) / 100 + 1000,
}
{% endhighlight %}


# line chart起点没有从0开始
当line chart的xAxis中包含categories时，xAxis的起点与yAxis会产生间距。可以使用labels属性来代替categories。

{% highlight js %}
var categories = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];
$('#container').highcharts({
    chart: {},
    xAxis: {

        labels: {
            enabled: true,
            formatter: function () {
                return categories[this.value]; // this.value reference to tickInterval
            }
        },
        tickInterval: 1,
        minPadding: 0,
        maxPadding: 0,
        startOnTick: true,
        endOnTick: true
    },

    series: [{
        data: [29.9, 71.5, 106.4, 129.2, 144.0, 176.0, 135.6, 148.5, 216.4, 194.1, 95.6, 54.4]
    }]
});
{% endhighlight %}
