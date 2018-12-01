---
layout: post
title: Maven-Gitflow插件发布项目
subtitle: Maven-Gitflow插件发布项目
tags: [Git,Maven]
category: 软件工程
---

## 特性开发

```
从develop分支创建feature分支, 使用之前的版本号更新pom(s), 可选择更新或不更新版本号, 默认使用feature名字更新版本号更新pom(s)

# 确保没有未提交的修改
mvn -DpushRemote=true gitflow:feature-start
# 输入feature名称
# 检查所有出现版本号的位置是否被正确修改
# 进行开发
# 提交所有未提交的修改
将feature分支merge到develop分支, 使用之前的版本号更新pom(s), 删除feature分支

mvn -DskipTestProject=true gitflow:feature-finish
# 选择要完成的feature名称(可以同时有多个feature)
# 检查所有出现版本号的位置是否被正确修改
# -DskipTestProject=true 跳过测试
# -DkeepBranch=true 保留分支
git push origin develop:develop
# 可选的 触发ci发布新版本
```


## BUG修复

```
mvn -B -DhotfixVersion=1.6.0-fixOrder gitflow:hotfix-start
mvn -DskipTestProject=true gitflow:hotfix-finish
```

## 项目发布

### 1.准备上线产物

> 切换到项目develop分支并执行

```
mvn -DpushRemote=true -DallowSnapshots=false  gitflow:release-start
```

PS: !!!执行完上面的操作,会自动同步release分支到远端仓库,这里配置了自动构建,如gitlab-ci,请静静的等待CI的结束

### 2.执行预发布操作
> 进入部署工具,如jenkins,将正式版本的产物如1.1.0,部署到预生产环境进行初步验证.没有预生产环境?好吧,那就使用测试环境替代.

### 3.预发布环境回测
> 配合QA进行预生产环境产物的验证.验证不通过,将原来的发布分支删除,不需要做finish操作.然后从develop再次做release-start的操作.这时会出一个小版本的产物,如1.1.1.(原则:仅在产物部署到生产环境,才做release-finish)


### 4.执行上线操作
> 进入部署工具,如jenkins,将正式版本的产物如1.1.0,部署到生产环境完成上线.

### 6.上线完成
> 线上回归测试通过后.需要执行上线结束的相关工作,这时在项目执行:(当然,这步可能提前到步骤3和步骤5进行,并重新开一个新的小版本产物出来)

```
mvn -DpushRemote=true -DallowSnapshots=false -DskipTestProject=true gitflow:release-finish
```

## 最佳实践:

1. 验证并修复maven版本号的工具[有必要时候再做]

```
mvn -B build-helper:parse-version org.codehaus.mojo:versions-maven-plugin:2.5:set -DnewVersion=1.0.0-SNAPSHOT
```
2. 请熟悉参考资料里的参数,有惊喜.

## 参考资料

### Maven-Gitflow插件参数说明



| 参数 |  说明 | 生命周期 | 其他 |
| --- | --- | --- | --- |
| skipTestProject| 是否跳过maven的test goal,默认false |release-start <br/>release-finish <br/>xxx-finish|   |
| skipFeatureVersion| 控制是否跳过feature名追加到maven项目版本号后面,默认是false|feature-start |   |
| featureNamePattern| 特性分支名称规则| feature-start |   |
| allowSnapshots | 是否允许有Snapshot的依赖 | release-start <br/>release-finish ||
| commitDevelopmentVersionAtStart |  |  release-start <br/>release-finish  ||
| fromCommit | 控制从哪次提交开启发布动作 | release-start |
| fetchRemote | 是否同步远端分支到本地分支,默认true | xxx-start |
| pushRemote | 是否同步到远端分支,默认false | xxx-start<br/>xxx-finish | start默认不会推送,finish默认推送.
| keepBranch | 本地是否保留分支,默认false | xxx-finish | pushRemote为true,keepBranch为false,则会同步删除远端的分支
| skipTag | 是否跳过Tag,默认值为false,(对于release和hotfix的操作,会打tag) |release-finish<br/>hotfix-finish|
| digitsOnlyDevVersion | 是否移除版本号额外的标志,默认false. | release-finish|如,release的版本号:1.1.0-Final,下一个develop的版本号会是:1.1.1-SNAPSHOT |
|versionDigitToIncrement |下个develop版本从哪个位置迭代,默认为空.可指定[0,1,2,3...] |release-finish| 如versionDigitToIncrement=1,release版本号是1.2.3.4 ,则下一个版本号会是:1.3.0.0-SNAPSHOT |
|  commitDevelopmentVersionAtStart | 控制develop分支的版本号变更的时机,默认为false |release-start<br/>release-finish| **true**:在开始做release的时候,也就是release-start的时候,先把develop分支的版本号变更到release的版本号,紧接着完成release分支的创建后,把版本变更到下一个版本号 <br/>**false**:在release分支做finish操作合并并到develop和master后触发develop版本的变更 |
|useSnapshotInHotfix  | 是否允许使用快照方式发布hotfix | hotfix-start<br/>hotfix-finish |
|releaseRebase  | 是否采用rebase的方式进行合并,默认false,使用merge的操作 |  |
|preReleaseGoals | 执行release操作前执行的maven的goals | |如,-DpreReleaseGoals=test |
|postReleaseGoals | 执行release操作后需要执行的maven goals | |如 -DpostReleaseGoals=deploy  |
|----|----|----|-----|
|featureName | -B模式下指定分支名 |feature-start<br/>feature-finish |  |
|fromBranch | -B模式下指定从那个分支开出 |hotfix-start|  |
|hotfixVersion |-B 指定版本号 |hotfix-start<br/> hotfix-finish|  |
|featureName | -B模式下指定分支名 |feature-start<br/>feature-finish |  |
|developmentVersion | -B模式下指定develop版本号 |release-finish |  |
|releaseVersion | -B模式下指定release版本号 |release-start |  |

### 相关连接
- [maven-gitflow-plugin](https://github.com/aleksandr-m/gitflow-maven-plugin)
- [versions-maven-plugin](http://www.mojohaus.org/versions-maven-plugin/)


