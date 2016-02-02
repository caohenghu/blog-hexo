title: angularjs项目下的gulp配置（一）
date: 2016-01-29 20:26:45
tags:
---
> 这是我的一个线上产品的真正gulp配置，可能不是最好的解决方案，但却是一个能用的方案

首先我们需要nodejs环境，没有安装过的同学，请参照[nodejs](https://nodejs.org/en/)官方文档，下载相应的安装包安装即可，我当前使用的nodejs版本为`4.2.6 LTS`。这里假设已经安装好了nodejs环境，下面开始介绍gulp。

<!--more-->

### gulp安装

***

1、全局安装gulp，这个是在命令行中执行用的。

```
$ npm install -g gulp
```

2、 在项目根目录下安装gulp，这个是在gulp配置文件中用的。

```
$ npm install --save-dev gulp
```

3、在项目根目录下创建一个名为gulpfile.js的配置文件，添加一些简单的内容如下。

```
var gulp = require('gulp');

gulp.task('default', function() {
    // 将你的默认的任务代码放在这
});
```

4、运行gulp。

```
$ gulp
```
执行这行命令，将会运行默认名为 default 的任务（task），它和执行`gulp default`是一样的，在这里，这个任务并未做任何事情。想要单独执行特定的任务（task），请输入 `gulp <task>`。请注意，中间是有空格的哦。

> 到这里，你已经学会如何安装gulp环境，接下来我们将开始配置一个运行angular环境的gulpfile.js。

### gulp插件

***

首先我们先来看看用到了哪些gulp的插件

``` javascript
var gulp = require('gulp'),
    del = require('del'),
    //autoprefixer = require('gulp-autoprefixer'),
    changed = require('gulp-changed'),
    concat = require('gulp-concat'),
    //connect = require('gulp-connect'),
    htmlReplace = require('gulp-html-replace'),
    htmlmin = require('gulp-htmlmin'),
    inject = require('gulp-inject'),
    jshint = require('gulp-jshint'),
    //livereload = require('gulp-livereload'),
    ngHtml2js = require('gulp-ng-html2js'),
    minifyCss = require('gulp-minify-css'),
    ngAnnotate = require('gulp-ng-annotate'),
    rename = require('gulp-rename'),
    replace = require('gulp-replace'),
    rev = require('gulp-rev'),
    sass = require('gulp-sass'),
    sequence = require('gulp-sequence'),
    //sourcemaps = require('gulp-sourcemaps'),
    uglify = require('gulp-uglify');
```

这里我们重点介绍一下angular相关插件

```javascript
gulp-ng-html2js
gulp-ng-annotate
```

#### gulp-ng-html2js

***

[gulp-ng-html2js](https://www.npmjs.com/package/gulp-ng-html2js)是一个处理angular里templateCache的插件，它会把所有的html模板合并存储到一个js当中，用法如下

```javascript
gulp.task('build-html', function () {
    return gulp.src(['src/app/**/*.html', 'src/common/**/*.html'])
        .pipe(htmlmin())
        .pipe(ngHtml2js({
            moduleName: 'template-app'
        }))
        .pipe(concat('template.tpl.js'))
        .pipe(gulp.dest(buildDir + '/js'));
});
```

执行`gulp build-html`后生成的`template.tpl.js`的代码如下，这里只列举出一部分

```javascript
(function(module) {
try {
  module = angular.module('template-app');
} catch (e) {
  module = angular.module('template-app', []);
}
module.run(['$templateCache', function($templateCache) {
  $templateCache.put('create/header/navBtn.tpl.html',
    '<li ng-class="{\'active\': isMe()}" ng-click="showMe()"><div ng-transclude></div><i ng-show="isMe()" class="arrow"></i></li>');
}]);
})();

(function(module) {
try {
  module = angular.module('template-app');
} catch (e) {
  module = angular.module('template-app', []);
}
module.run(['$templateCache', function($templateCache) {
  $templateCache.put('create/sideMenu/background.tpl.html',
    '<div class="eqc-background-panel" ng-class="{\'on\': backgroundPanel.isShow}"><div class="set-item"><span>背景颜色</span><eqd-input-color select-color="background.change" default-color="\'rgba(208,207,216,1)\'"></eqd-input-color></div></div>');
}]);
})();
```

可以看到html模板里的内容都被放到templateCache里方便我们使用了，当然，这里只是举一个例子，真正的html内容肯定不只是这么一点了，大家不用太纠结。

#### gulp-ng-annotate

***

[gulp-ng-annotate](https://www.npmjs.com/package/gulp-ng-annotate)是一个处理angularjs依赖注入的插件，它的用法如下


```
gulp.task('build-app-js', function () {
    return gulp.src('src/app/**/*.js')
        .pipe(ngAnnotate({single_quotes: true}))
        .pipe(gulp.dest(buildDir + '/js/app'));
});
```

正常我们写angular代码是这样子的

```
angular.module('app', [])
    .controller('AppCtrl', ['$scope', function($scope) {
        // 别的代码
    }])
```

但使用了这个插件之后，每一个依赖注入的项就不用再写两遍了，如

```
angular.module('app', [])
    .controller('AppCtrl', function($scope) {
        // 别的代码
    })
```

gulp-ng-annotate会帮我们生成带中括号的写法 ，这样子是不是节省了很多重复工作呢？尤其是在注入的服务非常多的时候，可以少写很多代码，并且也不用担心顺序有没有写错。

> 今天我们先介绍到这，重点介绍了和angularjs相关的两个插件，可能有些同学还不知道，同时也欢迎大家吐槽。