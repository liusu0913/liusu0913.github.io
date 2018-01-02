---
layout: blog
vue: true
background-image: http://ot1cc1u9t.bkt.clouddn.com/17-7-16/91630214.jpg
category: vue
title: Vue的组件之间的通信
tags:
- vue
- vue的父子组件
- 组件通信
---
## Vue父组件传递给子组件数据
> 双向数据流动：父组件数据改变影响子组件数据；子组件数据变化不影响父组件；

> 单项数据流动：只有妇组件影响子组件，子组件数据变化不影响父组件；（vue提倡这种）

- [x] 父组件传递给子组件的数据利用prop组件传递；

### 传递的数据类型问题
> 传递的是值类型肯定没问题，假如是对象或者数组这样的引用类型，变量共用同一内存，所以父子会出现双向影响，这个问题一定要注意

> 当子组件直接改变父组件传递的数据时，会爆出一个错误，解决办法是将父组件传递所来的数据利用自身的data来存储在进行使用即可；

### 父组件传递异步的数据给子组件时：

> 假设父组件的数据是异步ajax获取的，然后渲染页面。那么此时，这就得有个监听机制。当子组件监听到父组件的数据到位后，将本地数据更新即可。vue提供的watch可以达到此目的；

> watch设置在子组件上，监听的是传递过来的数据；

## 子组件数据改变影响父组件；
> $on就是监听事件，大家使用过jquery的话应该都不陌生；而这个$emit是发射、触发的意思。一个监听，一个触发，这是个典型的观察者模式。

> 我们可以在父组件中注册一个监听事件，专门用作监听子组件的变化；而当子组件发生变化时，就会触发监听事件，从而让父组件得到子组件的变化。就是父组件利用on来检测，子组件利用emit来触发发送的数据；

## 兄弟组件之间的数据的传递
> 我们可以创建一个公共对象（官方文档也称之为event bus），专门用来注册监听事件。两个兄弟组件需要通信，只需共同引入这个公共对象即可

> 方法就是引入共同的bus.js在busjs上设置事件的触发和监听；

##  例子
在github上的vue-dome库
