---
layout: post
title: Maven-Gitflow插件发布项目
subtitle: Maven-Gitflow插件发布项目
tags: [Git,Maven]
category: 软件工程
---

## 1.准备上线产物

> 切换到项目develop分支并执行

```
mvn -DpushRemote=true -DallowSnapshots=false  gitflow:release-start
```

PS: !!!执行完上面的操作,会自动同步release分支到远端仓库,这里配置了自动构建,如gitlab-ci,请静静的等待CI的结束   
 
## 2.执行预发布操作
> 进入部署工具,如jenkins,将正式版本的产物如1.1.0,部署到预生产环境进行初步验证.没有预生产环境?好吧,那就使用测试环境替代.

## 3.预发布环境回测
> 配合QA进行预生产环境产物的验证.验证不通过,如果需要重新修正代码,这时有两条路可以选:(各有优缺点,这里强烈推荐方案1)


| 方案 | 优点 | 缺点 |
|---|---|---|
| 1. 原来的发布分支删除,不需要做finish操作.然后从develop再次做release-start的操作.这时会出一个小版本的产物,如1.1.1.(原则:仅在产物部署到生产环境,才做release-finish) | 版本清晰,tag出的名称和实际发的版本一致 | 和预定的上线版本不一致,有小版本的差异|
| 2. 直接在release分支上做代码变更,并手工迭代release分支的版本号到1.1.1,推送到远端进行构建. | 逻辑上的版本仍然是预定的版本 | 上线tag出的版本名和真实maven打出的产物版本会不一致. |



## 4.执行上线操作
> 进入部署工具,如jenkins,将正式版本的产物如1.1.0,部署到生产环境完成上线.

## 5.生产环境回测
> 配合QA进行生产环境的回归测试.其他同步骤3

## 6.上线完成 
> 线上回归测试通过后.需要执行上线结束的相关工作,这时在项目执行:(当然,这步可能提前到步骤3和步骤5进行,并重新开一个新的小版本产物出来)

```
mvn -DpushRemote=true -DallowSnapshots=false -DskipTestProject=true -DkeepBranch=false gitflow:release-finish
```

## 最佳实践:

1. 验证并修复maven版本号的工具[有必要时候再做]

```
mvn -B build-helper:parse-version org.codehaus.mojo:versions-maven-plugin:2.5:set -DnewVersion=1.0.0-SNAPSHOT
```
2. 请熟悉参考资料里的参数,有惊喜.

## 参考资料

### Maven-Gitflow插件Release参数说明

| 生命周期 | 参数 | 说明 | 其他 |
| --- | --- | --- | --- |
| gitflow:release-start | skipTestProject  | 是否跳过maven的test goal,默认false |  |
|   | allowSnapshots | 是否允许有Snapshot的依赖 |  |
|   | commitDevelopmentVersionAtStart |  |  |
|  | fromCommit | 控制从哪次提交开始发布 |  |
|  | fetchRemote | 是否同步远端分支到本地分支,默认true |  |
|  | pushRemote | 是否同步到远端分支,默认false |  |
| --- | --- | --- | --- |
| gitflow:release-finish | keepBranch | 默认false |  |
|  | pushRemote |  |  |
|  | skipTag | 是否跳过Tag,默认值为false,(对于release和hotfix的操作,会打tag) |  |
|  | skipTestProject |  |  |
|  | allowSnapshots | 是否允许有Snapshot的依赖 |  |
|  | digitsOnlyDevVersion | 是否移除版本号额外的标志,默认false. | 如,release的版本号:1.1.0-Final,下一个develop的版本号会是:1.1.1-SNAPSHOT |
|  | versionDigitToIncrement | 控制下一个开发的版本号是在第一个版本迭代,默认为空.可指定1. | 如versionDigitToIncrement=1,release版本号是1.2.3.4 ,则下一个版本号会是:1.3.0.0-SNAPSHOT |
|  | commitDevelopmentVersionAtStart | 控制develop分支的版本号变更的时机,默认为false | true:在开始做release的时候,也就是release-start的时候,先把develop分支的版本号变更到release的版本号,紧接着完成release分支的创建后,把版本变更到下一个版本号 false:在release分支做finish操作合并并到develop和master后触发develop版本的变更 |
|  |releaseRebase  | 是否采用rebase的方式进行合并,默认false,使用merge的操作 |  |
|  | preReleaseGoals | 执行release操作前执行的maven的goals | 如,-DpreReleaseGoals=test |
|  | postReleaseGoals | 执行release操作后需要执行的maven goals  |如 -DpostReleaseGoals=deploy  |

### 相关连接
- [maven-gitflow-plugin](https://github.com/aleksandr-m/gitflow-maven-plugin)
- [versions-maven-plugin](http://www.mojohaus.org/versions-maven-plugin/)


