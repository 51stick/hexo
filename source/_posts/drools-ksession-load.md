---
title: Drools加载StatefulKnowledgeSession的三种方法
date: 2017-07-07 19:20:14
categories: drools
tags: drools
---

本文主要介绍drools中加载StatefulKnowledgeSessionde的方式
<!-- more -->

注意：本文主要摘抄与官方文档翻译。

### 概要介绍
本文主要介绍drools中三种加载StatefulKnowledgeSession的方法，分别如下：

- 从本地加载StatefulKnowledgeSession

- 从KnowledgeAgent加载StatefulKnowledgeSession

- 从远程URL加载StatefulKnowledgeSession


>Note: The source code path: ./drools/ksession, KnowledgeAgentUsage.drl should synchronised to Guvnor and org.drools.examples build successful in Guvnor

###  从本地加载 ksession
代码如下：
```java
private static StatefulKnowledgeSession readKnowledgeSession(String name) throws Exception {
    kbuilder.add(ResourceFactory.newClassPathResource(name),ResourceType.DRL);
    KnowledgeBuilderErrors errors = kbuilder.getErrors();
    KnowledgeBase kbase = KnowledgeBaseFactory.newKnowledgeBase();
    kbase.addKnowledgePackages(kbuilder.getKnowledgePackages());
    return kbase.newStatefulKnowledgeSession();
}
```

### 从KnowledgeAgent加载StatefulKnowledgeSession
代码如下：
```java
private static StatefulKnowledgeSession readKnowledgeSession(String changeset) throws Exception {
    kagent = KnowledgeAgentFactory.newKnowledgeAgent("kagent");
    ResourceFactory.getResourceChangeNotifierService().start();
    ResourceFactory.getResourceChangeScannerService().start();
    kagent.setSystemEventListener(new PrintStreamSystemEventListener());
    kagent.applyChangeSet(ResourceFactory.newClassPathResource(changeset));
    KnowledgeBase kbase = kagent.getKnowledgeBase();
    return kbase.newStatefulKnowledgeSession();
}
```

### 从远程URL加载StatefulKnowledgeSession
代码如下：
```java
private static StatefulKnowledgeSession newKnowledgeSession(String url) throws Exception {
    UrlResource resource = (UrlResource) ResourceFactory.newUrlResource(url);
    resource.setBasicAuthentication("enabled");
    resource.setUsername("admin");
    resource.setPassword("admin");
    KnowledgeBuilder kbuilder = KnowledgeBuilderFactory.newKnowledgeBuilder();
    kbuilder.add(resource, ResourceType.PKG);
    KnowledgeBase kbase = kbuilder.newKnowledgeBase();
    return kbase.newStatefulKnowledgeSession();
}
```