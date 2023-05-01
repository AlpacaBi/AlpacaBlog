+++
title = "在鸿蒙2.0beta手机版发布的第二天，我写了一个鸿蒙的物联网应用手机APP"
date = "2020-12-17T13:39:39+08:00"
author = "Alpaca Bi"
authorTwitter = "" #do not include @
cover = ""
tags = ["鸿蒙"]
categories = ["业余折腾"]
keywords = ["鸿蒙", ""]
description = "昨日(12月16日)，华为发布了鸿蒙2.0手机开发者beta版，开放完整的鸿蒙OS 2.0系统能力、丰富的API（应用开发接口），以及强大的开发工具DevEco Studio等技术装备，现在我第一时间尝尝鲜，写一个鸿蒙的手机App。"
showFullContent = false
+++

# <span style="color:#F90">1.前言 </span>
其实鸿蒙2.0今年在9月份的时候已经发布，只不过那个时候只能开发手表和电视应用，还不支持手机。

那个时候在[掘金](https://juejin.cn/)也写过体验鸿蒙电视应用开发的文章，点击下面👇即可看到：

<span style="color:#F90">[鸿蒙2.0发布，让我给大家整个活](https://juejin.cn/post/6870840090608795662)</span>
![1](https://cdn.alpaca-bi.com/blog/harmonyBeta/1.png)

而现在能给手机端开发了，于是去了华为<span style="color:#F90">[HarmonyOS Developer](https://developer.harmonyos.com/cn/home)</span>网站，下载了最新的<span style="color:#F90">[HUAWEI DevEco Studio](https://developer.harmonyos.com/cn/develop/deveco-studio)</span>

创建项目，就看到了可以选择手机端开发了👇👇
![2](https://cdn.alpaca-bi.com/blog/harmonyBeta/2.jpg)

<br/>

之前为了折腾下阿里云的serverless，就做了一个物联网小项目,还起了个Alpaca IOT的名字

简单来说就是给我家里的树莓派和工位上的树莓派接上dht11温度传感器
![3](https://cdn.alpaca-bi.com/blog/harmonyBeta/31.png)

然后把采集到的温湿度数据上传到我自己写的阿里云serverless应用

之后撸一个前端页面，从我自己写的阿里云serverless应用里读取温湿度数据，显示下来。

然后把这个网页部署到服务器，就是下面的链接👇👇

<span style="color:#F90">[Alpaca IOT](https://iot.alpaca.run)</span>

网站如图所示
![4](https://cdn.alpaca-bi.com/blog/harmonyBeta/41.jpg)


所以我现在的工作，就是把我这个前端网页的业务，移植到鸿蒙app上


# <span style="color:#F90">2.新建项目 </span>

选择手机开发项目，然后项目的目录就出来了。
![5](https://cdn.alpaca-bi.com/blog/harmonyBeta/5.png)

当然了，第一件事并不是写代码，而是去弄一个运行鸿蒙系统的虚拟机（其实流程和Android Studio是一样的），这样我写的代码才有地方跑
![6](https://cdn.alpaca-bi.com/blog/harmonyBeta/6.png)

![7](https://cdn.alpaca-bi.com/blog/harmonyBeta/7.png)

然后砰的一下，一个运行鸿蒙os的虚拟手机就跑起来了。
![8](https://cdn.alpaca-bi.com/blog/harmonyBeta/8.png)

其实这玩意并不是真的运行在我本地的鸿蒙虚拟机，而是一个远程桌面而已，所以待会写出来的代码，估计是跑在远程的华为机器，然后以远程桌面的形式返回回来

所以体验有点不好，因为远程传输的画面很糊，操作起来也卡（不是系统卡，而是远程桌面网不好的话有延时）

好了，既然虚拟机跑起来了，就直接执行项目代码吧，然后一个hello world出来了，说明运行成功。
![9](https://cdn.alpaca-bi.com/blog/harmonyBeta/9.png)

由于我只是个卑微的前端菜🐔，而鸿蒙又支持js开发，所以我就用js开发了，然后顺便也对比下开发过程中和前端的区别，然后点进去源码的js文件夹，实现这个hello world的其实也是靠三座大山（hml、css、js）

他们的初始代码如下:
- hml: 理解成变种的html语法，大体和html一样，不过会有一点区别

```html
<!- index.hml -->
<div class="container">
    <text class="title">
        {{ $t('strings.hello') }} {{title}}
    </text>
</div>
```

<br/>

- JavaScript: 标准ECMA,就是js运行时会和浏览器的有一点区别（比如一些api不一定能直接用）

```js
// index.js
export default {
    data: {
        title: ""
    },
    onInit() {
        this.title = this.$t('strings.world');
    }
}
```
<br/>  
- css: 基本没发现和普通css有什么区别

```css
/* index.css */
.container {
    flex-direction: column;
    justify-content: center;
    align-items: center;
}

.title {
    font-size: 100px;
}

```  

看了下和小程序的模式类似，基本上也是h5+js+css,大体上和前端开发的区别不大，也是走的MVVM数据驱动视图那套模式，就是api和一些细节不一样，基本上可以说会Vue的就能上手。

所以一开始我不看文档，直接复制粘贴不加工，看看能不能“无痛移植”

结果发现不行，就我发现的，和前端网页相比，有以下区别
- hml
    1. 这里的布局只有flex和grid(默认flex)，所以默认的前端html传统文档流在这里会垮掉（比如div标签并不会换行，换行操作要靠设置flex相关属性）
    2. 文字内容要用`<text>`标签包起来，不包的话文字不会显示出来
![10](https://cdn.alpaca-bi.com/blog/harmonyBeta/10.png)
- js
    1. 如图，看来貌似这里的js运行时不支持async/await（有没有Promise没试过，看来貌似要地狱回调）
    2. 看来一些api也不能直接用，比如`fetch`，后来看了下开发文档想要搞网络请求要从`@system.fetch`引,而且引过来的fetch只能靠回调拿数据
![11](https://cdn.alpaca-bi.com/blog/harmonyBeta/11.png)
- css 
    1. 大致相同，就我发现的可能这里的css规则会更严格，比如我想设置背景色我用background不行，必须要background-color

所以我后面把开发文档看了一遍，基本上和小程序一样，总体架子不变，只有少量的区别，要根据文档修修改改才能把前端业务移植过来。


# <span style="color:#F90">3.开始移植 </span>

先给你看看我原来前端页面代码是怎样的（可能你们会觉得我这里的html语法怪异，因为我这个网页是基于ef.js弄得）
```html
 <!DOCTYPE html>
<html>
    <header>
        <title>Alpaca IOT</title>
        <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0,minimum-scale=1.0,user-scalable=0" />
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8"> 
        <style>
            body { 
                background-color: black;
                color: white;
            }

            #app {
                height: 95vh;
                display: -webkit-flex; /* Safari */
                display: flex;
                flex-direction: row;
                flex-wrap: wrap;
                justify-content: center;
                align-items: center;
                align-content: center;
            }

            #app .auther {
                position: fixed;
                bottom: 40px;
                font-size: 16px;
                cursor: pointer;
            }

            #app .auther a {
                text-decoration:none;
                color: white
            }

            #app .auther a:hover {
                text-decoration:solid;
                color: #f90;
            }

            #app .time {
                position: fixed;
                bottom: 15px;
                font-size: 16px;
            }

            #app .content{
                width: 350px;
                text-align: center;
                padding: 5px;
                margin: 25px 20px;
                background: #f90;
                border-radius: 8px;
                color: black;
            }

            #app .title {
                font-size: 30px;
                margin-bottom: 10px;
            }

            #app .data-box {
                display: flex;
                flex-direction: row;
                flex-wrap: wrap;
                justify-content: space-around;
            }

            #app .data-box .text{
                font-size: 24px;
            }

            #app .data-box .num{
                font-size: 30px;
            }

        </style>
    </header>
    <body>
        <div id="app"></div>
        <script src="https://alpaca.cdn.bcebos.com/js/ef.js"></script>
        <script>
            const { t } = ef;
            // 创建ef对象
            const app = new t`
            >div.content
                >div.title
                    .家里实时温湿度
                >div.data-box
                    >div
                        >span.text
                            .温度：
                        >span.num
                            .{{home_temperature}}
                        >span.symbol
                            .°C
                    >div
                        >span.text
                            .湿度：
                        >span.num
                            .{{home_humidity}}
                        >span.symbol
                            .%
            >div.content
                >div.title
                    .工位实时温湿度
                >div.data-box
                    >div
                        >span.text
                            .温度：
                        >span.num
                            .{{comp_temperature}}
                        >span.symbol
                            .°C
                    >div
                        >span.text
                            .湿度：
                        >span.num
                            .{{comp_humidity}}
                        >span.symbol
                            .%
            >div.auther
                >a
                    #href = https://github.com/AlpacaBi
                    #target="_blank"
                    .@Alpaca Bi
            >div.time
                .最后更新时间： {{time}}
            `
            // alicloud serverless 接口
            const url = "https://1055462284844024.cn-shenzhen.fc.aliyuncs.com/2016-08-15/proxy/alpaca-iot/alpaca-temp/getData"
            // 从接口拿数据赋给ef对象的data层
            const updateData = async() => {
                let res = await fetch(url)
                let resData = await res.json()
                app.$data = {
                    home_temperature: resData.home.temperature,
                    home_humidity: resData.home.humidity,
                    comp_temperature: resData.comp.temperature,
                    comp_humidity: resData.comp.humidity,
                    time: (new Date()).toLocaleString()
                }
            }
            updateData()
            // 把ef对象挂载在dom树上
            app.$mount({target: document.getElementById('app')})
            // 每15秒更新一次数据
            setInterval("updateData()", 60000);
        </script>
    </body>
</html>
```


后来磕磕碰碰，就只改了这3个文件，就好了，修改后的代码如下（你们可以看看和普通前端有什么区别）

- index.hml  
```html
<!- index.hml -->
<div class="container">
    <div class="content">
        <div class="title">
            <text class="text">家里实时温湿度</text>
        </div>
        <div class="data-box">
            <div>
                <text class="text">温度:</text>
                <text class="num">{{home_temperature}}</text>
                <text class="symbol">°C</text>
            </div>
            <div>
                <text class="text">湿度:</text>
                <text class="num">{{home_humidity}}</text>
                <text class="symbol">%</text>
            </div>
        </div>
    </div>
    <div class="content">
        <div class="title">
            <text class="text">工位实时温湿度</text>
        </div>
        <div class="data-box">
            <div>
                <text class="text">温度:</text>
                <text class="num">{{comp_temperature}}</text>
                <text class="symbol">°C</text>
            </div>
            <div>
                <text class="text">湿度:</text>
                <text class="num">{{comp_humidity}}</text>
                <text class="symbol">%</text>
            </div>
        </div>
    </div>
</div>
```  
<br/>  

- index.js  
```js
// index.js
import fetch from '@system.fetch';
export default {
    data: {
        home_temperature: 0,
        home_humidity: 0,
        comp_temperature: 0,
        comp_humidity: 0,
    },
    onInit() {
        let _this = this
        // 阿里云serverless接口
        const url = "https://1055462284844024.cn-shenzhen.fc.aliyuncs.com/2016-08-15/proxy/alpaca-iot/alpaca-temp/getData"
        // 每60秒向阿里云serverless读取数据并更新到前端页面
        setInterval(function() {
            fetch.fetch({
                url: url,
                success: function(response) {
                    const resData = JSON.parse(response.data)
                    _this.home_temperature = resData.home.temperature
                    _this.home_humidity = resData.home.humidity
                    _this.comp_temperature = resData.comp.temperature
                    _this.comp_humidity = resData.comp.humidity
                },
                fail: function(data, code) {
                    console.log('fail callback')
                },
            });
        }, 60 * 1000)
    }
}
```
<br/>  

- index.css  
```css
/* index.css */
.container {
    height: 100%;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: center;
    align-items: center;
    align-content: center;
    background-color: black;
}

.container .content{
    width: 600px;
    flex-wrap: wrap;
    align-items: center;
    padding: 10px;
    margin: 25px 20px;
    background-color: #f90;
    border-radius: 8px;
    color: black;
}

.container .content .title {
    width: 580px;
    margin-bottom: 20px;
}

.container .content .title .text {
    width: 580px;
    text-align: center;
    font-size: 60px;
}

.container .content .data-box {
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: space-around;
}

.container .content .data-box .text{
    font-size: 48px;
}

.container .content .data-box .num{
    font-size: 60px;
}


```  

然后我的鸿蒙物联网APP Alpaca IOT就这样跑起来啦！！！
![12](https://cdn.alpaca-bi.com/blog/harmonyBeta/12.png)

现在既然做出来了，最后一步，就是打包app了，怎么打包我就不说了，也就是按几个菜单而已，最后就打包出hap后缀的安装包了。

![13](https://cdn.alpaca-bi.com/blog/harmonyBeta/13.png)


所以从现在开始，我可以说自己是开发过鸿蒙APP的工程师了！！！

ps.一个开发过程中遇到的一个小插曲👇👇
![15](https://cdn.alpaca-bi.com/blog/harmonyBeta/16.png)



# <span style="color:#F90">4.其实有些事情还没做好 </span>

发现了吗，我上面一直都是对着远程的华为设备进行隔空操作，哪怕APP也打包出来了，也没地去安装。

所以很核心的问题来了，要跑在真手机上才有意义！！！

还好昨天华为就公测招募华为鸿蒙OS 2.0手机开发者Beta版，支持Mate 30 / P40系列OTA升级。
![14](https://cdn.alpaca-bi.com/blog/harmonyBeta/14.png)

而我恰好也有一台P40PRO，符合设备要求，所以我也提了申请了！！！
![15](https://cdn.alpaca-bi.com/blog/harmonyBeta/15.jpg)

等到申请通过，我的手机就可以OTA升级到鸿蒙系统，到时候把我刚刚打包的APP跑到我的手机上，再更新相关内容。

另外如果有人的设备刚好符合要求，有兴趣升级到鸿蒙OS的,<span style="color:#F90">[点击这里](https://developer.huawei.com/consumer/cn/activity/301607581257578636)</span>即可申请


# <span style="color:#F90">5.最后 </span>

看到这里你们肯定要打我，标题鸿蒙物联网APP说的那么高大上，现在看来无非就是做了个前端，每60秒往后端拿数据而已，随便一个前端菜🐔都能做。

这个时候估计不少大佬都要开始嘲讽我了:就这？？？

所以想想看，连我这种前端菜🐔也能有模有样弄个还能用的APP，而且我还觉得不难，那么那些有着安卓开发经验的大佬岂不是随随便便都能搞出一堆有用的应用，所以意义就在这里了。



最后欢迎转载我这个文章，就是说明出处就行了（出处来源就写来自于https://blog.alpaca.run就行了）


