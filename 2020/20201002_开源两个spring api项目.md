
### 开源两个spring api项目

```
  工作也有五年有余了，中间一直迫于时间和能力没法从零开始构建一个完整的项目，实在太过于遗憾。
  现在，我决定把这个遗憾弥补上了，尽管这些并不是很完美，尤其是组件在实际业务需求的时候也没法尽善尽美，不过这些总会有个循序渐进的阵痛期
不过我已经做好准备，同时也希望在这条路上有更多的愿意分享的同行，在这里我先感谢哈。现在，Let's start 🏄‍  
```

#### 第一个项目
 
 `这是一个基于springboot2.3的简单api项目，项目主要面向的是对外接口服务，由于api项目的特殊性，所以代码并没有构建页面相关功能`

+ 项目地址
  - [mee-api](https://github.com/funnyzpc/mee-api)
+ 本项目自带的核心功能
  - spring core 核心框架(IOC、AOP)
  - Transation spring事务
  - schedule spring定时任务(可跟进需要开启)
  - Async 异步业务调用(可跟进业务情况开启使用)
  - undertow 基于nio的高性能web容器
  - 基于Mybatis的Dao框架(本项目并没有通过接口代理的形式使用)

+ 本项目拓展封装功能
  - Jackson序列化功能
    - `JacksonUtil`
  - 分布式ID生成器功能(仅为抛砖引玉之作,需根据实际需求修改)
    - `SeqGenService` and `SeqGenUtil`
  - 基于新日期LocalDataTime&DateTimeFormatter封装的日期类
  - 功能entity封装(主要还是围绕自动主键生成而开发的)
  - 基础相应类封装(统一响应格式并开放自定义message)


