layout: post
title: 编写Bootstrap插件的步骤
date: 2017-06-18 10:13:30
tags:
 - Bootstrap
 - JavaScript
 - jQuery
categories:
 - 学习小结
feature: /assets/images/features/bootstrap-logo.jpeg
toc: true
---
最近在给自己的动力学模拟程序写个web界面，目前采用了bootstrap + flask的组合, 看了下bootstrap的一些组件样式和内置的插件的使用，在实现自己的网站的时候肯定会有自己的需求，于是需要开发或者扩展Bootstrap插件，这里就总结下开发Bootstrap插件的步骤。

Bootstrap插件本质上就是一个jQuery插件,除了将插件函数绑定到jQuery的原型对象上以外，Bootstrap还需要遵守一些其他的额外的步骤方便维护和学习等。

<!-- more -->

## Bootstrap的JS插件的实现步骤
### 1. 声明立即调用函数
这个就和开发jQuery插件相同，是为了在文档加载完之后执行回调函数, 类似
``` javascript
$(document).ready(function() {
    // ...
});
```
或者
``` javascript
$(function() {
    // ...
});
```
但是为了避免与其他库的命名冲突，Bootstrap里统一使用这种方式:
``` javascript
;(function($) {
    "use restrict";
    // ...
})(jQuery)
```

### 2. 定义插件核心类以及相关的原型方法
通过定义插件类，我们能够将选择器获取的元素以及操作元素的接口都封装到类中。这样Bootstrap插件实际上都是核心类的原型方法的调用。

### 3. 定义jQuery插件并重设插件构造函数
这一步是在上一步已经定义好操作jQuery对象的接口的基础上的，我们定义一个jQuery插件(就是一个js函数绑定到jQuery原型`jQuery.fn`上)。执行这个插件的过程就是将第2步插件类实例化并绑定到相应元素和判断执行相应原型方法的过程。

### 4. 防冲突处理
毕竟是自定义插件，难免有可能会与空间中定义好的其他插件重名造成冲突，这个时候我们需要写一个函数可以将插件的名称还给原始的插件。这部分比较固定，一般是这样处理:
``` javascript
// 例如我的插件名叫form_status

// 将初始的插件先保存下来
var old = $.fn.form_status

// 定义一个放冲突函数，可以将原始的插件对象归还回来
$.fn.form_status.noConflict = function() {
    $.fn.form_status = old;
    return this;
};
```

### 5. 绑定触发事件
在前面4步以后已经可以通过js直接调用插件实现相应效果，这里的事件绑定主要是针对Bootstrap的HTML声明式的事件触发。

## 插件开发实例
这里我就以一个简单用于我自己web应用的一个现实表单状态现实的Bootstrap插件的开发为例子说明下。

1. 声明立即调用函数
    ``` javascript
    ; (function($){
        "use restrict";
    })(window.jQuery);
    ```

2. 定义插件类
    ``` javascript
    /* Define FormStatus class */
    var FormStatus = function (element) {
        this.$element = $(element);
    };

    FormStatus.DEFAULTS = {
        show: true,
        status: 'error',
        msg: 'Error!'
    };
    ```
    这里主要是定义类构造函数，已经类属性用于存放默认参数。属性中包含相应的元素的jQuery对象，可以在原型方法中方便的操作相应的元素.

    然后我们需要定义用于操作绑定元素的接口：
    ``` javascript
    // Prototype methods.
    FormStatus.prototype.show = function(option) {
        var wrapper = '<div class="form-group has-' + option.status + '"></div>';
        this.$element.wrap(wrapper);
        var msg_element = '<label class="control-label">' + option.msg + '</label>';
        this.$element.before(msg_element);
    };

    FormStatus.prototype.remove = function(option) {
        if (this.$element.parent('form-group')) {
            this.$element.unwrap();
        }
        this.$element.prev('label.control-label').remove();
    };
    ```

3. 创建form_status jQuery插件
    ``` javascript
    // New jQuery plugin.
    var old = $.fn.form_status;
    $.fn.form_status = function(option) {
        return this.each(function() {
            var $this = $(this);
            var data = $this.data('bs.form_status');

            // Merge options.
            var options = $.extend({}, FormStatus.DEFAULTS, option);

            // Bind FormStatus instance to element.
            if (!data) {
                $this.data('bs.form_status', (data = new FormStatus(this)));
            }

            // Call corresponding prototype methods.
            if (options.remove) {
                data.remove(options);
            } else if (options.show) {
                data.show(options);
            }
        });
    };
    $.fn.form_status.Constructor = FormStatus;
    ```
    可以见到我们将插件类的实例绑定到了元素的`bs.form_status`属性中，当我们调用`$(selector).form_status(options)`的时候便会根据传入的参数选择调用原型方法。

4. 防冲突处理
    ``` javascript
    // No conflict.
    $.fn.form_status.noConflict = function (){
        $.fn.form_status = old;
        return this;
    };
    ```

5. 触发事件绑定
    由于我不是很喜欢HTML声明式调用，于是我就没有写相应的绑定事件，绑定会在js中单独编写就好了。


## 总结
以上就是开发一个Bootstrap插件的过程。主要是如何编写插件类已经绑定jQuery插件，插件类是为了封装操作接口，jQuery插件用于调用接口操作元素对象。我自己开发的代码都放在GitHub上:https://github.com/PytLab/kynetix-webapp
