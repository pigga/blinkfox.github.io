---
title: 初窥微服务
date: 2019-04-15 23:41 +0800
layout: post
Author: Yannis
permalink: /blog/2019/04/15/microservice.html
categories:
  - 微服务
tags:
  - 微服务
---
#### 初窥微服务

在当前的这个年月,微服务已经不是一个新的事物.不过我觉得还是应该写几篇文章,整理一下个人对于微服务的一些认识.

##### 什么微服务
以下内容来自[维基百科](https://en.wikipedia.org/wiki/Microservices)
> **Microservices** are a [software development](https://en.wikipedia.org/wiki/Software_development "Software development") technique—a variant of the [service-oriented architecture](https://en.wikipedia.org/wiki/Service-oriented_architecture "Service-oriented architecture") (SOA) architectural style that structures an [application](https://en.wikipedia.org/wiki/Application_(computing) "Application (computing)") as a collection of [loosely coupled](https://en.wikipedia.org/wiki/Coupling_(computer_programming) "Coupling (computer programming)") services. In a microservices architecture, services are [fine-grained](https://en.wikipedia.org/wiki/Service_granularity_principle "Service granularity principle") and the [protocols](https://en.wikipedia.org/wiki/Protocol_(computing) "Protocol (computing)") are lightweight. The benefit of decomposing an application into different smaller services is that it improves [modularity](https://en.wikipedia.org/wiki/Modular_programming "Modular programming"). This makes the application easier to understand, develop, test, and become more resilient to architecture erosion.<sup>[[1]](https://en.wikipedia.org/wiki/Microservices#cite_note-Micro_Chen-1)</sup> It parallelizes [development](https://en.wikipedia.org/wiki/Software_development "Software development") by enabling small autonomous teams to develop, [deploy](https://en.wikipedia.org/wiki/Software_deployment "Software deployment") and scale their respective services independently.<sup>[[2]](https://en.wikipedia.org/wiki/Microservices#cite_note-2)</sup> It also allows the architecture of an individual service to emerge through continuous [refactoring](https://en.wikipedia.org/wiki/Refactoring "Refactoring").<sup>[[3]](https://en.wikipedia.org/wiki/Microservices#cite_note-Ach_Chen-3)</sup> Microservices-based architectures enable [continuous delivery](https://en.wikipedia.org/wiki/Continuous_delivery "Continuous delivery") and deployment.<sup>[[4]](https://en.wikipedia.org/wiki/Microservices#cite_note-4)</sup>

以下内容来自[百度](https://baike.baidu.com/item/%E5%BE%AE%E6%9C%8D%E5%8A%A1/18758759)

>所谓的微服务是SOA架构下的最终产物，该架构的设计目标是为了肢解业务，使得服务能够独立运行。微服务设计原则：1、各司其职 2、服务高可用和可扩展性

综合以上,微服务(MicroServices)就是微小紧凑的服务, 提供统一简捷的 API 供外部访问, 实现一组独立的功能.可能还是觉得有些模糊.不过没关系,后续应该会逐步的清晰.
##### 微服务的历史
2011年5月在威尼斯附近举行的一个软件架构师研讨会使用了“微服务”这个术语来描述参与者所看到的一种共同的架构风格，他们中的许多人最近都在探索这种风格。2012年5月，该组织决定将“微服务”作为最合适的名称。2012年3月，James Lewis在Kraków的第33届Microservices (Java, Unix方式)课程上展示了其中的一些想法作为一个案例研究。Netflix的Adrian Cockcroft将这种方法描述为“细粒度SOA”，他在web规模上开创了这种风格.至此开启了微服务统领的架构时代.
##### 微服务发展背景
看了上面的那些内容,那么势必有个疑问,为什么是微服务,而不是其他来承接了后续的架构设计?
为了明白这个问题,我们简单的了解一下微服务架构出现之前的架构设计形式.

在 2000 年初，面向服务架构（Service Oriented Architecture，SOA）是一种非常流行的软件架构设计范式。简而言之，SOA 是一种软件架构模式，用于构建大型的企业应用程序，这些应用程序通常要求集成多种服务，而每种服务使用不同的平台和编程语言来构建，并通过通用的通信机制进行交互。

以下是**面向服务架构（SOA）**的简单图示：
![SOA架构图](http://ppmzbzfyf.bkt.clouddn.com/1.png)


关键点

1.  SOA 是大型软件产品（如企业应用程序）的首选。<br/>
2.  SOA 侧重于将多个服务集成到单个应用程序中，而不是强调模块化应用程序。<br/>
3.  在 SOA 中，用于服务间交互的通用通信机制被称为企业服务总线（Enterprise Service Bus，ESB）。<br/>
4.  基于 SOA 的应用程序本质是单体。也就是说，单个应用程序层包含了用户界面或表示层、业务逻辑或应用程序层，以及数据库层，这些全部都集成到一个平台中。<br/>

随着业务的发展,我们发现有一些比较严重的问题

###### 1、代码重复率高
1）从技术架构角度看，传统垂直架构的特点是本地API接口调用，不存在业务的拆分和互相调用，使用到上面功能就本地开发，不需要过度依赖于其他的功能模块。<br/>
2）从考核角度看，共享很难推行。开发只需要对自己开发的模块交付质量负责，没有义务为他人提供并维护公共类库，这个非常耗费成本。<br/>
3）时间依赖很难把控。对于公共类库的使用者而言，依赖别人提供此功能，但是功能提供者可能有更重要的事情在做，交付时间无法满足使用者。与其坐等别人提供，还不如自己开发效率更高。<br/>
4）跨地域、跨开发小组协调很困难，业务团队可能跨地域研发，内部通常也会分成多个开发小组，各个开发小组之间的协调和沟通成本非常高。<br/>
功能的重复开发会导致研发成本的上升，代码质量的下降，架构腐蚀，为后续系统的运维和新功能的开发带来了巨大的挑战。<br/>

###### 2、需求变更困难  

代码重复率变高之后，已有功能变更或者新需求加入都会非常困难，以充值缴费功能为例，不同的充值渠道开发了相同的限额保护功能，当限额保护功能发生变更之后，所有重复开发的限额保护功能都需要重新修改和测试，很容易出现修改不一致或者被遗漏，导致部分渠道充值功能正常，部分存在Bug的问题，如下图所示：

 ![原始版本](http://ppmzbzfyf.bkt.clouddn.com/d1.png)

   修改不一致导致的移动APP充值失败示例，如下图所示

![修改不一致版本](http://ppmzbzfyf.bkt.clouddn.com/d2.png)

图--充值限额保护功能需求变更后

###### 3、代码维护困难

在传统的MVC架构中，业务流程是由一长串本地接口或者方法调用串联起来的。臃肿而冗长，而且往往由一个人负责开发和维护，随着业务的发展和需求变化，本地代码再不断的迭代和变更，最后形成了一个个垂直的功能孤岛，只有原理的开发者才能理解接口调用的关系和功能需求，一旦原有的开发者离职或者调到其他项目组，这些功能模块的运维就变得非常困难，如下图所示：

 ![image](http://ppmzbzfyf.bkt.clouddn.com/d3.png)

图--垂直架构导致的功能孤岛

###### 4、部署效率低

部署效率低主要体现在以下三个方面：
1） 业务没有拆分，很多功能模块都能打到同一个war包中，一旦有一个功能发生变更，就需要重新打包和部署。<br/>
2）巨无霸应用由于包含功能模块过多，编译、打包时间比较长，一旦编译过程出错，需要根据错误重新修改代码再编译，耗时较长。<br/>
3）测试工作量较大，因为存在大量重复的功能类库，需要针对所有调用方法进行测试，测试工作量大，另外，由于业务混在一起打包，需要针对集成打包进行专项测试，客观上也增加了测试工作量。<br/>

基于以上问题,微服务的出现不可避免,那么微服务有那些优势呢?
##### 微服务的优势
微服务的核心竞争力:更小,更快,更强
###### 更小
更小意味着单个应用的体积小,覆盖的功能也相对比较小。也就是通常说的服务解耦。有一个一般的原则是，单一服务提供的功能是可以独立被替换和升级的。 也就是说如果有 A 和 B 两个功能，如果 A 功能发生变化时同时 B 功能也需要变化，那么 A 和 B 这两个功能应该被划在一个服务中。 但这些服务可独立于其他服务或整个应用程序本身而构建、部署、伸缩和维护。
![image](http://ppmzbzfyf.bkt.clouddn.com/d5.png)
基于服务注册中心的订阅发布机制，可以实现服务消费者和提供者之间的解耦，消费者不需要配置服务提供者的地址信息，即可以实现位置无关的透明化路由，它的开发体验与本地API接口调用相似，但是却实现了远程服务调用。通过服务注册中心，可以管理服务的订阅和发布关系，查看服务提供者和服务消费者的详细信息。有了服务订阅关系，业务和服务之间的调用管理系统变得透明化，不合理的接口依赖、调用关系一目了然。
###### 更快
更快主要体现在以下三个方面：
1） 开发速度快,基于微服务的架构,使得单个服务更集中于单一的功能实现,便于开发人员进行功能迭代及新的开发人员快速熟悉微服务对应的代码。<br/>
2）部署时间快,小微型的应用,对其他应用没有直接的依赖关系,自然启动速度会比较快。<br/>
3）迭代速度快,微服务的架构,使得单点可容错的机会变大,单点的改动也相对来讲比单体的应用更快。<br/>
###### 更强
不是所有系统都打算持续存在。它们在需要时创建，在不适合某个用途时就会被删除。正如在上面提到的。微服务的架构使得服务与服务间保持这松耦合,减少了关联故障的影响。

正是基于微服务的这些特点作为基础,才有了现在比较流行的DevOps。对于DevOps,有机会的话在后续会整理一片文章来进行说明。
##### 微服务特点总结
微服务有如下的特点：<br/>
1）单一职责原则：每个服务应该负责单独的功能，正是SOLID原则之一。<br/>
2）独立部署、升级、扩展和替换：每个服务都可以单独部署及重新部署而不影响整个系统。这使得服务很容易升级，每个服务都可以沿用着 “Art of Scalability” 一书定义的X轴和Z轴进行扩展。<br/>
3）支持异构/多种语言：每个服务的实现细节都与其他服务无关，这使得服务之间能够解耦，团队可以针对每个服务选择最合适的开发语言、工具和方法。<br/>
4）轻量级：微服务通常有轻量级的分布式服务框架承载，采用了P2P通信，无中心节点，性能可以线性增长；第三方软件依赖减少，减少类冲突和冗余依赖，集成和升级更方便。<br/>
##### 微服务与容器化
微服务将单体应用拆分成很多小的应用，因而运维和持续集成会工作量变大，而容器技术能很好的解决这个问题。关于该部分的详细内容,后续会增加相关文档进行说明。
##### 微服务的划分
1）一个单独的微型服务，尽量保证纯粹，良好的封装性。良好的封装表示不会对外暴露自身的细节。<br/>
2）微型服务的组织尽量保证有一定的层次性，不要胡乱的或者网状的组织微型服务，不要出现胡乱依赖，循环依赖。<br/>
3）微型服务切记不要划分的太细，也不要划分的太粗。具体那个度适中，这个需要根据具体的case进行思考。<br/>
4）每个微型服务都应该时刻警醒===>自己依赖的服务。每个这些外部服务的健康状况都会对你的服务产生直接影响。同时你要经常梳理自己的服务的依赖，如果不是真正的必要，不要引入别的服务的依赖。<br/>
##### 微服务的选择
综合以上的了解,我们发现微服务简直是一个必须的选择啊。但是是否我们需要立马拥抱微服务,还是需要慎重。
我们先来看一下传统的单体应用与微服务的优缺点对比图。<br/>
![image](http://ppmzbzfyf.bkt.clouddn.com/dff1.jpg)
通过以上对比,我们发现虽然单体有各种问题,但是单体还是有其自身的优点。另外我们需要考虑单体架构列出的一些问题是否已经严重影响了我们的业务?<br/>
企业新的业务系统是否要满足快速迭代、弹性等需求?<br/>
团队内是否有 DevOps 氛围?<br/>
企业内是否有足够的动力和技术储备去接触新的技术?<br/>
了解了单体应用和微服务应用的优劣特点，分析了企业自身的业务诉求和实际情况，最终还是决定转型微服务架构，那么我们也要清楚这不是一朝一夕的事情，需要分阶段逐步推进。<br/>