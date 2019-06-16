---
layout: post
title: 'IDEA远程DEBUG'
tags: [read]
---

> 在调试代码的过程中，为了更好的定位及解决问题，有时候需要我们使用远程调试的方法。在本文中，就让我们一起来看看，如何利用 IntelliJ IDEA 进行远程 Tomcat 的调试。

首先，配置remote：

![1](../images/idea/20171111163857328-1560677959918.jpg){:.center}

如上图所示，点击Edit Configurations，进入如下界面：

![2](../images/idea/20171111164103227.jpg){:.center}

如上图所示，我们进入了Run/Debug Configurations界面，然后点击左上角的+，选择Remote：

![remote](../images/idea/20180103164200755.jpg){:.center}

- 标注 1：运行远程 JVM 的命令行参数；
- 标注 2：传输方式，默认为Socket；
- 标注 3：调试模式，默认为Attach；
- 标注 4：服务器 IP 地址，默认为localhost，需要修改为目标服务器的真实 IP 地址；
- 标注 5：服务器端口号，默认为5005，需要修改为目标服务器的真实端口号；
- 标注 6：搜索资源是使用的环境变量，默认为<whole project>，即整个项目。

如上图所示，其中 标注 2 和 标注 3 又分别有两种分类，对于 标注 2，

- 标注 2：传输方式，默认为Socket； 
  - Socket：macOS 及 Linux 系统使用此种传输方式；
  - Shared memory： Windows 系统使用此种传输方式。

对于 标注 3，

- 标注 3：调试模式，默认为Attach； 
  - Attach：此种模式下，调试服务端（被调试远程运行的机器）启动一个端口等待我们（调试客户端）去连接；
  - Listen： 此种模式下，是我们（调试客户端）去监听一个端口，当调试服务端准备好了，就会进行连接。

然后，复制 标注 1，即 IntelliJ IDEA 自动生产的命令行参数，然后导入到 Tomcat 的配置文件中。以 Linux 系统为例，导入语句为：

- export JAVA_OPTS='-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8001'
  如果是 Windows 系统，则导入语句为：
- set JAVA_OPTS=-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8001

两者的区别在于导入语句的关键字不同以及有无引号，Linux 系统的导入关键字为export，Windows 为set；Linux 的导入值需要用单引号''括起来，而 Windows 则不用。

接下来，修改 Tomcat 的 bin 目录下的catalina.sh文件（如果是 Windows 系统则修改catalina.bat文件），将上述的导入语句添加到此文件中即可：

![cata](../images/idea/20180103164534540.jpg){:.center}

至此，IntelliJ IDEA 远程调试 Tomcat 的配置已经完成了，调试的后续步骤按正常的调试技巧进行就可以啦！

题外话：

　　在这里，我们假设服务器的 IP 地址为10.11.12.39，端口号为16203，设置完成后，进入Debug模式，如果连接成功，则会出现如下提示：

![5](../images/idea/20171111170443363.jpg){:.center}

此外，如果我们是跨多个系统进行调试，则只需要在想要调试的系统中配置Remote，打上断点，启动Debug模式，然后在服务开始的地方执行程序即可进入到我们设置的断点。而且，如果我们在本地配置Remote并关联到某个 Tomcat，在Debug模式下，所有涉及到断点所在代码的功能，都会进入我们设置的断点。

例如，对于服务器上的 Tomcat A，多个系统都用到了这个 Tomcat，如订单子系统、账户子系统、路由子系统等，并且多个系统间互相调用，如订单子系统调了账户子系统，账户子系统又调了路由子系统，则当我们在这三个子系统中配置Remote并在对应的代码（如在订单子系统中查询商户的账户信息，则调到账户子系统；在账户子系统中又通过路由子系统调到其他底层服务查询商户的账户余额等）上打上断点，启动Debug模式之后，通过单元测试或者页面操作触发订单子系统中的查询商户的账户信息功能，则会依次进入到在上述三个子系统中设置的断点。

此外，在我们配置完远程调试之后，就算别人启动相关服务，也会进入到我们的断点，而且会受到我们设置的断点的影响，只有在我们执行完测试之后，服务才会继续执行下去。最后，远程调试的功能真的很强大，善用远程调试，远离 Bug！

其他参考文章：

<https://blog.csdn.net/mingjie1212/article/details/52281847>