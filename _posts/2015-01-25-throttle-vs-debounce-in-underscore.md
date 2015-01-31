---
layout: post
title: "throttle vs. debounce in underscore"
description: ""
category: 
tags: []
---

###1. 简介
  在浏览器中，有不少事件或者函数调用频率会非常高，如mousemove，resize、scroll等，针对这种事件绑定函数时，函数可能会被频繁调用。如果绑定的函数比较复杂，性能会变得很差。
throttle和debounce都是作为节流函数来解决这类问题的。    
  但是这两者间各自的作用是什么，它们又有什么区别呢？

###2. 对比
  先看一个例子，感受一下两者的不同：[http://jsfiddle.net/30jrhnra/1/](http://jsfiddle.net/30jrhnra/1/)

####2.1 区别
  1. 频繁点击throttle按钮时，调用会以稳定的频率触发，该频率就是我们设置的时间（函数的第二个参数，例子中是1s）
  2. 频繁点击debounce按钮时，调用间距如果在设定的频率范围内，该事件会被直接丢弃

####2.2 使用场景
  1. throttle适合处理高频度触发的事件，如mousemove、resize、scroll等，可以控制其调用频率，提高性能
  2. debounce适合处理可能误操作的事件，一个典型的例子是表单提交（可以延伸为任何可能与后台交互的用户事件），用户不小心做了一次双击，表单也应该只提交一次

####2.3 api

#####1. _.throttle(function, wait, [options]) 
  参数说明：    
  function  要做节流的函数    
  wait      节流函数的触发频率    
  options   {leading: true/false, trailing: true/false}控制函数执行的前后摇

#####2. _.debounce(function, wait, [immediate])
  参数说明：    
  function  要做节流的函数    
  wait      节流函数的触发频率    
  immediate true/false 是否立即执行第一次    
  注意： immediate为false时，第一次执行也会延迟指定的时间才执行，所以对延迟较大或者响应实时要求比较高的该参数都要指定为true