---
layout: post
title: browserify-react-babel workflow
categories: ["browserify", "babel", "react"]
---

# 安装依赖

``` sh
npm install --save-dev browserify babelify
npm install --save react  
```

# package.json

``` js
{
  "name": "bgbb",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "browserify": {
    "transform": [
      [
        "babelify",
        {
          "ignore": "/node_modules/",
          "extensions" : [".js", ".jsx"]
        }
      ]
    ]
  },
  "devDependencies": {
    "babelify": "^6.1.2",
    "browserify": "^10.2.4",
  },
  "dependencies": {
    "react": "^0.13.3"
  }
}
```

# 使用es6语法创建component

``` jsx
class Container extends React.Component {
	render() {
		return (
			<div>Container {this.props.message}</div>
		);
	}
}
```

# 使用es5创建component

``` jsx
var Container = React.createClass({
	render : function() {
		return (
			<div>Container {this.props.message}</div>
		);
	}
});
```

