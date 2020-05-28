---
layout: post
title:  "阿里巴巴开发手册(一)"
categories: Java规范
tags:  Java 码出高效
author: ZyWang
excerpt: 阿里巴巴开发手册
---

****
# 前言 #
最近在读阿里巴巴开发手册，查漏补缺
# 编程规约 #
1. 代码中命名不能以下划线和美元符号开始或结束。**反例**:_name/$name/name_/name$
   
   	个人认为可能是有些语言底层使用到这些标识符，例如Python中下划线就有特殊含义
2. 类名大驼峰，方法名、参数名、成员变量、局部变量小驼峰，固定用法，但以下类除外:DO/BO/DTP/VO/AO/PO/PUID等