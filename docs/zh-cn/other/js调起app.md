>最近在做项目的时候需要和app的做些对接工作
在手机浏览器中下载某app时，能判断该用户是否安装了该应用。如果安装了该应用，就直接打开该应用；
如果没有安装该应用，就下载该应用
下面是以打开淘宝的app代码为demo介绍

```
<button id="openApp">淘宝客户端</button>
<script type="text/javascript">

    var appCommon = {
        androidSchema: 'taobao://m.taobao.com/',
        androidDownUrl: 'https://www.taobao.com',
        iphoneSchema: 'taobao://m.taobao.com',
        iphoneDownUrl: 'https://itunes.apple.com/cn/app/%E7%99%BE%E5%BA%A6hi/id537923517?mt=8',
        openApp: function () {
            var self = this;
            if(navigator.userAgent.match(/(iPhone|iPod|iPad);?/i)){
                window.location.href = this.iphoneSchema; //ios app协议
                window.setTimeout(function() {
                    window.location.href = self.iphoneDownUrl;
                }, 500);
            }
            if(navigator.userAgent.match(/android/i)){
                window.location.href = this.androidSchema//android app协议
                window.setTimeout(function() {
                    window.location.href = self.androidDownUrl//android 下载地址
                }, 500);
            }
        }

    };
    document.getElementById('openApp').onclick = function(e){
        e.preventDefault();
        appCommon.openApp();

    };
```
> 首先试着打开手机端某个app的本地协议；如果超时就转到app下载页，下载该app。

几个疑问: 
1: 本地协议又是怎么建立的呢?
其实就是在app中将http协议转换为本地协议  具体怎么转换得需求查阅资料
但需要在app里面对配置文件做一下设置（一般是在manifest.xml文件的activity的intent filter里面）：
2: apps custom url schemes是什么呢？
就是你与app约定的一个协议URL，在IOS客户端或者Android客户端中可以设置一个URL Scheme
```
例如，设置URL Scheme：app，然后其他的程序就可以通过“ URLString=app://”调用该应用。
还可以传参数，如：app://reaction/?uid=1
```
3:怎么判断用户是否安装了该app呢?
其实很简单  用js代码请求该协议，如果在500ms内，如果有应用程序能解析这个协议，那么就能打开该应用；如果超过500ms就跳转到app下载页