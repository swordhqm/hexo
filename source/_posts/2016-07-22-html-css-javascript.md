---
title: html+css+javascript
date: 2016-07-22 18:55:47
tags: 
    - html
    - css
    - javascript
categories:
    - IT
---
# 需要用到的链接
[web api](https://developer.mozilla.org/en-US/)
[css兼容性查询](http://caniuse.com/)

# css example
* bootstrap 横版responsive 导航栏
`样式` :
![Alt text](/img/html_css_javascript/ex1_pc_show.png "PC 端样式")
![Alt text](/img/html_css_javascript/ex1_mobile_show.png "Mobile 端样式")
![Alt text](/img/html_css_javascript/ex1_1.png "navbar-header解析")
![Alt text](/img/html_css_javascript/ex1_2.png "navbar-header里隐藏折叠button")
![Alt text](/img/html_css_javascript/ex1_3.png "navbar collapse pc端显示")
![Alt text](/img/html_css_javascript/ex1_4.png "navbar collapse mobile端显示")

大体来说
1. 结构
navbar-header 包含两个部分 一个折叠button，一个文字链接
collapase 就是展开部分，主要是一个下拉列表。
2. css 要点
float, 大屏幕下navbar-header 会含有该属性，而进到小屏幕会去除该属性。大屏幕下，collapse 一直只有 collapse样式，该样式display: block.那么看到效果collapse宽度正好是container宽度，而navbar-header浮动在左边，不会影响到collapse宽度。
```css
@media (min-width: 768px) {
  .navbar-header {
    float: left;
  }
}
```

当点击button时，会为collapse添加上 in class，这个时候
```css
.collapse {
  display: none;
}
.collapse.in {
  display: block;
}
```

```html
<div class="container">
    <div class="navbar-header">
        <button class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse"> <span class="fa fa-bars"></span>
 
        </button> <a class="navbar-brand" href="/">example.com</a>
 
    </div>
    <div class="collapse navbar-collapse navbar-responsive-collapse">
        <ul class="nav navbar-nav navbar-right">
            <li class="dropdown"> <a href="#" class="dropdown-toggle" data-toggle="dropdown">
                admin
                <b class="caret"></b>
            </a>
 
                <ul class="dropdown-menu">
                    <li> <a id="account_messages" href="/messages/inbox/"><i class="fa fa-envelope"></i> Messages</a>
 
                    </li>
                    <li><a id="account_settings" href="/account/settings/"><i class="fa fa-user"></i> Settings</a>
                    </li>
                    <li><a href="/admin/"><i class="fa fa-cog"></i> Admin</a>
                    </li>
                    <li class="divider"></li>
                    <li> <a id="account_logout" href="/account/logout/"><i class="fa fa-power-off"></i> Log out</a>
 
                    </li>
                </ul>
            </li>
        </ul>
        <form id="accountLogOutForm" style="display: none;" action="/account/logout/" method="POST">
            <input type="hidden" name="csrfmiddlewaretoken" value="ij3Oh2VSjzAsDse29ve6PMs3FND13rcE">
        </form>
    </div>
</div>
```

# javascript basic
* 理解prototype
[ECMAScript](http://www.ecma-international.org/ecma-262/7.0/index.html#)
cf1, cf2, cf3, cf4, cf5 只会从protype 里继承CFP1，但不包括构造函数CF里的P1, P2
![--](/img/ecmascript_pic_s1.png "--")
* --

# 总结

* 关于流式布局
下面这部分利用table-cell 和 inline-block 以及 vertical-align:bottom 替代float
```css
#bars {
    display: table-cell;
    border: solid 1px black;
}
#bars > div {
    display: inline-block;
    vertical-align: bottom;
    width: 5px;
    background-color: #999;
    margin-left: 2px;
}
#bars > div:first-child {
    margin-left: 0;
}
```

# bootbox
关于处理focus问题，目前假设一个text input，一直通过blur或者focusout获取光标，如果这个时候，弹出bootbox，可以预见的问题，bootbox无法再把focus返回到这个text input，可以通过bootbox的callback里添加setTimeout来delay将focus重新送到text input，但是终究不太好，比如短时间发生连续输入，容易出现部分输入丢失，ex.扫描枪.
此外，看起来bootbox会受到无法focus的问题，导致javascript maximum call stack size exceeded 问题。

```javascript
#下面这个能够有效
$(document).on('hidden.bs.modal','.bootbox', function () {
    console.trace("-")
    if(!($('.modal').hasClass('in'))){
        console.trace("--");
        focus_manage(FOCUS_MANAGE_TYPE["FOCUS_TO_MAIN_PAGE_SCAN_CODE"]);        
    }
    
    if($('#Package_Modal').hasClass('in')){
        console.trace("---");
        focus_manage(FOCUS_MANAGE_TYPE["FOCUS_TO_Package_Modal_SCAN_CODE"]);
    }
});

$(document).on('show.bs.modal','.bootbox', function () {
    if(!($('.modal').hasClass('in'))){
        focus_manage(FOCUS_MANAGE_TYPE['FOCUS_TO_BOOTBOX']);    
    }
    
    if($('#Package_Modal').hasClass('in')){
        focus_manage(FOCUS_MANAGE_TYPE['FOCUS_TO_BOOTBOX']);
    }
});

#这个是用于解决tab key导致focus 跑到页面外面
$(document).bind('keydown', function(e){
    if(e.keyCode == 9) {
        console.trace("document get tab key");
        e.preventDefault();
    }
});
```

# bootstrap modal
```javascript
$('#Package_Modal').on('hidden.bs.modal'...)  如果modal有class fade 是不成功的
```

# 常见数据序列化或者format
[关于json](http://www.json.org/json-zh.html)

```javascript
ex. 
    data = {
        'plan': {
            'id':'1',
            'name':'1_plan'
        },
        'shipments': [
            {
                'id':'1-1',
                'name':'1_shipment'
            },{
                'id':'1-2',
                'name':'2_shipment'
            }
        ]
    }

    JSON.stringify(data), output 看起来会是:
    "{"plan":{"id":"1","name":"1_plan"},"shipments":[{"id":"1-1","name":"1_shipment"},{"id":"1-2","name":"2_shipment"}]}"
    JSON.parse(...) 会解析回对象

    下面这三个是jquery提供的方法：
    jquery.serialize() 会返回key1=val1&key2=val2&key3=val3 这种形式
    jquery.serializeArray() 会返回[Object, Object, Object,...], 每个Object形式
    {
        name:'',
        value:''
    }
    $.param(Object) 使用面比较窄，一般情况下只用于把{key:value, key:value}这种形式format成key1=val1&key2=val2
```


# encodeURI 和 encodeURIComponent
[link](http://osbornm.com/2010/05/05/how-and-when-to-encode-for-the-web/)
* encodeURI：不会进行编码的字符有82个 ：!，#，$，&，'，(，)，*，+，,，-，.，/，:，;，=，?，@，_，~，0-9，a-z，A-Z
* encodeURIComponent: 不会进行编码的字符有71个：!， '，(，)，*，-，.，_，~，0-9，a-z，A-Z
* 关于处理不会编码的特殊字符
```javascript
function fixedEncodeURIComponent (str) {
  return encodeURIComponent(str).replace(/[!'()*]/g, function(c) {
    return '%' + c.charCodeAt(0).toString(16);
  });
}
```

# javascript 浅拷贝和深拷贝

    // 浅层复制（只复制顶层的非 object 元素） 
    var newObject = $.extend({}, oldObject); 
    第一参数为目标对象，第二个参数为被拷贝对象，拷贝完将第一个参数作为返回值
    // 深层复制（一层一层往下复制直到最底层） 
    var newObject = $.extend(true, {}, oldObject); 第一个参数表示是否进行深拷贝
    //浅拷贝
    $.map(obj_arr, function (obj) { return obj; });  这个是浅拷贝会返回原始数组
    $.map( [obj1,obj2,obj3], function(obj){
        return $.extend(true,{},obj); //返回对象的深拷贝，每个元素都是拷贝的
    });

# notification

>Html5 notification 可以做到在桌面弹窗提示，参见facebook。只是facebook案例更特殊，有时候即使没有打开facebook，也会看到弹窗提示。这部分应该是依赖于google提供的api。
[链接](https://www.engadget.com/2015/03/12/google-chrome-beta-42-push-notification/)
[google push notification api](https://developers.google.com/web/fundamentals/getting-started/codelabs/push-notifications/)

#CSS

## CSS Basic ##
> 盒模型由， 内容， 内边距， 边框， 外边距组成
> 背景会应用于内容和内边距组成的区域
> width和height指的是元素内容区域的宽度和高度

> 外边距叠加
> 外边距叠加是一个相当简单的概念。 但是，在实践中对网页进行布局时， 它会造成许多混淆。 简单的说， 当两个或更多个垂直边距相遇时， 它们将形成一个外边距。这个外边距的高度等于两个发生叠加的外边距的高度中的较大者。
一般来说， 垂直外边距叠加有三种情况：

* 元素自身叠加
* 相邻元素叠加
* 包含（父子）元素叠加

_元素自身叠加_<br>
我们知道， 盒模型是 内容→内边距→边框→外边距 这样逐层包含的结构。 当元素没有内容（即空元素）、内边距、边框时， 它的上下边距就相遇了， 即会产生叠加（垂直方向）。 当为元素添加内容、 内边距、 边框任何一项， 就会取消叠加。
_相邻元素叠加_<br>
这个比较好理解， 相邻的两个元素， 如果它们的上下边距相遇，即会产生叠加。
_包含（父子）元素叠加_<br>
包含元素的外边距隔着 父元素的内边距和边框， 当这两项都不存在的时候， 父子元素垂直外边距相邻， 产生叠加。 添加任何一项即会取消叠加。
只有普通文档流中块框的垂直外边距才会发生外边距叠加。 行内框、 浮动框或绝对定位框之间的外边距不会叠加。

>vertical-align: The vertical-align CSS property specifies the vertical alignment of an inline or table-cell box.

```css
img.top{
    vertical-align: text-top
}
```

>css长度值及时间，频率，角度单位

```css
/*vw, vh 可以实现动态布局单位*/
css {
    vw:7 /*视窗宽度是100vw*/
    vh:2 /*视窗高度是100vh*/
}
```

>css 里关于position:fixed
如果使用 top, left 参数，element就会fixed relative to window.
如果使用 margin-top, margin-left参数，element就会fixed relative to its container.
[相关连接](https://www.gravitywell.co.uk/latest/design/posts/css-tip-fixed-positioning-inside-a-relative-container/)
此外：

```html
.top_banner {
    width:100%
    height:200px
    position: relative
    z-index:100
}
.side_bar {
    position: fixed
    margin-top: 50px
    z-index:20
}

<div class="top_banner">
    <div class="side_bar">
    </div>
</div>

```

relative 或者 absolute 的子元素的z-index 都是相对于父元素 而不是整个page。换句话说，上面的fixed 元素的z-index 就是相对于父元素。 

## CSS Ref
[vertical-align](http://www.w3schools.com/cssref/playit.asp?filename=playcss_vertical-align&preval=baseline)
[Centering in CSS: A Complete Guide](https://css-tricks.com/centering-css-complete-guide/)
[Email Template](https://webdesign.tutsplus.com/tutorials/creating-a-future-proof-responsive-email-without-media-queries--cms-23919)