---
layout: post
title:  "一次记忆深刻的线上排错"
date:   2019-07-16 22:59:53 -0700
categories: tip
tag: [AWS, spring boot]
---

### Tips

才换工作不久以后，就遇见了一次记忆深刻的线上排错

*ENV*: `AWS EC2 Instance`, `ECS deploymnet`, `Conatainers of Java Microserivices`, `MariaDB`, `Rabbit MQ`

*Deployment*
- 10 core microserices
- 3 container node per services, scale based on memory usage
- （Spring Boot + AWS） Infrastructure services
- Backed by 25 physical machines initially

*背景:* 

公司业务会源源不断将客户从某个老系统移动到新系统->10个主微服务中，上线尤一个data sync service负责。 遇见超大客户一半会在周末实行上线，将他们几十年的庞大数据进行intial转移，会耗费2～3天时间。成功上线以后会每30秒进行一次新老系统数据更新，使得客户能在新老系统中无缝切换，最终完全转移到新系统。问题的隐患就在于初始设计data sync service的时候默认客户大小都差不多，超庞大客户的30秒之内能产生的业务数据量埋下了一个炸弹。

*症状*：

从早晨7点开始，EC2 instance memory 警告开始在slack里响彻，ECS发现data sync service OOM, 就毫不犹豫重启container, 结果发现下一个instance继续OOM，便一个接一个不停重启。服务宕机。

- 纠察的第一个10分钟： 肯定是全程吓尿的。
- 纠察的第一个15分钟： 翻看上一个Sprint到上线之间的新feature，发现都是无关痛痒的修改，确认是新上线大客户造成的问题，所以回滚和重启不会有任何的作用。Deploy历史上内存警告时常发生，但是OOM并且Kill Container是第一次看到。史无前例，没有前车之鉴。决定要辞职。
- 纠察的第一个30分钟：开始冷静思考，OOM不可怕，但是Kill Container就不对了，ECS发现应付不过来应该Scale新的instance。在同事大神提醒下，开始看container dockerfile, 发现了解决燃眉之急的一个点：运维在写base java container的时候用了
```-XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=1 ```, 这个会让Container去索取目前JVM所能给的所有内存，但是在Container环境下这个是不现实的，所以在高load的情况下会瞬间发生OOM，ECS发现以后就会重启。谷歌大法，更新Java Version， 替换了container friendly的JVM args： ```-XX:+UseContainerSupport -XX:MinRAMPercentage=75.0 -XX:MaxRAMPercentage=75.0```这个会限制container的最高索取内存的比例。
- 纠察的第一个小时：rebuild, hotfix push, prod push。ECS 终于不滥杀无辜了，也把核心问题暴露了出来：data sync service是message driven的架构，在监视RabbitMQ channel的时候发现之前有一条消息永远无法被consume，被尝试了无数次，每次都造成OOM，于是便被下一个container pickup， 便造成死循环OOM。估计是一个体量很大的sync job。但是这个是RabbitMq的一个缺陷，无法set多次access某个病态message无果以后的行为。如果要彻底修复，需要更换新的MQ。
- 纠察的第一个星期：
  - 开会归类出了会造成庞大耗时的sync job的种类，将其改写成一串batch access
  - devOps方面:
    - 开始implement MariaDB read/write separate，以此针对一些写的不严谨的只读service controller endpoint上强制了```@Transactional(readOnly = true)```
    - ECS设定给data sync service特供的back physical machine

（小组向上提出更改MQ的时候，遭遇了上方神秘力量的阻拦，一个production issue，甚至探出了公司的神秘政治文化）

从此往后，到写这篇博客的今天，OOM没有再出现过，内存使用警报也不再有了。美滋滋