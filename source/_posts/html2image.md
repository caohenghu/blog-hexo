title: 将HTML转换成图片
date: 2016-02-25 15:52:49
tags:
---
由于工作需要，自己写了一个将html转换成图片的方法。在网上找了很久，没有找到特别好的方法，其中有一个叫html2canvas.js的插件，但是长期都没有更新了，并且有一些css的属性也不支持，所以经过自己不断折腾整出了一个自己的html2image.js。

<!--more-->

> 这是一个纯js的插件，没有依赖任何库，可以很方便集成

### 如何使用

```
html2image(elem, {
    size: 300, // 画布会以这个值进行缩放
    width: 1000, // 画布实际的宽
    height: 2000, // 画布实际的高
    success: function (canvas) {
        // 成功后会得到canvas，可用下面的方法转成图片
        // var dataUrl = canvas.toDataURL();
    }
});
```

### 实现原理
首先，需要将elem转换为字符串，可以通过下面的方式

```
var html = (new XMLSerializer).serializeToString(element);
```

> 因为需要使用`svg`里的`foreignObject`来引用外部的`html`，而由于安全性，`foreignObject`中是无法引用外部资源的，比如说图片。哪怕是同一个域名下的资源，也是不允许访问的，但可以转换成`base64`后再使用。后面我们将详细说明


接下来看看我们会使用到哪些外部资源，这时候需要进行处理

* 图片 `<img src="">`
* 背景图片 `background-image: url()`
* svg里的图片 `<image xlink:href=""></image>`
* 外部字体 `font-face`

在处理这几种资源时，除了字体外，我们可以通过正则拿到相应的`url`

```
// 图片和svg里的图片
var reg = /(src|xlink:href)="(http[^"]*)"/g;
// 背景图片
var reg = /url\((")?(http[^\)&]*)(")?\)/g;
// 字体
var reg = /font-family: '?([^';]*)'?/g;
```

通过`ajax`以`blob`的形式获取`url`对应的资源，结果会返回`blob`

```
/**
 * 通过ajax将图片和文字url转换为二进制blob
 * @param url
 * @param onSuccess
 */
function ajax2Blob(url, onSuccess) {
    var xhr = new XMLHttpRequest();
    xhr.open('get', url, true);
    xhr.responseType = 'blob';
    xhr.onreadystatechange = function () {
        if (this.readyState == 4 && this.status == 200) {
            onSuccess(this.response, url);
        }
    };
    xhr.send();
}
```

将blob转化成base64

```
/**
 * 将blob转换为dataUrl，避免跨域的问题
 * @param blob
 * @param callback
 * @param url
 */
function readBlobAsDataURL(blob, callback, url) {
    var fileReader = new FileReader();
    fileReader.onload = function (e) {
        callback(e.target.result, url);
    };
    fileReader.readAsDataURL(blob);
}
```

下面是将图片和字体转换为base64的例子，字体因为是定义在`css`里的，所以处理方式会不一样。注释的代码为非关键性代码，不影响阅读，但是在项目里需要

```
/**
 * 将图片url转换为base64存起来
 * @param urls
 */
function img2Base64(urls) {
    for (var i = 0; i < urls.length; i++) {
        ajax2Blob(urls[i], function (blob, url) {
            readBlobAsDataURL(blob, function (dataUrl, url) {
                // imgUrlCurrentCount++;
                var reg = new RegExp(url, 'g');
                html = html.replace(reg, dataUrl);
                // if (imgUrlTotalCount + fontUrlTotalCount !== imgUrlCurrentCount + fontUrlCurrentCount) return;
                // svg2canvas(html);
            }, url);
        });
    }
}

/**
 * 将文字url转换为base64存起来
 * @param urls
 */
function font2Base64(urls) {
    for (var i = 0; i < urls.length; i++) {
        ajax2Blob(urls[i], function (blob, url) {
            readBlobAsDataURL(blob, function (dataUrl, url) {
                // fontUrlCurrentCount++;
                var fontName = /\/([^\/]*)\.woff/.exec(url)[1];
                var font = '@font-face {font-family: ' + fontName + '; src: url(' + dataUrl + ');}';
                fonts.push(font); // fonts在外部定义的
                if (fontUrlCurrentCount == fontUrlTotalCount) {
                    fontFaceText = fonts.join('');
                }
                // if (imgUrlTotalCount + fontUrlTotalCount !== imgUrlCurrentCount + fontUrlCurrentCount) return;
                // svg2canvas(html);
            }, url);
        });
    }
}
```

通过以上的处理，所有的外部资源就已经全部转换为`base64`格式，`foreignObject`可以识别了。接下来就是将`html`转换成`canvas`了，看代码之前先说一下流程。
1. `svg`里可以通过`foreignObject`引用`html`
2. `style`里的样式是我的项目里需要才加的，可根据自己的项目进行调整，然后添加需要的字体
3. `svg`可通过`blob`包装后转换成`base64`
4. `img`通过`base64`转换为`canvas`

```
/**
 * 将 html 转换成svg，再将 svg 转换成 canvas
 * @param html
 */
function svg2canvas(html) {
    var data = '<svg xmlns="http://www.w3.org/2000/svg" width="' + width + '" height="' + height + '">' +
            '<foreignObject width="100%" height="100%">' +
            '<style>*{margin: 0;padding: 0;border: 0;outline: 0;}img,svg,li{vertical-align: middle;}li{list-style: none;}' + fontFaceText + '</style>' +
            html +
            '</foreignObject>' +
            '</svg>',
        blob = new Blob([data], {type: 'image/svg+xml;charset=utf-8'}),
        //url = domUrl.createObjectURL(blob),
        canvas = document.createElement('canvas'),
        ctx = canvas.getContext('2d'),
        img = new Image();

    readBlobAsDataURL(blob, function (dataUrl) {
        img.onload = function () {
            var newImg = changeImgSize(img.width, img.height);
            canvas.height = newImg.height;
            canvas.width = newImg.width;
            ctx.drawImage(img, 0, 0, img.width, img.height, 0, 0, newImg.width, newImg.height);
            //domUrl.revokeObjectURL(url);
            successFun(canvas);
        };
        img.src = dataUrl; // 如果使用domUrl.createObjectURL(blob)，会有跨域问题，因为svg里用了foreignObject
    });
}

/**
 * 根据传入参数改变输出尺寸
 * @param imgWidth
 * @param imgHeight
 * @returns {*}
 */
function changeImgSize(imgWidth, imgHeight) {
    var size = defaults.size,
        newImg = {};
    if (!size) return {
        width: imgWidth,
        height: imgHeight
    };
    if (imgWidth > imgHeight) {
        newImg.height = imgHeight * size / imgWidth;
        newImg.width = size;
    } else {
        newImg.width = imgWidth * size / imgHeight;
        newImg.height = size;
    }
    return newImg;
}
```

到这里，整个原理就讲完了。功能虽然是实现了，但可能还存在一些浏览器兼容问题。目前firefox支持得非常好，但chrome对于字体的识别不是特别好，图片在有些版本下会有些出不来，但最新的内核为48的版本图片基本都没问题了。别的浏览器我也没怎么测试了