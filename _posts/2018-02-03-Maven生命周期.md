---
layout: post
title: MAVEN工作原理
subtitle: "MAVEN工作原理"
tags: [Maven, markdown, Maven生命周期]
category: 软件工程
---



## maven插件的生命周期
![](http://oqk2bkk90.bkt.clouddn.com/14834267005578.jpg)

## maven指令与生命周期阶段的关系
![](http://oqk2bkk90.bkt.clouddn.com/14830016483295.jpg)


    mvn compile  //让当前项目经历生命周期中的1-->7 阶段 ：完成编译主源代码编译
    mvn package  //让当前项目经历生命周期中的1-->17阶段 ：完成打包
    mvn install  //让当前项目经历生命周期中的1-->22阶段 ：完成包安装到本地仓库
    mvn deploy   //让当前生命经历生命周期中的1-->23阶段 ：完成包部署到中心库中
## 日常关注的阶段

    1 应该将resource资源文件准备好，放到指定的target目录下----process-resources 阶段；
    2 将java源文件编译成.class文件，然后将class 文件放置到对应的target目录下----compile阶段；
    3 将test类型的resource移动到指定的 target目录下------process-test-resource阶段；
    4 将test类型的java 源文件编译成class文件，然后放置到指定的target目录下------test-compile阶段；
    5 运行test测试用例-------test阶段；
    6 将compile阶段编译的class文件和resource资源打包成jar包或war包--------package阶段；
    7 将生成的包安装到本地仓库中------install阶段
    8 将生成的包部署到远程仓库中-----deploy阶段

## MAVEN默认的生命周期和插件绑定
![](http://oqk2bkk90.bkt.clouddn.com/14830018675188.jpg)

## JAR和WAR默认的生命周期
![](http://oqk2bkk90.bkt.clouddn.com/14830019553942.jpg)

## MAVEN插件原理
![](http://oqk2bkk90.bkt.clouddn.com/14830020691173.jpg)

![](http://oqk2bkk90.bkt.clouddn.com/14830021059886.jpg)


## 使用mvn的help插件
### 查看maven-surefire-plugin插件test目标的详细介绍

    mvn help:describe -Dplugin=compiler
    mvn help:describe -Dplugin=compiler -Ddetail=true
    mvn help:describe -Dplugin=compiler -Dgoal=compile
    mvn surefire:help -Ddetail=true -Dgoal=test

### help插件的goals

    help:active-profiles
    help:all-profiles
    help:describe
    help:effective-pom
    help:effective-settings
    help:evaluate
    help:expressions
    help:help
    help:system



