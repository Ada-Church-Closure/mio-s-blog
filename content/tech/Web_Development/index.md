+++
date = '2025-08-01T12:05:12+08:00'
draft = false
title = 'Web_Development'

+++

# Web_Development

> ​	本文是关于**web开发导论**的一些内容，主要是为了对于web开发有一个现代并且全面的认知，说来惭愧，笔者现在大二已经结束，但是对于web开发还是没有一个深刻的认知，仅仅停留在写过一些前端和Java代码上面，从这里作为起点，我们来开始对于Web开发的探索，希望能成为我dev的指明灯。
>
> ​	这里就仅仅是**通识性**的知识。仅作个人笔记使用。
>
> ​	“什么是”，但是不是“怎么用”。

> ## Part 1: 启蒙与巨石时代

###  1.B/S架构软件与Web开发的初步认识

> 简单对于前端学习可以看freecodecamp，很不错。（但是前端本身是很复杂的）
>
> 必须经过系统学习。

C/S	B/S---》通用：浏览器（**Browser/Server**）

HTML---》**基本**网页内容和**结构**	CSS（级联样式表）在HTML中选择元素进行设计	Javascript---》真正的编程语言，网站的**交互**（用户和浏览器，浏览器和服务器）

早期 **jQuery**

### 2.前后端交互的初步认识

> 历史记录，登录验证这样的处理是怎样做到的？
>
> 用户之间是怎么加好友发消息的？

### 后端：主要和数据处理相关

1.服务器

2.编程语言：Java php golong ruby rust python Node.js

3.框架：spring spring boot flask gin bego rails

4.数据库：CRUD **MySQL** MongoDB Redis---》非关系型，主要用于缓存

5.Linux，可能包含运维相关的东西。

6.API：应用程序编程接口。

### 3.单体架构应用

> 一个文件内部实现前后端，每次更改重新构建项目。

1.混沌时期：

##### **2005**：博客，论坛，早期电商

单体架构应用（**Monolithic Architecture**）

一个软件的所有功能都在一个单元内部。前后端没有分离。---》php一个项目

技术栈：**LAMP/WAMP**组合。

L/W：**Linux**服务器还是Windows服务器。

A:Apache，主流web服务器。负责接收来自浏览器的http请求，并且转发给后端的程序。（HTTP Server）

P:PHP，调用Apache服务器，或者py（不适合做web开发）/Perl。

**工作流程:**

1.浏览器向服务器发送一个请求。GET/ Products?id=123

2.ApacheServer收到请求，发现是PHP请求，商品123的信息。

3.PHP脚本开始执行，根据数据和预先写好的**html模板**(静态的)进行混合渲染，动态生成一个完整的页面（不同用户网站数据不相同，定制的,就比如B站的个人主页）。

4.这个文本返回给**Apache**，相应返回给用户的浏览器。

5.在用户的浏览器上进行渲染。JS做一些动画之类的效果。

PHP：脚本，在html内部写php语言（前后端混在一起了），**jsp**是类似的，直接就可以和数据库进行交互之类的操作。

功能复杂，模块化。所以php和jsp基本已经被淘汰了。

### 4.前后端分离的进程和技术的演进

##### 2004-2010：前后端分离的萌芽期

​	**Ajax**：一种用js实现的技术。**Asynchronous**：异步，可以不用请求整个页面就可以局部维护或者更新数据。

​	**XML**：一种数据格式。

​	现在用**JSON**：Javascript object notation

##### 2010-2014：标准的前后端分离

**SPA**：单页面应用。

**Web API**的标准：前端和后端之间通信的标准和规范。

**restful API**：

​	API：application programming interface

​	1.请求什么数据2.以什么格式请求3.返回什么格式的数据

**REST**：REpresentational state transfer	表现层状态转移

所有东西都是资源	资源都有唯一标识符，就是**URL**

这样的一个“链接”就是**URL**。

```url
https://ja.wikipedia.org/wiki/%E3%83%A1%E3%82%A4%E3%83%B3%E3%83%9A%E3%83%BC%E3%82%B8
```

表现层：

JSON	XML	HTML	在Client和Server之间转移的形式

状态转移：

资源发生变化的过程：比如用户发起一个删除的操作。

login.jsp：已经淘汰了。

计算机网络：**Cookie**工作原理。每当有一个用户的时候，就要创建一个对象。

不用session的原因。（反而会使效率变低）

负载均衡，分担流量。

**restful API**	无状态

**六大约束：**

1.客户端和服务器**必须相互独立**。

2.**无状态**	server不能存储任何**session**	为什么学**Spring**

​	所有请求都必须包含服务器所需要的全部信息。

3.**可缓存**

4.**统一接口** uniform interface

​	资源标识符	JSON，用**表现层操作资源**

​	自描述信息	**HATEOAS**，**超媒体**作为应用状态的引擎

5.**分层系统**

​	Client不知道自己具体和谁在通信。

6.**按需代码**

### 5.HTTP,RESTful API,Token,GraphQL

> 介绍相关的概念。

**Http Verbs**:一次行动的目的。GET POST......

> 我们用汇编写了一个Server，对于这些理解的就会很深刻了。

你要对于地址做什么动作，获取信息还是提交表单。

CRUD：最早是http verb。

GET：获取信息。

POST：创建。登录，注册。

PUT：更新替换。

DEL：删除。

Patch：局部更新。

实际企业使用只使用POST和GET。不用物理删除，用**软删除**，标记状态即可。

Cookie：不安全。不用Session和Cookie，保存状态。

中间人攻击，流量劫持。

现代使用**Token**。JWT技术。（JSON Web Token）令牌格式。

**授权协议**

**OAuth 2.0**

1.访问令牌 JWT

2.刷新令牌



**认证协议**

OpenID connect OIDC



**身份协议**

ID TOKEN

#### 2015至今

React Vue.js的诞生 **Angular**

Node.js

**GraphQL**:架构风格。解决什么问题？

> 流行技术。
>
> 比较有意思。

/users/123	返回了很多的数据，数据过载

/users/123/posts?limit=5	多次请求

**精确确定**需要什么数据，不会过载。

一次请求完成多个数据。

For example, the query:

```GraphQL
{
  me {
    name
  }
}
```

Could produce the following JSON result:

```GraphQL
{
  "data": {
    "me": {
      "name": "Luke Skywalker"
    }
  }
}
```



前端工程化。

四驾马车：

1.包管理器（库很多）：NPM，Yarn安装第三方库

2.构建工具：**webpack**，并行开发，技术栈独立演进，职责边界清晰。

​	整合工具。

3.框架，库	**react vue**

4.编译器/转译器：比较新。**ECMAScript**这是一种设计的规范。

> 接下来是前后端分离的实际演示。  

服务器软件。

前几天学了web安全，理解起来比较轻松，通识课还是比较简单。

### 6.后端框架Java与Spring全家桶的时代演进

过去很笨重 Servlet API -> Java Web J2EE

#### **Spring全家桶**

打包成war包，然后部署到Tomcat上面去	重量级并且繁琐

Spring Boot	

2004年：**Spring Framework**诞生

1.IOC：控制反转	容器	DI注入

2.AOP：面向切面编程	处理权限问题



Spring MVC（model view controller）一种设计上的方法	

专门处理web请求 http verb

Controller：控制路由	复杂业务就是Service，比如操作数据库的内容

配置非常繁琐，很复杂，那么就有：



**Spring Boot**---配置说明书，帮你组装配件	2014年，很多大项目都使用

> 约定优于配置。优点是稳定，为微服务打下了基础。

> ## Part 2: 工业革命与未来之光

### 7.微服务架构与分布式系统的时代历程与技术演进

#### 微服务革命	

> 感谢你带来更多的就业岗位。（）

原来是单体架构

2015至今	业务需求演变

​	不是简单的一个应用，分为很多的服务模块，比如B站的直播应用就是一个单独的应用，以及某某区，我们都要把这些东西分开。

微服务：

> 三独立原则。

1.独立开发	不同团队负责不同的模块，有100%控制权

2.独立部署：最大的优势

3.独立数据存储，每个微服务都有自己的数据库，不能跨数据库查询

不同服务之间通过自己的**API**进行交互。



优势

1.技术的异构性	**每个团队都可以采用不同的技术**

微信小程序也是依赖于微服务的API接口

> 所以我们说**语言和框架**不重要，对于大厂。

性能Rust Golang

稳定，核心：Java Spring

推荐算法，数据科学：Py

2.极致的**扩展性**---外科手术般的精准扩展

3.**容错和隔离**

系统有很强的健壮性，就是因为独立性

局部坏死没有关系	熔断，服务降级



分布式---分布式系统（御坂网络）	网络互联的计算机的集合

Netflix部署

> 莫名奇妙跟风微服务，某些商城项目。
>
> 2019，技术鸡汤年，滥用微服务。拆分不见得是好处。

解决分布式的问题：

如何分配任务？	负载均衡:分担流量。

相互沟通？	网络通信。

出错？	容错，高可用。

版本？	配置管理。

出错？	分布式追踪。



微服务架构是一种**分布式系统**（比如bit coin），一种设计的原则。

集群



三大论文：分布式数据库相关问题。

GFS

MapReduce

bigtable

它们**带来了大数据时代**。

hadoop hbase



云计算：在这些分布式系统上进行计算。

AWS	亚马逊云，按需分配



资源共享和高性能计算。

Spark tensorflow



地理分布，低延迟

CDN 内容分发网络，计算机网络中我们学习过。

#### 简单实现

学生服务	CRUD 3001端口（不能冲突）

课程服务	3002端口

前端应用

API网关---总服务台	3000端口

nmp vite vues node.js快速搭建微服务

#### 补充：RPC和gRPC

不同服务之间不能直接访问数据库，可以用API

这里用RPC，远程过程调用，调用另一台计算机上的方法，不用关心计算机网络，更方便。

同TCP协议。

gRPC用http，更快，效率更高。

.proto文件

#### Nginx反向代理

service a localhost：3001

service b localhost：3002

我要管理这些服务。

访问某个域名的时候，到底选择哪个服务。

### 8.前端工程化的新潮流技术提醒

yarn npm pnpm（新技术）

包管理工具

现在service用ts比较多，比js舒服一些

express 比较老的框架	有simform	Rspack	Bun

Deno

> 反正新东西多得夸张，新的库，框架，层出不穷，日新月异。

CSS---tailwind CSS

编译器：Babal Web

预处理器：sass-lang	less-css

CSS框架：getbootstrap	bulma

前端测试：jest mocha js 

状态管理：redux mobX VueX Zustand

前端安全： CORS（web安全）

静态生成器：hugo，本网站就使用了hugo来搭建

**Vercel**：服务器部署---趋势（重点）

前端监控：sentry---体量很大

前端代码质量：eslint prettier

文档生成：storybook gitbook

前端组件库：Ant material（md风格，很纯的安卓） element ui

前端动画：gasp framer motion anime.js

前端图表库：d3.js chart.js echarts

前端模块化：es modulws common.js AMD front

前端**跨平台**：react native重置过的项目很多 **Flutter**（感觉更先进）---趋势 **Dart**

PWA 渐进式web应用

### 9.再谈后端的发展趋势和技术词

小公司：前后端都用js或者ts。---全栈工程师，技术很杂，很依赖个人能力和团队规范，不适合CPU密集型任务。ts解决这个问题，js的超集。

**Spring**全家桶：超级航空母舰，更可靠。

> 这取决与应用场景。

**Spring Cloud**：服务发现，API网关，声明式http客户端，熔断器，分布式配置中心

**Python**？语法简单。Django，Flask FastAPI

效率更低	全局解释器🔒	在有的微服务项目中也会大量的使用

websockets RabbitMQ Rocket MQ

webhooks	服务器之间事件回调的机制

ORM	对象关系映射	和Java合作的**数据库框架** mybatis不用JDBC

Prisma python数据库	typeorm	sequelize

改不了字段怎么办

后端测试：pytest unit testing mocha integration apitest json **Postman**（不能不会）

web server：Nginx

### 10.云原生时代的演进与发展，容器化，容器编排，DevOps：Golang的时代潮流在哪里？

#### 云原生浪潮和现代工程化

容器化技术	解决很难部署的问题，大量服务器，还有版本的问题

环境隔离的解决方案---虚拟机

物理机---虚拟机

VM hypervisor在服务器模拟计算机

那么每一个虚拟机都要一个完整的OS，这是很麻烦的，我们就迎来了容器。

#### **2013：Docker 容器**	

我们也学习过一些关于docker的内容

> 轻量级，容器镜像不包含操作系统内核。

 Linux namespaces	命名空间，认为自己是隔离的

docker仓库的概念，什么都有，拉取镜像。

#### 容器编排

**Google Kubernetes k8s**

理念：提供一种声明式的工作方式。

**控制循环**：当前状态和期望状态进行比较，高级部署策略，自动扩容，自动修复。

存储化编排。

管理微服务集群的解决方案。

#### CI/CD DevOps

CI：持续集成，开发者每天会把自己的代码自动合并和build。

CD：持续交付/部署。

DevOPs：**筒仓效应**。(Soli Effect)---过度分工

> YOU BUILD IT, YOU RUN IT.

#### **Golang**

最耀眼的就是多并发场景。

Goroutine：超轻量级线程。数百万个都是有可能的。

实现非阻塞的高并发。跨平台编译。性能之王。

​	B站主站的微服务架构（有1600个微服务，全部由Golang实现），Bangumi，Youtube，适合直播，庞大的流量。

​	Kratos，B站的开源项目。

​	Docker引擎就是Go写的。

接近C，Cpp但是很安全和方便。

Gin	Fiber	Go的框架

### 11.元框架技术形式的当下与未来

元框架的大一统

Next.js->React

Nuxt.js->Vue.js

1.混合渲染模式

CSR 仪表盘，数据频繁变动---可以自行决定前端还是后端进行渲染

SSR 服务端渲染

数据响应式



SSG 静态站点生成

ISR 增量静态再生	比如动态墙

2.*API路由，后端的内化

文件夹即路由，即API

### 12.Serverless与边缘计算的趋势与未来

物理机	所有东西都要自己配置

虚拟机	买云服务器 + 域名，变得更简单

容器化

Serverless无服务器	开发者不应该担心运维问题

FaaS 函数即服务---服务器没有使用就不收费，只有function在工作的时候才会收费（事件驱动）

可以弹性伸缩。

场景：实时处理，AI生成内容，物联网比较重要。问题是状态的保存，要数据库。

BaaS 后端即服务，只用调用平台提供的SDK即可，不需要自己安装服务。

**supabase**



**边缘计算**---光速终究有限

游戏加速器的实现

中心仓库	CDN内容分发

**Vercel**很不错

> ​	AI开发可以，但是在可预见的未来，公司的实际项目不会使用，或者最多只能参与测试的一小部分。辅助，但不是全自动化的。

### 13.版本控制（VCS）的发展历程和git

> git是怎么实现的？

追踪记录

版本回溯

协同工作---多个开发者工作

备份和恢复



1.Local VCS本地版本控制---追踪代码和文件变更，不支持多人开发

SCCS Source Code Control System

七十年代 贝尔实验室

RCS Revision

2.集中式 VCS

CVS 八十年代 不支持原子提交

**SVN** Subversion 版本控制的内容更好了 2000年 中央服务器单点故障的问题

3.分布式 DVCS

2005年 git

Linus

每个开发者有完整的代码仓库，有完整的历史记录。

本地进行，速度快，强大的分支模型。

数据完整性。

托管代码仓库平台 Github 2008年上线，塑造了现在的基本盘

> 不会就直接查文档，git写的很nb，有问题直接查。

### 14.依赖管理的意义

第三方库	拓扑排序的感觉

你引入的库相互依赖，且相互依赖的版本还不一样。

**手动管理会带来灾难。**

依赖管理器 自动化工具

查找

Manifest File 清单文件

Lock file 锁定文件

> ​	总之，我们已经理解，web开发到了现在已经不仅仅是web开发的问题，而是一个更为庞大的生态，有趣并且复杂。虽然就业很难,但是一开始的学习以兴趣为导向是没错的(我是说你有时间的时候),如果是为了找实习找工作,那就是另一个话题了对么.
