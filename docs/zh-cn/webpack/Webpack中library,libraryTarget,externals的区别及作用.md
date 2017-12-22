>经常我们会希望通过script方式引入库，如CDN方式的jquery，
我们在使用的时候依旧用require方式，但是却不希望webpack将他编译到文件中。 如下
```
<script src="http://code.jquery.com/jquery-1.12.0.min.js"></script>
```
>因为模块化开发，杜绝一切全局变量，我们想这样去使用他
```
const $ = require("jquery");
$("#content").html("XXXX");
```
> 这时，我们需要配置externals
```
module.exports = {
     output: {
           libraryTarget: "umd"
     },
     externals: {
           jquery: "jQuery" 
    },
    ...  

```
> 看看编译后什么变化
```
({   0: function(...) { var jQuery = require(1);
     }, 1: function(...) {
       module.exports = jQuery; // 这里是把window.jQuery赋值给了module.exports
                                             // 因此可以使用require来引入
    },
    /* ... */
})
```
> 实际写个例子，我们经常会有自己处理业务数据的工具库Utils.js,传统的方法没有提供UMD的那些功能，只能从window或者global暴露方法
```
window.Utils = {
　　add: function (num1, num2) { return num1 + num2 },
}
```
然后引入 然后script方式引入
```
<script src="http://xxx/Utils.min.js"></script>
使用如下
const res = Utils.add(1,2);
```
配置externals改造成模块化开发方式
```
module.exports = {
     output: {
           libraryTarget: "umd"
     },
     externals: {
           Utils: "Utils" 
    },
    ...  
}
此时使用方式
const utils = require("Utils");
const res = utils.add(1,2);
```

#### externals的配置
1.首先是libraryTarget的配置，我们使用umd方式，这样便可以用任何一种引入方式，即支持cmd，amd，及全局
```
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD
        define(['jquery'], factory);
    } else if (typeof exports === 'object') {
        // Node, CommonJS之类的
        module.exports = factory(require('jquery'));
    } else {
        // 浏览器全局变量(root 即 window)
        root.returnExports = factory(root.jQuery);
    }
}(this, function ($) {
    //    方法
    function myFunc(){};
  
    //    暴露公共方法
    return myFunc;
}));
```
2.library和libraryTarget的使用
有时我们想开发一个库，如 underscore工具库，可以用commonjs和amd方式使用也可以用script方式引入。
这时候我们需要借助library和libraryTarget，我们只需要用ES6来编写代码，编译成通用的UMD就交给webpack了。
如:
```
exports default {
    add: function (num1, num2) {
         return num1 + num2; 
    }         
}
接下来配置webpack
module.exports = {
  entry: {
        myTools: "./src/Utils.js"     
    },
  output: {
        path: path.resolve(_dirname, "build"),
        filename: [name].js,
        chunkFilename: [name].min.js,
        libraryTarget: "umd",
        library: "Utils"   
   }          
}
library指定的是你require时候的模块名。
```