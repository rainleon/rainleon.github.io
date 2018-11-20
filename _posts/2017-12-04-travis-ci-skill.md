---
layout: post
title: Travis CI 整理
subtitle: Travis CI 整理
tags: [CI, Travis]
category: 软件工程
---


### 简介
> Travis CI 是目前新兴的开源持续集成构建项目，它与jenkins，GO的很明显的特别在于采用yaml格式，简洁清新独树一帜。目前大多数的github项目都已经移入到Travis CI的构建队列中，据说Travis CI每天运行超过4000次完整构建。对于做开源项目或者github的使用者，如果你的项目还没有加入Travis CI构建队列，那么我真的想对你说out了


-------

### 原理
#### 构建生命周期
> Travis CI的构建分为两步：`install` 和 `script`,整个完整的生命周期如下：

1. OPTIONAL Install `apt addons`
2. OPTIONAL Install `cache components`
3. `before_install`
4. `install`
5. `before_script`
6. `script`
7. OPTIONAL `before_cache` (for cleaning up cache)
8. `after_success` or `after_failure`
9. OPTIONAL `before_deploy`
10. OPTIONAL `deploy`
11. OPTIONAL `after_deploy`
12. `after_script`

- 如果 before_install, install 或者 before_script 返回一个非0的执行结果，构建报错并且立刻停止执行
- 如果script 返回一个非0的执行结果，构建会失败，但是在返回最终失败结果之前，仍然会继续执行
- after_success, after_failure, after_script 以及随后的其他阶段不会影响构建结果，但是任何一个阶段的超时会导致最终结果为失败。

#### 语法定义

##### 构建特定分支

- 黑名单 指定的分支名不会触发构建

        branches:
            except:
            - legacy
            - experimental

- 白名单 指定的分支会触发构建。和黑名单同时指定白名单优先

        branches:
            only:
            - master
            - stable

- 正则指定

        branches:
          only:
          - master
          - /^deploy-.*$/

##### 环境变量
> 使用`env`来定义环境变量

- 公开的环境变量定义

        env:
        - DB=postgres
        - SH=bash
        - PACKAGE_VERSION="1.0.*"

- 定义多个变量

        rvm:
          - 1.9.3
          - rbx
        env:
          - FOO=foo BAR=bar
          - FOO=bar BAR=foo
> 上面的定义，使用了矩阵的变量组合，将触发4个构建
    1. Ruby 1.9.3 with FOO=foo and BAR=bar
    2. Ruby 1.9.3 with FOO=bar and BAR=foo
    3. Rubinius latest version (rbx) with FOO=foo and BAR=bar
    4. Rubinius latest version (rbx) with FOO=bar and BAR=foo

- 全局变量(Global Variables)

        env:
          global:
            - CAMPFIRE_TOKEN=abc123
            - TIMEOUT=1000
          matrix:
            - USE_NETWORK=true
            - USE_NETWORK=false
    > 上面将触发两组构建

        USE_NETWORK=true CAMPFIRE_TOKEN=abc123 TIMEOUT=1000
        USE_NETWORK=false CAMPFIRE_TOKEN=abc123 TIMEOUT=1000

 - 加密的变量

            env:
            global:
              - secure: mcUCykGm4bUZ3CaW6AxrIMFzuAYjA98VIz6YmYTmM0/8sp/B/54JtQS/j0ehCD6B5BwyW6diVcaQA2c7bovI23GyeTT+TgfkuKRkzDcoY51ZsMDdsflJ94zV7TEIS31eCeq42IBYdHZeVZp/L7EXOzFjVmvYhboJiwnsPybpCfpIH369fjYKuVmutccD890nP8Bzg8iegssVldgsqDagkuLy0wObAVH0FKnqiIPtFoMf3mDeVmK2AkF1Xri1edsPl4wDIu1Ko3RCRgfr6NxzuNSh6f4Z6zmJLB4ONkpb3fAa9Lt+VjJjdSjCBT1OGhJdP7NlO5vSnS5TCYvgFqNSXqqJx9BNzZ9/esszP7DJBe1yq1aNwAvJ7DlSzh5rvLyXR4VWHXRIR3hOWDTRwCsJQJctCLpbDAFJupuZDcvqvPNj8dY5MSCu6NroXMMFmxJHIt3Hdzr+hV9RNJkQRR4K5bR+ewbJ/6h9rjX6Ot6kIsjJkmEwx1jllxi4+gSRtNQ/O4NCi3fvHmpG2pCr7Jz0+eNL2d9wm4ZxX1s18ZSAZ5XcVJdx8zL4vjSnwAQoFXzmx0LcpK6knEgw/hsTFovSpe5p3oLcERfSd7GmPm84Qr8U4YFKXpeQlb9k5BK9MaQVqI4LyaM2h4Xx+wc0QlEQlUOfwD4B2XrAYXFIq1PAEic=
            matrix:
              - USE_NETWORK=true
              - USE_NETWORK=false
              - secure: <you can also put encrypted vars inside matrix>

 - 在Rep设置页面定义变量
 - 默认的环境变量

| name | value | Description |
| --- | --- | --- |
| CI | true |  |
| TRAVIS | true |  |
| CONTINUOUS_INTEGRATION | true |  |
| DEBIAN_FRONTEND | noninteractive |  |
| HAS_JOSH_K_SEAL_OF_APPROVAL | true |  |
| USER | travis | do not depend on this value |
| HOME | /home/travis | do not depend on this value |
| LANG | en_US.UTF-8|  |
| LC_ALL | en_US.UTF-8 |  |
| RAILS_ENV | test |  |
| RACK_ENV | test |  |
| MERB_ENV | test |  |
| JRUBY_OPTS | "--server -Dcext.enabled=false -Xcompile.invokedynamic=false" |  |
| JAVA_HOME |  |  |

 - 可选配置的变量

| name | value | description |
| --- | --- | --- |
| TRAVIS_ALLOW_FAILURE | true or false | 任务是否允许失败 |
| TRAVIS_BRANCH |   | 代码提交的场景，指分支的名称 代码合并的场景，指合并到的目的分支 |
| TRAVIS_BUILD_DIR |  | 绝对路径 |
| TRAVIS_BUILD_ID |  | Travis CI内部用的构建id |
| TRAVIS_BUILD_NUMBER |  | 当前构建的序列号 |
| TRAVIS_COMMIT |  | 当前构建正在测试的提交 |
| TRAVIS_COMMIT_MESSAGE |  | 提交的信息 |
| TRAVIS_COMMIT_RANGE |  | 提交范围 |
| TRAVIS_EVENT_TYPE | push/pull_request/api/cron | 表示构建的触发原因 |
| TRAVIS_JOB_ID |  | Travis CI内部用的任务id |
| TRAVIS_JOB_NUMBER |  | 当前任务的序列号 |
| TRAVIS_OS_NAME | linux/osx | 运行任务的平台 |
| TRAVIS_PULL_REQUEST |  | 如果是pr，则是请求的序列号，否则为false |
| TRAVIS_PULL_REQUEST_BRANCH |  | 如果是pr，则是PR的源分支名； 如果是push 则为空 |
| TRAVIS_PULL_REQUEST_SHA |  | 如果是PR，则是PR最后提交的SHA值， 如果是push，为空 |
| TRAVIS_PULL_REQUEST_SLUG | owner_name/repo_name or “" |  |
| TRAVIS_REPO_SLUG | owner_name/repo_name |  |
| TRAVIS_SECURE_ENV_VARS | true/false | 是否有加密的环境变量 |
| TRAVIS_SUDO | true/false | 是否开启sudo权限 |
| TRAVIS_TEST_RESULT | 0/1 | 0表示成功，1表示失败 |
| TRAVIS_TAG |  | 如果是git tag 则为tag的名称 |

 - 语言变量

| name | value | description |
| --- | --- | --- |
| TRAVIS_DART_VERSION |  |  |
| TRAVIS_GO_VERSION |  |  |
| TRAVIS_HAXE_VERSION |  |  |
| TRAVIS_JDK_VERSION |  |  |
| TRAVIS_JULIA_VERSION |  |  |
| TRAVIS_NODE_VERSION |  |  |
| TRAVIS_OTP_RELEASE |  |  |
| TRAVIS_PERL_VERSION |  |  |
| TRAVIS_PHP_VERSION |  |  |
| TRAVIS_PYTHON_VERSION |  |  |
| TRAVIS_R_VERSION |  |  |
| TRAVIS_RUBY_VERSION |  |  |
| TRAVIS_RUST_VERSION |  |  |
| TRAVIS_SCALA_VERSION |  |  |




-------

### 实战

#### 使用Travis CI构建Github项目

1. 使用github账户授权登录[travis－CI](https://travis-ci.org/)
2. 进入[Profile页面](https://travis-ci.org/profile),点击同步repo
    ![](http://img.aiuuwe.com/14919915587703.png)

3. 选中制定项目同步
    ![](http://img.aiuuwe.com/14919915172046.png)

4. 在项目设置页面可以设置触发构建的条件，以及定义环境变量
5. 在项目中添加.travis.yaml文件，提交即可触发一次travis－ci的构建：


        language: java

        jdk:
        - oraclejdk8

        cache:
          directories:
          - "$HOME/.m2"

        env:
          global:
          - CI_BUILD_REF_NAME=$TRAVIS_BRANCH
          - GIT_SERVICE=https://github.com
          - INFRASTRUCTURE=github
          - MAVEN_SKIP_RC=true

        before_install:
        - wget https://archive.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.zip
        - unzip -qq apache-maven-3.3.9-bin.zip
        - export M2_HOME=$PWD/apache-maven-3.3.9
        - export PATH=$M2_HOME/bin:$PATH

        script:
        - echo "mvn install"
        - mvn install
        - echo "mvn deploy"
        - mvn deploy

        services:
        - docker
        sudo: required


#### 最佳实践

1.  不触发构建
    git提交携带 [ci skip] 或者 [skip ci] 消息，则会被TravisCI忽略不触发构建
2. MAVEN_OPTS问题
    > 项目所有的MAVEN参数通过MAVEN_OPTS来拼接传递，但是使用travis所有参数失效。排查后，发现 travis replace MAVEN_OPTS with file /etc/mavenrc, 解决办法：添加参数

        env:
          global:
          - MAVEN_SKIP_RC=true
3. 项目使用gradle构建报错

        3.24s$ ./gradlew assemble
        Error: Could not find or load main class org.gradle.wrapper.GradleWrapperMain

    > 重写`install`脚本，跳过gradlew的默认检查
4. travis的构建过程，输出的日志不能超过4m，超4m会强制终止构建任务
5. 定义敏感信息的环境变量，建议在构建的setting中设置，且务必隐藏value值，使其不在日志中打印。


-------

### 参考资料

-[Travis官网](https://docs.travis-ci.com/user/customizing-the-build/#The-Build-Lifecycle)

