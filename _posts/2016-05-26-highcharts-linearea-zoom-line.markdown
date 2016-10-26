---
title: highcharts可放大线性区域图并在图中划线
layout: post
categories: highcharts
date: 2016-05-26 10:11:08 +0800
---

option参数配置

{% highlight js %}
getOption: function(data) {
    return {
        chart: {
            renderTo: this._id,
            type: 'area',
            zoomType: 'x',
            resetZoomButton: {
                position: {
                    // x: -130,
                    y: -35
                }
            }
        },

        tooltip: {
            useHTML: true,
            borderColor: '#D5DBDB',
            formatter: function () {
                var d = {
                    time: moment(this.x).format("YYYY-MM-DD HH:mm:ss"),
                    bandWidth: this.y,
                    maxBandWidth: data.maxBandWidth
                };
                return bandWidthRender(d);
            }
        },

        plotOptions: {
            series: {
                marker: {
                    symbol: 'circle',
                    radius: 3
                }
            },

            area: {
                events: {
                    legendItemClick: function () {
                        return false;
                    }
                },
                marker: {
                    enabled: false
                }
            }
        },

        xAxis: {
            type: 'datetime',
            dateTimeLabelFormats: {
                day: '%m-%e'
            },
            startOnTick: false,
            endOnTick: true,
            minPadding: 0,
            maxPadding: 0
        },

        yAxis: {
            title: {
                text: ''
            },
            lineWidth: 0.5,
            tickWidth: 0.5,
            labels: {
                format: '{value:,.1f} Mbps'
            },
            min: 0,
            max: data.maxBandWidth,
            tickAmount: 4,
            plotLines: [{
                color: '#4094D2',
                dashStyle: 'ShortDot',
                value: data.maxBandWidth,
                width: 2
            }]
        },

        title: {
            text: ''
        },

        subtitle: {
            text: '点击并拖拽可放大细节',
            align: 'right',
            y: 14,
            x: -80
        },

        legend: {
            enabled: true,
            symbolRadius: 5,
            symbolWidth: 10,
            symbolHeight: 10
        },
        series: [
            {
                name: '峰值带宽',
                color: '#4094D2'
            },
            {
                name: '带宽统计',
                color: '#F7CA6F',
                data: data.data
            }
        ]
    };
}
{% endhighlight %}

## 说明

* 在图上划线

{% highlight js %}
plotLines: [{
    color: '#4094D2',
    dashStyle: 'ShortDot',
    value: data.maxBandWidth,
    width: 2
}]
{% endhighlight %}

* 打点
当需要在图中打点时，可以在series的data中指定marker

{% highlight js %}
{
    "x": 1464192900000,
    "y": 2.33,
    "marker": {
        "fillColor": "#4094D2",
        "lineWidth": 3,
        "lineColor": "#4094D2",
        "enabled": true
    }
}
{% endhighlight %}

* 开启缩放
在chart中配置如下参数

{% highlight js %}
zoomType: 'x',
resetZoomButton: {
    position: {
        // x: -130,
        y: -35
    }
}
{% endhighlight %}

zoomType指定缩放是作用与x轴还是y轴，也可以同时指定。该参数在某些版本中可能存在bug，如：指定的是x，缩放时却出现了同时缩放的情况。针对此情况，可以在对应轴的配置中增加max参数，用于限定最大值。如上面配置中的

{% highlight js %}
yAxis: {
    min: 0,
    max: data.maxBandWidth,
}
{% endhighlight %}

resetZoomButton用于指定缩放按钮的位置

## data参数值

{% highlight js %}
{
    "data": [
        {
            "x": 1464192300000,
            "y": 0.6
        },
        {
            "x": 1464192600000,
            "y": 1.11
        },
        {
            "x": 1464192900000,
            "y": 2.33,
            "marker": {
                "fillColor": "#4094D2",
                "lineWidth": 3,
                "lineColor": "#4094D2",
                "enabled": true
            }
        },
        {
            "x": 1464193200000,
            "y": 1.39
        },
        {
            "x": 1464193500000,
            "y": 1.56
        },
        {
            "x": 1464193800000,
            "y": 0.73
        },
        {
            "x": 1464194400000,
            "y": 1.16
        }
    ],
    "maxBandWidth": 2.33,
    "amountFlow": 346
}
{% endhighlight %}