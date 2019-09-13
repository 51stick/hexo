---
title: drools drl 规则文件属性介绍
date: 2017-03-06 22:29:44
categories: drools
tags: drools
---

本文主要介绍drools规则文件中的相关属性，只有在充分了解规则文件属性含义，使用drools开发时才能得心应手，用得自信，用得放心，开发省心。
<!-- more -->

### 属性概要介绍

`规则属性`是用来控制规则执行的重要工具，在drools中目前有如下规则属性，分别是：activation-group、agenda-group、
auto-focus、date-effective、date-expires、dialect、duration、enabled、lock-on-active、no-loop、ruleflow-group、salience、when，这些属性分别适用于不同的场景，下面我们就来分别介绍这些属性的含义及用法。

### 属性详细介绍
#### salience
用来设置规则执行的优先级，salience 属性的值是一个整形数字，数字越大表示规则执行的优先级越高。同时它的值可以是一个负数。ssalience 的缺省值为0，如果未设置该属性值，规则的执行顺序是随机的。
```
rule "rule-01"
    salience 1
    when
        $messageFact : Message($status : status == Message.HELLO, $newMessage : message);
    then
        System.out.println("rule-01. Begin to chagne message of fact");
end
```

#### no-loop
该属性为布尔类型，可选值为true|false，缺省值是 false。其作用是控制已经执行过的规则当条件再次满足时是否再次执行。因为当规则引擎内部的`Working Memory`的Fact 更新时会引起引擎再次启动规则检查，并处罚规则的执行。如果no-loop设置为true，那么当规则再次执行时，引擎会忽略掉所有no-loop为true的规则，也就是说，执行过的规则不会因为Fact的更新再次执行。
```
rule "rule-01"
    salience 1
    no-loop  true
    when
        $messageFact : Message($status : status == Message.HELLO, $newMessage : message);
    then
        System.out.println("rule-01. Begin to chagne message of fact");
        $messageFact.setMessage("New message");
        update($messageFact)
end
```
上述示例，只会执行一次，如果no-loop设置为false，或则不设置该属性，那么该规则会陷入循环执行中，造成规则死循环。

#### date-effective
规则执行的有效时间，缺省状态下规则随时都可以执行，如果该属性设置了时间，那么只有当`系统时间` >= date-effective设置的时间时才会执行。默认情况下date-effective的格式为`dd-MMM-yyyy`，即英文操作系统中为`25-Sep-2016`,中文操作系统中为`25-九月-2019`
```
rule "rule-02"
    salience 2
    date-effective "25-十二月-2016"  // dd-MMM-yyyy
//    date-effective "25-Sep-2016"  // dd-MMM-yyyy
    when
        $messageFact : Message($status : status == Message.HELLO, $newMessage : message);
    then
        System.out.println("rule-02, Begin to chagne message of fact");
        $messageFact.setMessage("New message");
end
```

#### date-expires
规则的实效时间，和`date-effective`属性的意思相反。如果`系统时间` >= date-expires设置的时间，那么规则无效，不被执行。

#### enabled
设置该规则是否启用，布尔类型，可选值为false|true。

#### dialect
定义规则文件中使用的语言，可选值有`java`和`mvel`,缺省值是`java`。

#### duration
该属性值为长整型，单位为毫秒。如果设置了该属性，那么该规则在实行的时候，会在`duration`设置的毫秒数后，在另外一个线程中执行该规则。

#### lock-on-active
`no-loop` 的加强版。

#### activation-group、
规则组属性，如果多个规则具有相同`activation-group`属性值，那么表示这些规则属于同一个规则组中，同一规则组中的规则，当以有一个规则执行了，其余规则就不在执行了。规则组中的规则优先级还是通过`salience`来控制。
```
rule "rule-01"
    salience 1
    activation-group "group-01"
    when
        $messageFact : Message($status : status == Message.HELLO, $newMessage : message);
    then
        System.out.println("rule-01. Begin to chagne message of fact");
        $messageFact.setMessage("New message");
end

rule "rule-02"
    salience 2
    activation-group "group-01"
    when
        $messageFact : Message($status : status == Message.HELLO, $newMessage : message);
    then
        System.out.println("rule-02, Begin to chagne message of fact");
        $messageFact.setMessage("New message");
end
```
如上述的两个规则，会优先执行规则组`group-01`中的`rule-02`规则，而`rule-01`规则不会在执行了。

#### agenda-group
议程组属性，如果规则中设置了该属性，要使得议程组中的规则生效并正常执行，那么必须要给该规则再设置`auto-focus`属性。如果多个规则设置了该属性，组中的执行顺序由`salience`属性决定。如果有多个议程组规则，议程组的执行顺序由组中规则优先级最高的规则决定个，并且当一个议程组中的规则全部执行完成后(即使后面议程组的中的某些规则优先级比当前正在执行的议程组中的规则的优先级高，那么也得等当前议程组中的规则完了，才会执行下一个议程组中的规则)，才会执行下一个议程组中的规则，议程组的优先级，比普通规则优先级高，即使普通规则的`salience`最大，也还是会先执行议程组中的规则，而普通规则和`activation-group`的执行顺序则是由`salience`决定。

#### auto-focus
该属性的作用是用来在已设置了`agenda-group`的规则上设置该规则是否可以自动独取Focus，如果该属性设置为true，那么在引擎执行时就会执行该 议程组，否则不会。

#### ruleflow-group
在使用规则流的时候要用到ruleflow-group 属性，该属性的值为一个字符串，作用是用来将规则划分为一个个的组，然后在规则流当中通过使用ruleflow-group 属性的值，从而使用对应的规则。后面在讨论规则流的时候还要对该属性进行详细介绍。

到此，`drl`规则文件在中的属性已介绍完了，如果有写得不对的地方，请指正，谢谢。同时，通过自己写作记录，提升了自己对drools的先关认知，也希望帮助有所需要的朋友。