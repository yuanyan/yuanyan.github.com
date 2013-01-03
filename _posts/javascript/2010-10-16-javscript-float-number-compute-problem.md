---
layout: post
category : javascript
title:  Javascript 浮点运算问题分析与解决
header: Javascript 浮点运算问题分析与解决
tagline:
tags : [javascript, number, IEEE-754]
---
{% include JB/setup %}

## 分析

JavaScript 只有一种数字类型 `Number` ，而且在Javascript中所有的数字都是以[IEEE-754](http://zh.wikipedia.org/zh-cn/IEEE_754)标准格式表示的。
浮点数的精度问题不是JavaScript特有的，因为有些小数以二进制表示位数是无穷的：

    十进制           二进制
    0.1              0.0001 1001 1001 1001 ...
    0.2              0.0011 0011 0011 0011 ...
    0.3              0.0100 1100 1100 1100 ...
    0.4              0.0110 0110 0110 0110 ...
    0.5              0.1
    0.6              0.1001 1001 1001 1001 ...

所以比如 `1.1` ，其程序实际上无法真正的表示 '1.1'，而只能做到一定程度上的准确,这是无法避免的精度丢失：

    1.09999999999999999

在JavaScript中问题还要复杂些，这里只给一些在Chrome中测试数据：

     输入               输出
    1.0-0.9 == 0.1     False
    1.0-0.8 == 0.2     False
    1.0-0.7 == 0.3     False
    1.0-0.6 == 0.4     True
    1.0-0.5 == 0.5     True
    1.0-0.4 == 0.6     True
    1.0-0.3 == 0.7     True
    1.0-0.2 == 0.8     True
    1.0-0.1 == 0.9     True

## 解决

那如何来避免这类 ` 1.0-0.9 != 0.1 ` 的非bug型问题发生呢？下面给出一种目前用的比较多的解决方案,
在判断浮点运算结果前对计算结果进行精度缩小，因为在精度缩小的过程总会自动四舍五入:

    (1.0-0.9).toFixed(digits)                   // toFixed() 精度参数须在 0 与20 之间
    parseFloat((1.0-0.9).toFixed(10)) === 0.1   // 结果为True
    parseFloat((1.0-0.8).toFixed(10)) === 0.2   // 结果为True
    parseFloat((1.0-0.7).toFixed(10)) === 0.3   // 结果为True
    parseFloat((11.0-11.8).toFixed(10)) === -0.8   // 结果为True
	
## 方法提炼	

	// 通过isEqual工具方法判断数值是否相等
	function isEqual(number1, number2, digits){
		digits = digits == undefined? 10: digits; // 默认精度为10
		return number1.toFixed(digits) === number2.toFixed(digits);
	}
	
	isEqual(1.0-0.7, 0.3);  // return true
	
	// 原生扩展方式，更喜欢面向对象的风格
	Number.prototype.isEqual = function(number, digits){
		digits = digits == undefined? 10: digits; // 默认精度为10
		return this.toFixed(digits) === number.toFixed(digits);
	}
	
	(1.0-0.7).isEqual(0.3); // return true