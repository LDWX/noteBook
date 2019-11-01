# 浏览器渲染过程解析

### DOMContentLoaded

**用法：** 

```JavaScript
window.addEventListener('DOMContentLoaded', function() {
    console.log('this is DOMContentLoaded.')
})
```

### window.onload

**用法：**
```JavaScript
window.onload = function() {
    console.log('this is window.onload')
}
或者
window.addEventListener('load', function() {
    console.log('this is window.onload')
})
```

### document.readyState

**状态：**

- `loading: ` document正在加载
- `interactive: `文档已经完成加载，文档已经被解析。但诸如图像、样式表、框架之类的子资源仍在加载
- `complete: `文档和所有子资源已经完成加载。接下来将触发`window.onload`事件

**用法：**
`readyState`无法直接使用，但可以放在`document.onreadystatechange`事件中进行检测。

```JavaScript
document.onreadystatechange = function() {
    if（document.readySate == 'complete"） {
       console.log('this is onreadystatechange: ', document.readyState) 
    }    
}
```

### body onload

**用法：**
```html
<body onload='myLoadFunc()'>
</body
```

## 运行时机
### DOMContentLoaded
html 文档加载完毕，并且HTML 所引用的内联 js 以及外链 js 的同步代码都执行完毕后触发 **(当前页面DOM中的js中异步加载的css、js，图片资源都还没加载完成)**

### document.readyState == 'complete'
js中异步加载的css、js以及图片资源都已加载完成后触发，并在window.onload触发前触发

### window.onload
**js中异步加载的css、js以及图片资源都已加载**完成后触发

（viedo, audio, flash 不会影响`load`事件的触发）

### body onload
介于`document.readySate == 'complete'`与`window.onload`之间触发

## 浏览器渲染顺序
[参考文档](
https://yq.aliyun.com/articles/609917)


**1、浏览器首先下载该地址所对应的 html 页面。
2、浏览器解析 html 页面的 DOM 结构。
3、开启下载线程对文档中的所有资源按优先级排序下载。
4、主线程继续解析文档，到达 head 节点 ，head 里的外部资源无非是外链样式表和外链 js。
发现有外链 css 或者外链 js，如果是外链 js ，则停止解析后续内容，等待该资源下载，下载完后立刻执行。如果是外链 css，继续解析后续内容。
5、解析到 body**

1. 只有DOM元素
    1.  DOM树构建完成，页面进行首次渲染
2. DOM && js
    1. 当解析到外链js时，把js之前的DOM渲染到首次页面渲染上，同时阻塞后面DOM的构建。（js能够并行下载，但执行js仍会阻塞DOM树的构建）
    2. 所以在js执行完之前，我们在页面上看不到该js后面的DOM元素
3. DOM && css
    **1. 外链css不会影响css后面的DOM构建，但会阻塞渲染**
    2. 所以在外链css加载完之前，页面无法进行首次渲染
4. DOM && js && css
    1. 外链js与外链css的顺序会影响页面的渲染。当解析器获取到一个script标签，DOM将无法继续构建直到JavaScript执行完毕，而**JavaScript在CSS下载完，解析完，并且CSSOM可以使用的时候，JavaScript才能执行**    
     ![resourceLoad](images/resourceLoad.png)    
    2. 当body中js之前的外链css为加载完之前，页面不会进行首次渲染
    3. 当body中js之前的外链css加载完成后，js之前的DOM树和css进行合并，并将该js之前的DOM结构进行首次页面渲染

**6、文档解析完毕，页面重新渲染。当页面引用的所有 js 同步代码执行完毕，触发 DOMContentLoaded 事件。
7、html 文档中的图片资源，js 代码中有异步加载的 css、js 、图片资源都加载完毕之后，load 事件触发。**

## head 中资源的加载 &&  body 中资源的加载
1. js 资源加载都会停止后面 DOM 的构建，但不会影响后面资源的下载
2. css 资源不会阻塞后面的 DOM 构建，但会阻塞页面的首次渲染
3. 在 body 中第一个 script 资源下载完成之前，浏览器会进行首次渲染，将该 script 标签前面的 DOM 树和 CSSOM 合并成一棵 Render 树，渲染到页面中。这是页面从白屏到首次渲染的时间节点，比较关键。



## 其他
### 浏览器下载资源限制
浏览器对同一域名下的资源并发下载线程数有限制（chrome为6个）。超过6个资源需要下载，则后续资源需要在下载队列中等待。

解决方案：

将资源收敛在不同域名服务器下。只要保证需要下载的资源在不同**域名**下就能实现超过6个的下载限制，实现同时下载n个资源。

### JS执行时间限制
单个JS任务执行时间不要超过100ms。因为超过100ms的任务会导致UI更新延迟过长，用户会感觉页面有卡顿的感觉。

解决方案：

- 将一个大任务分割成多个小任务
- 使用定时器，将任务分割到不同的执行时间片段，为UI更新留出执行时间
- 将计算量较大的任务放在Worker中执行

### 资源下载
2008开始，浏览器可以异步对外链资源进行下载（js, css, image），但是js的执行还是同步的
![resourceDownload.png](images/resourceDownload.png)同步的脚本总是比异步的脚本拥有更高的优先级。视口中可见的图像会比那些底下的图片先下载完。

# defer与async区别

[参考文件1](
https://github.com/xiaoyu2er/blog/issues/8)
[参考文件2](
https://www.zcfy.cc/article/building-the-dom-faster-speculative-parsing-async-defer-and-preload-x2605-mozilla-hacks-8211-the-web-developer-blog-4224.html?t=new)

### 示意图：

 ![defer&async](images/defer与async.png)

### 什么都不加
```JavaScript
<script src='a.js'></script>
```
会阻塞后续DOM的加载与渲染过程。

浏览器会做如下处理：
- 停止解析 document.
- 请求 a.js
- 执行 a.js 中的脚本
- 继续解析 document

### defer
```javaScript
<script src="d.js" defer></script>
<script src="e.js" defer></script>
```
加载后续DOM的过程和`script.js`的加载异步进行，但`script.js`的执行则要等到所有DOM解析完成之后，`DOMContentLoaded`事件触发之前完成。`defer`比`async`先引入。

浏览器会做如下处理：
- 不阻止解析 document， 并行下载 d.js, e.js
- 即使下载完 d.js, e.js 仍继续解析 document
- **按照页面中出现的顺序，在其他同步脚本执行后，DOMContentLoaded 事件前 依次执行 d.js, e.js。**

### async
```JavaScript
<script src="b.js" async></script>
<script src="c.js" async></script>
```
加载和渲染后续DOM的过程和`script.js`的加载与执行过程异步进行。所以在`script.js`执行时，DOM的加载和渲染会被阻塞。

**`async`无法保证 js 的执行顺序，先下载完的 js 会先执行，即使该 js 的位置比较靠后。**

浏览器会做如下处理：
- 不阻止解析 document, 并行下载 b.js, c.js
- 当脚本下载完后立即执行。（两者执行顺序不确定，执行阶段不确定，可能在 DOMContentLoaded 事件前或者后 ）

*注意：* async 和 defer 属性只对外部脚本起作用，如果没有 src 属性它们会被忽略。

### preload
对于重要的资源，可以使用`<link rel="preload">`来告诉浏览器你需要尽快地加载他们。

(一些资源可能会深藏在CSS或者脚本当中，浏览器可能要花上很长一段时间才会发现他们)
用法：
```javaScript
<link rel="preload" href="very_important.js" as="script">
```
`as`属性告诉浏览器要下载的是什么。可能的值：
- script
- style
- image
- font
- audio
- video

**注意：** 字体对于页面渲染非常关键，但是他们直到浏览器确认他们会被使用之前都不会被加载。这个检查只发生在CSS已经被解析，应用，并且浏览器已经将CSS规则匹配到对应的DOM节点上时。

这会导致文字渲染中的不必要延迟。因此可以通过使用`preload`属性来避免。

**注意：**预加载字体还必须设置`crossorigin`属性，及时字体在同一个域名下：
`<link rel="preload" href="font.woff" as="font" crossorigin>`
(火狐和IE浏览器不支持)
