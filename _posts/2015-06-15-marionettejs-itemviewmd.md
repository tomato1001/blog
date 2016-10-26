---
layout: page
title: marionettejs-ItemView
categories: front-end marionettejs
date: 2015-06-16 23:10:56
---
### ItemView
代表一个单一视图，一般会使用一个model或者collection，或者都不使用。当model和collection同时存在时，默认使用model。可以重写serializeData实现自定义序列化逻辑。

{% highlight javascript %}
var UserView = Marionette.ItemView.extend({
	el : '#id',	//required
	tagName : 'div',
	className : 'user',
	template : '', //生成模版函数, false将使用当前html文档的节点
	events : {
		'click #id' : 'clickId',
		'click @ui.name' : 'clickName'
	},
	ui : {
		'name' : '#name'
	},

	modelEvents : {
		'change:id' : 'changeId', //id改变时触发
		'change' : 'changeModel' //模型任何改变都会触发
	},

	collectionEvents : {
		'add' : 'modelAddes' //集合添加元素时触发
	},

	clickName : function(){

	},

	clickId : function(){

	},

	onBeforeRender : function(){
		//视图生成之前的回调函数
	},

	onRender : function(){
		//视图生成之后的回调函数
	},

	onBeforeDestroy : function(){

	},

	onDestroy : function(){

	},

	//为视图提供序列化的模型或者集合。当model存在时，将调用view的
	//`serializeModel`方法。如果collection存在，将调用view的
	//`serializeCollection`方法，并将序列化后的值放入`items`中。
	//如果model和collection同时存在，将默认使用model。可以重写
	//`serializeData`方法提供自定义序列化逻辑。	
    serializeData: function() {
      if (!this.model && !this.collection) {
        return {};
      }
  
      var args = [this.model || this.collection];
      if (arguments.length) {
        args.push.apply(args, arguments);
      }
  
      if (this.model) {
        return this.serializeModel.apply(this, args);
      } else {
        return {
          items: this.serializeCollection.apply(this, args)
        };
      }
    },

	// Serialize a collection by serializing each of its models.
	serializeCollection: function(collection) {
	  return collection.toJSON.apply(collection, _.rest(arguments));
	},

	// Serialize a model by returning its attributes. Clones
	// the attributes to allow modification.
	serializeModel: function(model) {
	  return model.toJSON.apply(model, _.rest(arguments));
	}
});

var userView = new UserView({
	model : m,
	collection : c
});
userView.render();
{% endhighlight %}

