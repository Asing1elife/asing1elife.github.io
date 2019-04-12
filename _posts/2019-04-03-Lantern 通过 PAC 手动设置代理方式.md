---
title: Lantern 通过 PAC 手动设置代理方式
date: 2019-04-03
author: asing1elife
categories:
- lantern
tags:
- lantern
---
> 现在科学上网的方式越来越玄乎，个人认为 Lantern 到目前为止都还算作比较好用的，但其默认的自动管理系统代理的方式真的是越来越不智能  
> 最直观的就是现在就算开的是局部代理，很多国内网站依然会被标记到走代理，比如 QQ 音乐就会认为是在墙外，不给播放在线音乐  
> 以下就提供如何让 Lantern 走自定义代理地址的方式  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 修改 Lantern 默认配置
1. 打开 Lantern 的设置界面，**管理系统代理** 默认是被勾选的，这里取消勾选
	* 然后下面会出现一句 **请注意巴拉巴拉…方法见上文** ，但并没找到什么上文可见，笑
2. 之后记住下面 **HTTP(S)代理服务器** 地址，后面会用到
![](http://asing1elife.com/sources/images/67DF127B-7EDE-4234-AE5C-DEA94304951D.png)

## 准备一个 proxy.pac 文件
1. PAC 文件的原理和其他一些深层含义这里就不介绍了，我也不太懂
2. 以下就是一个简单的 PAC 文件，内部实现其实是通过 JS 编写的
	* `autoproxy_host` 就是期望走代理的网址列表，此处我暂时只用了两个，准备以后有需要在挨个往里加就好
	* 其实这种地址在网上可以找到很多，但我还是喜欢按需添加
	* `return 'PROXY 127.0.0.1:51219';` 这行代码就是指定的 HTTP(S) 代理服务器地址
	* 这个地址在哪里找，就在前面 Lantern 的设置界面

```js
var autoproxy_host = {
    "google.com": 1,
    "twitter.com": 1
};

function FindProxyForURL(url, host) {
    var lastPos;
    do {
        if (autoproxy_host.hasOwnProperty(host)) {
        	return 'PROXY 127.0.0.1:51219';
        }
        
        lastPos = host.indexOf('.') + 1;
        host = host.slice(lastPos);
    } while (lastPos >= 1);
    return 'DIRECT';
}
```

## 为电脑的网络指定代理
1. 此处只演示 Mac 如何指定，因为我没有 Win 
2. 前往 **系统偏好设置** **网络** **高级** **代理** ，即可找到以下界面
3. 勾选 **自动代理配置** 之后，把前面准备好的 PAC 文件地址丢进去
	* 需要注意的是，**Mac 上的 PAC 文件指定不支持文件路径，必须是服务器路径**
	* 所以这里我是将 PAC 文件丢到了本地的 Tomcat 下，然后再指定
4. 文件指定好之后，还需要将网络断开重连才能生效
	* 一开始不知道要重连，以为是文件内容的问题，但是又不知道咋调试，真是搞死人
![](http://asing1elife.com/sources/images/9922DE5F-1FF1-4A4F-951D-A21CF09B6E73.png)

## 大功告成，提醒几句
1. 首先，文件指定好之后吗，必须重连网络，否则不会生效，这是关键
2. 其次，我认为这个最麻烦的是如果换了网络，则需要到网络设置里再指定一次