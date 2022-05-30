---
title: Spring 7种事务传播 
date: 2016-01-07 11:42:00 +0800
categories: [Java, Spring]
tags: [develop]
---

1. PROPAGATION_REQUIRED 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。
2. PROPAGATION_SUPPORTS 支持当前事务，如果当前没有事务，就以非事务方式执行。
3. PROPAGATION_MANDATORY 使用当前的事务，如果当前没有事务，就抛出异常。
4. PROPAGATION_REQUIRES_NEW 新建事务，如果当前存在事务，把当前事务挂起。
5. PROPAGATION_NOT_SUPPORTED 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
6. PROPAGATION_NEVER 以非事务方式执行，如果当前存在事务，则抛出异常。
7. PROPAGATION_NESTED 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与 PROPAGATION_REQUIRED 类似的操作。