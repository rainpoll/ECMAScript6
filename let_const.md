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

	var tmp=123;

	if(true) {
		tmp='abc';//ReferenceError
		let tmp;
	}
>ES6明确规定，如果区块中存在`let`和`const`命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域，凡是在声明之前就使用这些命令，就会报错。
>在代码块内，使用`let`命令声明变量之前，该变量都是不可用的。这在语法上称为"暂时性死区"(temporal dead zone).

	if(true) {
		//TDZ start
		tmp='abc';//ReferenceError
		console.log(tmp);//ReferenceError
	
		let tmp;//TDZ end
		console.log(tmp);//undefined
	
		tmp=123;
		console.log(tmp);//123
	}

###不允许重复声明
>`let`不允许在相同作用域内，重复声明同一个变量。

	// Uncaught SyntaxError
	function () {
	  let a = 10;
	  var a = 1;
	}
	
	// Uncaught SyntaxError
	function () {
	  let a = 10;
	  let a = 1;
	}

>不允许在函数内部重新声明参数:

	//Uncaught TypeError	
	function func(arg) {
		let arg;
	}

	//No Error
	function func(arg) {
		{
			let arg;
		}
	}
	

##2、块级作用域

###ES6的块级作用域
>`let`为JavaScript新增加了块级作用域：
	
	function f1() {
		let n=5;
		if(true) {
			let n=10;
		}
		console.log(n);//5	
	}
	f1();

>ES6允许块级作用域任意嵌套.

	{
		{
			{
				let vname='Hello World';
			}
			console.log(vname);//ReferenceError
		}
	}
>块级作用域的出现，使得获得广泛应用的立即执行匿名函数(IIFE)不再必要。

	//IIFE
	(function() {
		var tmp=...;
		// code here...
	}());
	
	//块级作用域写法
	{
		let tmp=...;
		// code here...
	}

>ES6也规定，函数本身的作用域，在其所在的块级作用域之内。

	function f() { console.log('I am outside!'); }
	(function () {
	  if(false) {
	    // 重复声明一次函数f
	    function f() { console.log('I am inside!'); }
	  }
	
	  f();
	}());
	
	output:"I am outside!"

>块级作用域外部，无法调用块级作用域内部定义的函数:

	{
		let a='hello';
		function f() {
			console.log(a);
		}
	}
	
	f();//ReferenceError

	//modyfied here
	
	let f;
	{
	  let a = 'hello';
	  f = function () {
	    console.log(a);
	  }
	}
	f() // "hello"

##3、const命令
>const也用来声明变量，但是声明的是常量。一旦声明，常量的值就不能改变。

	const PI=3.1415;
	console.log(PI);

	PI=3;//"PI" is read-only