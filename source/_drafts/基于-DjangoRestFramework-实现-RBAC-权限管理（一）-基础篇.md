---
title: 基于 DjangoRestFramework 实现 RBAC 权限管理（一） -- 基础篇
date: 2020-03-03 21:55:29
tags:

- Python
- Django

---

## 前言

打算写几篇博客来总结这段时间使用 Python DjangoRestFramework 构建权限管理系统的过程。

在这项目实现的过程中确实遇到了一些问题，比如权限管理中如何实现组织权限、数据权限，如何实现颗粒度到字段级别的权限等等。这些问题都将在之后的博客中给到详细解答。

本文适合刚入门 Django 小白，因为我也就这水平。如果你是带佬，文章中有哪些思路不正确或者逻辑错误的地方，欢迎指出，蟹蟹。

## 准备

我会准备一个后端的 demo 来演示整个权限管理系统是如何工作的。后端就用 DRF，后台管理界面先用 DRF default 的界面凑合着用吧。空了用 Vue 或者 React 实现一个管理后台。以下是具体技术栈：

- Python 3.7

- Django

- DjangoRestFramework

项目

