---
layout: post
title: Dell-OptiPlex 7070 安装黑苹果
subtitle: 黑苹果
tags: [Hackintosh]
category: 装机

# Dell-OptiPlex 7070 Hackintosh


## 系统镜像

[黑果小兵的镜像](https://blog.daliansky.net/macOS-Catalina-10.15.2-19C57-Release-version-with-Clover-5100-original-image-Double-EFI-Version.html)

## 显卡驱动

显卡，在CloverConfig里配置机型即可

## 声卡驱动
[声卡驱动注入](https://www.jianshu.com/p/d058e557983c)


这里使用了万能声卡驱动,过程中，首次万能声卡不工作，几番折腾，找到了几个增强的补充驱动`CodecCommander.kext`和`HDMIAudio.kext`,启动后，进入声音设置，左右声道调整下，解决人声低于背景音的问题

## 参考资料

* [Clover使用教程](https://blog.daliansky.net/clover-user-manual.html)
* [万能声卡](https://sourceforge.net/projects/voodoohda/files/)