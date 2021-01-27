# <span style="color:#27929f">基本信息</span>

- 杨云森 | 177-2502-1454 | www.402645063@gmail.com | 高级Java开发
- <span style="color:#27929f">个人博客:</span> https://www.yangyunsen.com
- <span style="color:#27929f">GitHub:</span> https://github.com/cloudsen
- <span style="color:#27929f">期望薪资：</span>13K+
- <span style="color:#27929f">期望工作地：</span>重庆主城

```
WORK IT HARDER, MAKE IT BETTER, DO IT FASTER, MAKE US STRONGER.
```

---


# <span style="color:#27929f">教育经历</span>

***<span style="color:#329ba9">重庆交通大学（2013.9 ~ 2017.7）</span>***

- <span style="color:#329ba9">专业:</span> 电子信息工程
- <span style="color:#329ba9">学位:</span> 工科学士

---

# <span style="color:#27929f">工作经历</span>

## <span style="color:#329ba9">电信科学技术第五研究所（2019年10月 ~ 至今）</span>

***第六研究室***

- 参与“十三五”国家型号项目设计与开发、5000A文档编写、框架搭建、自动化、脚本运维等
- 多源数据采集离线分析系统 [贡献100%]
  - 职责：项目负责人、后端开发
  - 大致设计：用户可在前端进行，动态增删各种类型的输入输出数据源、创建采集规则、创建采集任务、采集速率监控、数据检索查询、数据离线分析等操作。后端采用Confluent流处理平台，首先根据输出标准，创建并注册Schema，然后根据规则将采集到的数据做处理，处理完毕后通过AVRO序列化缓存到KAFKA中，接着由Kafa-connector-es-sink建立Kafka到ES的通道，将数据录入ES供后续聚合分析使用。
  - 独自设计实现了灵活的 Kafka-connector-es-sink
  - 协调两名前端开发，用一个月时间做完功能完备的第一版系统
- 某信号自识别系统 [贡献85%]
  - 职责：项目负责人、后端开发
  - 大致设计：硬件设备由不同类型的板卡和槽位组成，各个设备之间通过光纤连接，用户在前端注册线路和硬件设备信息，并创建针对某条线路的分析任务，待硬件分析完毕后可查看该条线路的各种信息。后端并发处理分析任务，通过Netty与设备进行socket通信，然后根据调度策略和硬件资源情况将处理流程逐级发送给不同的板卡处理。
  - 独自设计整体架构、通信协议、寻径策略、任务策略、设计线程安全类、设计响应处理器链
  - 协条两名后端开发，一名前端开发，用4个月时间完成整个系统的开发和联调工作成功上线
- 某设备集群网管系统 [贡献60%]
  - 职责：项目负责人、后端开发
- 某线路筛查系统重构 [贡献100%]
  - 职责：后端开发

## <span style="color:#329ba9">重庆超鲸科技有限公司（2018年11月 ~ 2019年4月）</span>

***区块链部门***

- 参与Sharder区块链的开发：https://github.com/Sharders/sharder-chain/graphs/contributors
  - 独自承担从需求->原型->前后端开发->测试->部署的整个流程；
  - 编写Linux shell读取设备信息，并且与JAVA后端进行交互，使用VUE全家桶开发前端；

## <span style="color:#329ba9">重庆春之翼信息科技有限公司（2017年4月 ~ 2018年8月）</span>

***重庆研发部***

- 维护MIS机务维修系统、BPS流程管理系统、通用航空维修系统, 并开发新需求。

----



# <span style="color:#329ba9">个人项目</span>

### 自动化脚本

- ArchLinux自动安装脚本：https://github.com/CloudSen/archlinux_installer_vm
- 项目部署/环境安装脚本：https://github.com/CloudSen/clouds3n-shell

### 自定义Starter

为项目定制spring-boot-starter，新建项目后即可直接编写业务代码无需关注配置和依赖，为业务开发和调试争取更多的时间：

https://github.com/CloudSen/tron

### 项目生成器

与自定义starter相辅相成，实现在web页面配置并一键生成可用项目[开发中]：

https://github.com/CloudSen/web-project-generator

### 个人博客

之前自己的笔记都是存放GitHub中的，不太方便查看和整理，于是抽时间搭建了一个自己的技术博客，主要记录和整理自己所学的一些东西，以及翻译一些英文文档或英文博客，方便回顾。

- <span style="color:#329ba9">技术栈：</span>Vue-CLI3 + Vue Router + Vuex + Vuetify + Spring Boot + MyBatis Plus + Nginx

- <span style="color:#329ba9">代码地址：</span>
  - 博客前端：https://github.com/CloudSen/cloudsen-blog-vue-front
  - 博客看板前端：https://github.com/CloudSen/clouds3n-blog-dashboard-vue
  - 后端：https://github.com/CloudSen/clouds3n-blog-java

### Python版个人博客

- <span style="color:#329ba9">技术栈：</span>Python3 + Django2.1 + Nginx + Gunicorn + Bootstrap 4 Material Design + jQuery 
- <span style="color:#329ba9">代码地址：</span>https://github.com/CloudSen/RedQueen

---


# <span style="color:#27929f">技能清单</span>

我在学习生涯与工作中逐渐掌握与学习了以下技术：

- <span style="color:#27929f">语言：</span>Java 8+ / Shell  / HTML5 / CSS3 / ES6
- <span style="color:#27929f">后端框架：</span>Spring / Spring Boot / Spring Cloud(Netflix) / Spring Data JPA / MyBatis / MyBatis-Plus
- <span style="color:#27929f">前端框架：</span>Bootstrap4 / jQuery / VUE-CLI3 / VUEX / VUE Router / Element-UI / Vuetify
- <span style="color:#27929f">中间件：</span> Redis / Kafka / Confluent
- <span style="color:#27929f">通信组件：</span> Netty
- <span style="color:#27929f">搜索引擎：</span>ElasticSearch 7
- <span style="color:#27929f">系统开发环境：</span>Linux(Arch | CentOS | 中标麒麟) / Windows
- <span style="color:#27929f">数据库相关：</span>MySQL / Oracle
- <span style="color:#27929f">版本管理：</span>Git / SVN
- <span style="color:#27929f">部署：</span>Docker / 自动化脚本 / Nginx
- <span style="color:#27929f">IDE/Editor：</span> IDEA / VS Code / VIM
- <span style="color:#27929f">代码规范：</span> 阿里巴巴JAVA规范 / ESLint
- <span style="color:#27929f">并发编程：</span> 熟悉并发编程，能设计实现线程安全类，规避常见并发陷阱
- <span style="color:#27929f">设计模式：</span> 熟悉常用的设计模式，能根据需求合理运用

能独立完成中小型项目并上线，其他的技术我会利用空余时间，查看官方文档，学习并应用。      

