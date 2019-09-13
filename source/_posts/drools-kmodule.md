---
title: drools中kmodule.xml文件介绍
date: 2017-03-05 22:21:13
categories: drools
tags: drools
---

本文主要介绍drools中的kmodule.xml文件的相关配置的含义。
<!-- more -->
### kmodule.xml文件
要想使用drools规则引擎，首先要在项目的`resources/META-INF`路径下添加`kmodule.xml`文件。如：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<kmodule xmlns="http://jboss.org/kie/6.0.0/kmodule">

    <!-- kbase是一个应用程序知识的定义的仓库，里面包含要使用的规则 -->
    <kbase name="rules" packages="com.drools.examples.mine">
        <ksession name="ksession-rules"/>
    </kbase>

</kmodule>
```
后面我们将介绍一下`kmodule.xml`文件中的相关配置的含义。

### kbase 属性介绍
| 属性名 | 缺省值 | 允许值 | 含义解释 |
| :--: | :--: | :--: | :---- |
| name | 无 | 任何 | 用于从KieContainer中检索KieBase，必须的属性。|
| includes | 无 | 以逗号分割的KieBase列表 | 用于将当前kmodule中的其他KieBases包含到当前的KieBase中。|
| packages | 所有 | 以逗号分割的KieBase列表 | 默认将resources下的所有文件夹下得drl包含到当前KieBase，用来限制编译那些drl属于当前KieBase。 |
| default | false | true，false | 定义是否为kmodule默认的KieBase，如果是，那么当从KieContainer中创建KieBase时，可以不用指定KieBase的name值。一个kmodule中最多只能有一个默认的KieBase。|
| equalsBehavior | identity | identity，equality | 定义Drools事实(fact)查询到工作内存(Working Memory)的行为，identity:不管当前工作内存中是否存在该事实，都创建一个FactHandle事实；equality：当工作内存中不存在当前事实时才创建。|
| eventProcessingMode | cloud | cloud, stream | 当以cloud模式编译时，KieBase被当成是一个普通的事实对待；而当以stream模式编译时，KieBase允许对它进行时间推理。|
| declarativeAgenda | disabled | disabled, enabled | 定义是否启用声明式议程 |

### ksession 属性介绍
| 属性名 | 缺省值 | 允许值 | 含义解释 |
| :--: | :--: | :--: | :---- |
| name | 无 | 任何 | 名字具有唯一性，用于从KieContainer中获取KieSession，必须的属性。|
| type | stateful | stateful, stateless | stateful:允许在工作内存中循环执行； stateless：对于工作内存中提供的数据集，只能执行一次。|
| default | false | true, false | 定义是否为kmodule默认的kieSession，如果是，那么从KieContainer中创建KieSession时，可以不用指定KieSession的name值，一个kmodule中，对于每一种type，最多只能有一个默认的KieBase。|
| clockType | realtime | realtime, pseudo | 定义事件的时间戳是由系统时钟还是应用程序的伪时钟决定。该lock在对时间规则进行单元测时非常有用。|
| beliefSystem | simple | simple, jtms, defeasible | 通过当前KieSession的此属性值来定义belief system的类型。|

`注意`：上述是关于kbase和ksession的属性的基本介绍，如有描述不当的地方，请指正。