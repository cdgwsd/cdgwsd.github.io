---
title: ComponentScan
tags:
  - Spring
  - 注解
  - ComponentScan
categories:
  - Spring
  - 注解
date: 2024-03-27 15:40:00
---

# @ComponentScan

`@ComponentScan` 注解的作用就是扫描指定路径，把路径中符合规则的类装配到 Spring 容器中。通常与 `@Configuration` 注解一起使用

`@ComponnetScan` 注解通过指定 `basePackageClasses` 或 `basePackages`（或其别名值）来定义要扫描的特定包。如果没有提供特定的扫描路径，默认从声明该注解的类所在的包开始扫描

`@ComponentScan` 注解提供与 Spring XML 配置中 [<context:component-scan>](../../Spring%20MVC/component-scan%20标签.md) 元素相同的功能


