#let和const命令
##1、let命令
###基本用法
>ES6中新增加了`let`命令，用来声明变量。用法类似于`var`，但它声明的变量，只在`let`命令所在的代码块内有效。

	{
		let a=10;
		var b=1;
	}
	
	console.log(a);// a is not defined.
	console.log(b);//1
>`for`循环的计数器：

	var a = [];
	for (var i = 0; i < 10; i++) {
	  a[i] = function () {
	    console.log(i);
	  };
	}
	a[6](); // 10
	a[5]();	//10
	
	var a = [];
	for (let i = 0; i < 10; i++) {
	  a[i] = function () {
	    console.log(i);
	  };
	}
	a[6](); // 6
	a[5]();	//5
>当变量i是var声明的，在全局范围内都有效。所以每一次循环，新的i值都会覆盖旧值，导致最后输出的是最后一轮的i的值10,10。

>如果使用let，声明的变量仅在块级作用域内有效，最后输出的是6,5。

###不存在变量提升
>`let`不像`var`那样，会发生"变量提升"。所以变量一定要在声明后使用。

	console.log(foo);//Reference Error
	let foo=2;
	
	typeof x;//Reference Error
	let x;

###暂时性死区
>只要块级作用域内存在`let`命令，它所声明的变量就"绑定"这个区域，不再受外部的影响。

