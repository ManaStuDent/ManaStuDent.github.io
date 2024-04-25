---
title: Javaagent 的简单使用
icon: circle-info
cover:
star: true
tag:
    - java
date: 2024-04-24
---

最近在使用 LiteFlow 的时候有发现控制台会打印这个 logo，就是类似于 SpringBoot 启动的那个 ASIIC 画的一个图标。于是乎我就找文档看看是否能够将这个 logo 给去掉。
但是好像没有发现有哪里配置能够将这个 logo 给去除掉。当然这个 logo 是一个 info 级别的 log 打印，正式使用的时候不会打印出来。但就是想给它去掉，咋弄呢？这个时候想了 Javaagent 这个技术。

<!-- more -->

## 介绍

LiteFlow 打印 logo 的方法在 `com.yomahub.liteflow.util.LOGOPrinter#print`，这里没有什么配置可以进行控制，只要用这个框架就会打印 info 级别的日志，并将 logo 输出到控制台或者文件中。
我想利用 javaagent 重写 print 方法，直接将其配置成空方法或者输出点其他什么的就好。

话不多说，先上代码。

```
package com.manastudent.agent;

import net.bytebuddy.agent.builder.AgentBuilder;
import net.bytebuddy.implementation.MethodDelegation;
import net.bytebuddy.matcher.ElementMatchers;

import java.lang.instrument.Instrumentation;

public class MyAgent {
    public static void premain(String agentOps, Instrumentation inst) {
        new AgentBuilder.Default()
                .type(ElementMatchers.named("com.yomahub.liteflow.util.LOGOPrinter"))
                .transform((builder, typeDescription, classLoader, module)
                        -> builder.method(ElementMatchers.named("print"))
                        .intercept(
                                MethodDelegation.to(MyAgent.class)
                        )).installOn(inst);
    }

    public static void print() {
        System.out.println("hello this is agent");
    }
}
```

这里为了简单还有代码的可阅读性使用了 `bytebuddy` 这个框架，这个框架我也不是很熟悉，第一次使用，后面有机会好好学习一下。

代码很简单，就是找到打印日志的类，然后替换掉它的 print 方法。

## 原理
Java 代理 (agent) 是在你的main方法前的一个拦截器 (interceptor)，也就是在 mai n方法执行之前，执行 agent 的代码。现在 IntelliJ IDEA 的破解（2022.1）还有以前的一些版本基本上都是利用这个技术进行破解的。还有阿里巴巴的 Arthas 应该也是。

## 编码

写一个 Javaagent 需要注意一些事项，下面都是基于 IDEA。

因为 Javaagent 的特殊性，需要一些特殊的配置，在 META-INF 目录下创建 MANIFEST.MF 文件

```
Manifest-Version: 1.0
Premain-Class: com.manastudent.agent.MyAgent

```

最小配置就上面 **三行** 代码，注意最后一行有一个空白行，为什么要，我暂时不清楚。

IDEA 打包的时候要注意，需要在 `Project Struct` 然后在 `Artifacts` 中新增一个 `JAR -> Empty` 即可，最后使用菜单栏上面的 `Build -> Build Artifacts` 即可在项目根目录下生成一个 `out` 文件夹。

![](https://img2022.cnblogs.com/blog/622937/202205/622937-20220506160010297-76490622.png)

![](https://img2022.cnblogs.com/blog/622937/202205/622937-20220506160022419-535108494.png)


