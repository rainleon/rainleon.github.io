---
layout: post
title: 使用Rancher2.0安装K8s
subtitle: 使用Rancher2.0安装K8s
tags: [Docker,Rancher,K8S,Kubernetes]
category: 软件工程
---

# 使用Rancher2.0安装K8s

在这篇文章中，我简单介绍一下Rancher以及你运行它的方式。然后，我将展示如何将它安装到主机（这里使用的是Ubuntu虚拟机），以便在几分钟内让Kubernetes在本地运行。

Rancher最近发布了2.0版本的alpha版本，这个版本All in Kubernetes，鉴于此我想体验看看。我现在只是在做试验，但Rancher看起来像是一个非常有前途的方式，让你获得一个本地的Kubernetes集群并运行。

> 注意：Rancher实验室有两个独立的产品 - Rancher和RancherOS。 RancherOS是一个有趣的操作系统，其中几乎所有东西都是集装箱的，但那不是我在这里尝试的内容。


## 什么是Rancher

如果大多数读这篇文章的人从来没有听说过Rancher，我也不会感到惊讶。事实上我也没有听说过，直到最近。 但是如果你只是想方设法掌握docker，并且考虑使用Kubernetes进行服务编排，那么Rancher更值得一看。

那么,究竟什么是Rancher,用官方自己的话来说:
> Rancher是一个开放源代码软件平台，使组织能够在生产环境中运行和管理Docker和Kubernetes。借助Rancher，企业不再需要使用一套独特的开源技术从头构建一个容器服务平台。 Rancher提供管理生产中容器所需的整个软件堆栈.

如果你还问我的话，那么这些话就没有什么意义了。

本质上，Rancher提供了一个简单的方法来通过Web界面管理你的容器编排器（如Kubernetes或者Docker Swarm）。 它是你编排工具中的一个编排器。 我知道提到一个GUI会让很多死硬的控制台粉丝厌恶,因为我和大部分人一样喜欢一个好的命令行，但是有时候有一个友好的用户界面也是很好的。

## Rancher如何工作

Rancher包含一个中央管理服务，在Docker容器中的主机上运行的服务器以及运行代理服务的一个或多个其他主机，同样在Docker容器中。 主机可以是任何Linux主机，而Rancher可让您轻松连接到EC2，Digital Ocean或Azure等云主机。 或者，您可以添加任何运行受支持版本的Docker的主机。

![](http://img.aiuuwe.com/15181589813882.png)

Rancher添加了一层Docker化的基础设施来连接所有主机，管理存储，网络和DNS等。只需按照向导添加一个新的主机，Rancher就可以设置并监控它.

Rancher不会替换您的“本地”协调器。您仍然可以使用Kubernetes从仪表板或命令行管理您的应用程序。 Rancher甚至提供了一个方便的浏览器内核，用于在Kubernetes集群上运行`kubectl`命令：
![](http://img.aiuuwe.com/15181591305889.png)

## Rancher能为我们带来什么
那么为什么要使用Rancher？ 就个人而言，Rancher看起来会在搭建Kubernetes集群上花费很多成本。 它只花了我几分钟（也许半小时，如果我仔细算起来！）从头开始到集群运行，到最后看到Kubernetes仪表板。

Rancher也以类似的方式提供了各种“应用程序”，您可以使用这些“应用程序”快速启动一系列服务。 这些可以是像Ghost这样简单的单容器应用程序，也可以是像Openfaas这样的整个多容器系统。 所有这些只需点击几下，让他们在您的主机上运行

![](http://img.aiuuwe.com/15181593368853.png)

从组织的角度来看，Rancher看起来有很多企业级的集成和支持，例如使用Active Directory进行身份验证以及管理对多个环境的访问。 我没有亲自看过这个方面，但它看起来应该勾选正确的方框，让人们开心。

## 如何开始Rancher
开始你需要做的一件事,就是准备一台可以安装Rancher服务器的Linux主机。只要它运行的是[docker的支持版本](http://rancher.com/docs/rancher/v2.0/en/quick-start-guide/#preparing-a-linux-host)，你才会很顺利的开始下一步。 它可能是一台本地机器，一台虚拟机，一台云主机，并不需要非常强大。

我决定让Rancher在本地虚拟机上运行，然后安装Kubernetes，看看体验是怎样的。这篇文章描述了我经历的所有步骤。


## 在本地虚拟机上运行Rancher

这篇文章的其余部分描述了我如何在本地虚拟机中运行Rancher 2.0，并部署了Kubernetes。正如我之前提到的，v2.0只是alpha版本，所以事情可能会改变，但对我来说一切都非常顺利.

> 友情提示，我是一个Windows的家伙，所以如果你更习惯于使用Linux和设置虚拟机，那么这篇文章的其余部分可能会感觉有些困难。这对我自己来说是一个很好的记录，所以如果有点慢的话，请谅解！

## 安装虚拟机
我不会在这里安装管理程序 - 我使用的是VMware Player，但我确信你可以使用Virtual Box或Hyper-V。

1. 安装Ubuntu

    Rancher服务器v1.6将支持任何现代Linux发行版，但对于v2.0，目前可能需要Ubuntu 16.04。我确信稍后会有扩展，但目前它可以满足我。
    从[https://www.ubuntu.com/download/desktop](https://www.ubuntu.com/download/desktop)下载Ubuntu 16.04并将其安装到您的虚拟机管理程序中。到桌面，打开一个终端（`CTRL + ALT + T`）。
    与新操作系统一样，首先要安装更新。你需要运行两个命令：
    * `sudo apt-get update` - Updates the list of available packages and their versions
    * `sudo apt-get upgrade` - Actually installs the updates
    
    ![](http://img.aiuuwe.com/15183248848568.png)
    现在我们有一个最新的操作系统，我们可以安装Docker。

2.  安装Docker
    
    Rancher在Docker容器中运行，因此您需要在主机上安装兼容版本的Docker。对于v2.0版本，兼容版本是：
    
    * Docker v1.12.6
    * Docker v1.13.1
    * Docker v17.03-ce
    * Docker v17.06-ce
    
    我使用了最新的兼容版本v17.06-ce。我按照从这里安装Docker的说明进行操作，但我在下面给出了重点:
    
    1. 安装软件包以允许apt通过HTTPS从存储库获取更新：
    
        ```
        sudo apt-get install \  
        apt-transport-https \
        ca-certificates \
        curl \
        software-properties-common
        ```
    
    2. 添加Docker的官方GPG密钥。这用于确保下载的包可以被信任。
    
        ```
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -  
        ```
    3. 设置稳定的Docker apt存储库，以便我们可以下载Docker包
    
        ```
        sudo add-apt-repository \  
       "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
       $(lsb_release -cs) \
       stable"
        ```
    4. 运行另一个`apt-get`更新，以从稳定的Docker apt仓库中获取软件包列表.
    
        ```
        sudo apt-get update  
        ```
    5. 使用`apt-cache`检查Docker的版本

        ```
        apt-cache madison docker-ce  
        ```
        这将显示可用软件包的列表，版本号显示在第二列中。我选择了第二个下来，`17.06.2~ce-0~ubuntu`
        ![](http://img.aiuuwe.com/15190403276894.png)
    6. 安装特定兼容的Docker版本
        
        ```
        sudo apt-get install docker-ce=17.06.2~ce-0~ubuntu  
        ```
    7. 通过`docker--version`检查安装
    
        ```
        docker --version  
        Docker version 17.06.2-ce, build cec0b72  
        ```
        现在我们有一个运行Docker的主机，我们可以开始安装Rancher！
3. 安装Rancher
    正如我提到的那样，Rancher在Docker中运行，所以安装Rancher服务器就像运行Docker容器一样简单：
    `sudo docker run -d --restart=unless-stopped -p 8080:8080 rancher/server:preview  `
    如果您现在在虚拟机中导航到http：// localhost：8080，您会看到Rancher服务器欢迎页面 - 很简单！
    ![](http://img.aiuuwe.com/15190406501001.png)
4. 添加一个主机并且安装k8s
    一旦Rancher运行，接下来要做的就是用它管理一些东西。 例如，如果您已经在Azure服务或Google Container Engine上运行了Kubernetes群集，则只需将此群集导入Rancher即可。 如果你这样做，Rancher不管理主机，但你可以在一个地方查看你的节点的状态。
    相反，我们将添加我们自己的主机，并使用Rancher在上面安装Kubernetes来为我们完成复杂的工作。
    
    1. 点击添加主机。 这为您提供了多种选择供您选择希望运行Linux主机的容器。 你可以启动一个EC2实例或者一个DO液滴，但是在这种情况下，我将添加与主机相同的VM，所以选择Custom。 这在生产中可能不是一个好主意，但它对我来说在本地是一个很好的实践.
    ![](http://img.aiuuwe.com/15190410642241.png)
    2. 输入主机的IP地址。您需要所有主机都能够相互通信 - 我刚刚输入了虚拟机的IP（使用`hostname -I`找到）
    ![](http://img.aiuuwe.com/15190411342183.png)
    3. 当你点击保存时，你会得到一个可以在linux主机上运行的脚本。
    
        ```
        sudo docker run --rm --privileged -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/rancher:/var/lib/rancher rancher/agent:v2.0-alpha4 http://192.168.178.129:8080/v3/scripts/E3C8476FE2408EA7070A:1464462400000:4RkyHoXcbZmWNaKEvOJeZ6bE9A  
        ```
        在主机上运行（在我们的例子中是同一个VM）。 这会启动主机上的Rancher代理，并包含一个密钥，以便它可以通过Rancher服务器进行身份验证。 一旦成功启动，您应该看到一条消息出现在底部，显示“一个新主机已注册”
        ![](http://img.aiuuwe.com/15190412909854.png)
    4. 等待一切启动。 一旦添加主机，Rancher将在其上安装Kubernetes，并开始监控主机的健康状况。 这会很慢，等待代理将所有进入Kuberentes集群的组件运行起来！
       您可以通过从右上角菜单切换到系统环境（默认情况下启动）来查看进度：
       ![](http://img.aiuuwe.com/15190414617114.png)
这显示了你的所有容器，所以你只需要等待一切变为绿色：
![](http://img.aiuuwe.com/15190414953803.png)
一旦你有了绿色，你就可以开始运行Kubernetes！如果忽略设置虚拟机和安装Docker的步骤，那么这实际上只需要2个Docker命令。确实很简单。

    5. 查看Kubernetes仪表板
        为了验证我们确实拥有了一个Kubernetes集群，让我们来体验一下。在“容器”页面上，单击“高级”选项卡。这给你下面的选项：
        ![](http://img.aiuuwe.com/15190417778863.png)
    如果您单击启动仪表板，Rancher会打开您的集群对应的Kubernetes仪表板，恰好展示了您确实在运行您自己的Kubernetes群集。
    ![](http://img.aiuuwe.com/15190418840783.png)

## 总结
在这篇文章中，我提供了一个Rancher快速入门文档，以及它适用于Docker和Kubernetes生态系统的地方。 我展示了如何将Rancher 2.0部署到本地VM，以便快速获得Kubernetes的运行。 到目前为止，我只体验了以部分它的功能，但我真的很喜欢它的这些特性。我会尽快的发布一些接下来的相关文章




## 参考资料

- [Rancher 2.0快速上手指南](https://segmentfault.com/a/1190000011487674)
- [译自installing-kubernetes-using-rancher-2-0](https://andrewlock.net/home-home-on-the-range-installing-kubernetes-using-rancher-2-0/)
- [Rancher overview (version 1.x)](http://rancher.com/docs/rancher/v1.6/en/)
- [Rancher 2.0 preview](http://rancher.com/rancher2-0/)
- [Quick start guide for Rancher 2.0](http://rancher.com/docs/rancher/v2.0/en/quick-start-guide/)
- [Install Docker CE](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)



