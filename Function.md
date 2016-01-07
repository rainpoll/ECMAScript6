#函数的扩展
#
##1、函数参数的默认值
###基本用法 
>在ES6之前，不能直接为函数的参数指定默认值，只能采用变通的方法。

	function log(x,y) {
		y=y||'World';
		console.log(x,y);
	}

	log('Hello') // Hello World
	log('Hello', 'China') // Hello China
	log('Hello', '') // Hello World
>

	function Point(x = 0, y = 0) {
	  this.x = x;
	  this.y = y;
	}
	
	var p = new Point();
	console.log(p); // { x: 0, y: 0 }

>参数变量是默认声明的，所以不能用let或const再次声明。

	function foo(x=5) {
		let x=1;
		const x=2;
	}//error
	//参数变量x是默认声明的，在函数体中，不能用let或const再次声明，否则会报错。
###与解构赋值默认值结合使用
>参数默认值可以与解构赋值的默认值，结合起来使用。
	
	function foo({x, y = 5}) {
	  console.log(x, y);
	}
	
	console.log(foo({})); // undefined, 5
	console.log(foo({x: 1})); // 1, 5
	console.log(foo({x: 1, y: 2})); // 1, 2
	console.log(foo()); // TypeError

>上面的代码使用了对象的解构赋值默认值，而没有使用函数参数的默认值。只有当函数`foo`的参数是一个对象时，变量`x`和`y`才会通过解构赋值而生成。如果函数`foo`调用时参数不是对象，变量`x`和`y`就不会生成，从而报错。如果参数对象没有`y`属性，`y`的默认值才会生效。 

	//写法一
	function m1({x=0,y=0}={}) {
		console.log([x,y]);
	}

	//写法二
	function m2({x,y}={x:0,y:0}) {
		console.log([x,y]);
	}

	m1();//[0,0]
	m2();//[0,0]

###参数默认值的位置
>通常情况下，定义了默认值的参数，应该是函数的尾参数。因为这样比较容易看出来，到底省略了哪些参数。如果非尾部的参数设置默认值，实际上这个参数是没法省略的。

	function f1(x=1,y) {
		console.log([x,y]);
	}

	f1();//[1,undefined]
	f1(2);//[2,undefined]
	f1(,1);//Error
	f1(undefined,1);//[1,1]

	function f(x,y=5,z) {
		console.log([x,y,z]);
	}

	f();//[undefined,5,undefined]
	f(1);//[1,5,undefined]
	f(1,2);//Error
	f(1,undefined,2);//[1,5,2]
>有默认值的参数都不是尾参数。这时，无法只省略该参数，而不省略它后面的参数，除非显式输入`undefined`。

>如果传入`undefined`,将触发该参数等于默认值，`null`则没有这个效果。

	function foo(x=5,y=6) {
		console.log(x,y);
	}

	foo(undefined,null);//5 null

###函数的length属性
>指定了默认值以后，函数的`length`属性，将返回没有指定默认值的参数个数。也就是说，指定了默认值后，`length`属性将失真。

	var len=(function(a){}).length;
	console.log(len);//1

	console.log((function(a=5){}).length);//0
	console.log((function(a,b,c=5){}).length);//2
>`length`属性的含义是，该函数预期传入的参数个数。某个参数指定默认值以后，预期传入的参数个数就不包括这个参数了。同理，rest参数也不会计入`length`属性。

	console.log((function(...args){}).length);//0

###作用域
>如果参数默认值是一个变量，则该变量所处的作用域，与其他变量的作用域规则是一样的。即先是当前函数的作用域，然后才是全局作用域 。

	var x=1;
	function f(x,y=x) {
		console.log(y);
	}

	f(2);//2
>如果调用时，函数作用域内部的变量`x`没有生成，结果就不一样。

	let x=1;
	function f(y=x) {
		let x=2;
		console.log(y);
	}

	f();//1
>如果全局变量`x`不存在，就会报错。

	function f(y = x) {
		let x=2;
		console.log(y);
	}

	f();//ReferenceError:

###应用
>利用参数默认值，可以指定某一个参数不得省略，如果省略就抛出一个错误。

	function throwIfMissing() {
		throw new Error('Missing parameter');
	}

	function foo(mustBeProvided=throwIfMissing()) {
		console.log(mustBeProvided);
	} 

	foo();//Uncaught Error: Missing parameter
>参数`mustBeProvided`的默认值等于`throwIfMissing`函数的运行结果，这表明参数的默认值不是在定义时执行，而是在运行时执行(即如果参数已经赋值，默认值的函数就不会运行)。可以将参数默认值设为`undefined`，表明这个参数是可以省略的。

	function foo(optional=undefined) {...}

##2、rest参数
>ES6引入rest参数，用于获取函数的多余参数，这样就不需要使用arguments对象了。rest参数搭配的变量是一个数组，该变量将多余的参数放入数组中。

	function add(...values) {
		let sum=0;

		for(var val of values) {
			sum+=val;
		}

		return sum;
	}

	console.log(add(2,5,3));//10

>rest参数代替arguments变量的例子：

	//arguments变量的写法
	const sortNumbers=() =>
		Array.prototype.slice.call(arguments).sort();

	//rest参数的写法
	const sortNumbers=(...numbers) => numbers.sort();
	
>rest参数改写数组push方法

	function push(array,...items) {
		items.forEach(function(item) {
			array.push(item);
			console.log(item);
		})
	}

	var a=[];
	push(a,1,2,3);// 1 2 3
>rest参数之后不能再有其他参数(即只能是最后一个参数)。

	function f(a,...b,c) {
		//...
	}

>函数的length属性，不包括rest参数。

	console.log((function(a) {}).length);//1
	console.log((function(...a){}).length);//0
	console.log((function(a,...b) {}).length);//1	

##3、扩展运算符
###含义
>扩展运算符(spread)是三个点(`...`)。它好比rest参数的逆运算，将一个数组转为用逗号分隔的参数序列。

	console.log(...[1,2,3]);//1 2 3
	
	console.log(1,...[2,3,4],5);//1 2 3 4 5

	console.log([...document.querySelectorAll('div')]);//[<div>,<div>,<div>]

>该运算符主要用于函数调用。

	function push(array,...items) {
		array.push(...items);
	}

	function add(x,y) {
		console.log(x+y);
	}

	var numbers=[4,38];
	add(...numbers);//42

###替代数组的apply方法
>由于扩展运算符可以展开数组，所以不需需要`apply`方法，将数组转为函数的参数了。

	//ES5的写法
	function f(x,y,z) {
		//...
	}

	var args=[0,1,2];
	f.apply(null,args);

	//ES6的写法
	function f(x,y,z) {
		//...
	}
	
	var args=[0,1,2];
	f(...args);
>通过`push`函数，将一个数组添加到另一个数组的尾部。

	//ES5的写法
	var arr1=[0,1,2];
	var arr2=[3,4,5];
	Array.prototype.push.apply(arr1,arr2);
	console.log(arr1);//[0, 1, 2, 3, 4, 5]

	//ES6的写法
	var arr1=[0,1,2];
	var arr2=[3,4,5];
	arr1.push(...arr2);
	console.log(arr1);//[0, 1, 2, 3, 4, 5]

###扩展运算符的应用
__(1)合并数组__
>扩展运算符提供了数组合并的新写法

	// ES5
	[1, 2].concat(more)
	// ES6
	[1, 2, ...more]

	var arr1 = ['a', 'b'];
	var arr2 = ['c'];
	var arr3 = ['d', 'e'];

	// ES5的合并数组
	var arr=arr1.concat(arr2, arr3);
	console.log(arr);// [ 'a', 'b', 'c', 'd', 'e' ]

	// ES6的合并数组
	var arr=[...arr1, ...arr2, ...arr3]
	console.log(arr);// [ 'a', 'b', 'c', 'd', 'e' ]
__(2)与解构赋值结合__
>扩展运算符可以与解构赋值结合起来，用于生成数组。

	const [first, ...rest] = [1, 2, 3, 4, 5];
	console.log(first); // 1
	console.log(rest);  // [2, 3, 4, 5]
	
	const [first, ...rest] = [];
	console.log(first); // undefined
	console.log(rest);  // []:
	
	const [first, ...rest] = ["foo"];
	console.log(first);  // "foo"
	console.log(rest);   // []
__(3)函数的返回值__
>JavaScript的函数只能返回一个值，如果需要返回多个值，只能返回数组或对象。扩展运算符提供了解决这个问题的一种变通方法。

	//从数据库取出一行数据
	var dateFields=readDataFields(database);
	var d=new Date(...dateFields);
__(4)字符串__
>扩展运算符还可以将字符串转为真正的数组,能够正确识别32位的Unicode字符。

	console.log([...'hello']);//[ 'h', 'e', 'l', 'l', 'o' ]
	console.log('x\uD83D\uDE80y'.length);//4
	console.log([...'x\uD83D\uDE80y'].length);//3
>正解返回字符串长度的函数：

	function length(str) {
		console.log([...str].length);
	}

	length('x\uD83D\uDE80y');//3
>凡是涉及到操作32位Unicode字符的函数，都有这个问题。因此，最好都用扩展运算符改写。

	let str='x\uD83D\uDE80y';

	console.log(str.split('').reverse().join(''));// 'y\uDE80\uD83Dx'

	console.log([...str].reverse().join(''));//'y\uD83D\uDE80x'
__(5)实现了Iterator接口的对象__
>任何Iterator接口的对象，都可以用扩展运算符转为真正的数组。

	var nodeList=document.querySelectorAll('div');
	var array=[...nodeList];
>对于那些没有部署Iterator接口的类似数组的对象，扩展运算符就无法将其转为真正的数组。

	let arrayLike={
		'0':'a',
		'1':'b',
		'2':'c',
		length:3
	};

	let arr=[...arrayLike];//TypeError

__(6)Map和Set结构，Generator函数__
>扩展运算符内部调用的是数据结构的Iterator接口，因此只要具有Iterator接口的对象，都可以使用扩展运算符,比如Map结构。

	let map=new Map([
		[1,'one'],
		[2,'two'],
		[3,'three']
	]);

	let arr=[...map.keys()];//[1,2,3]
>Generator函数运行后，返回一个遍历器对象，因此也可以使用扩展运算符。

	var go=function *() {
		yield 1;
		yield 2;
		yield 3;
	};

	console.log([...go()]);//[1,2,3]
>如果对没有`iterator`接口的对象，使用扩展运算符，将会报错。

	var obj={a:1,b:2};
	let arr=[...obj];//TypeError

##4、name属性
>函数的`name`属性，返回该函数的函数名。

	function foo() {}
	console.log(foo.name);//"foo"

>ES6对这个属性的行为做出了一些修改。如果将一个匿名函数赋值给一个变量，ES5的`name`属性，会返回空字符串，而ES6的`name`属性会返回实际的函数名。

>如果将一个具名函数赋值给一个变量，则ES5和ES6的`name`属性都返回这个具名函数原本的名字。

	const bar=function baz(){};

	//ES5
	console.log(bar.name);//"baz"
	
	//ES6
	console.log(bar.name);//"baz"
>`Function`构造函数返回的函数实例，`name`属性的值为"anonymous"。

	console.log((new Function).name);//"anonymous"
>`bind`返回的函数，`name`属性值会加上"bound"前缀。

	function foo() {};
	var res=foo.bind({}).name;
	console.log(res);//"bound foo"

	var ret=(function(){}).bind({}).name;
	console.log(ret);//"bound"

##5、箭头函数 
###基本用法
>ES6允许使用"箭头"(=>)定义函数。

	var f=v=>v;
	
	//上面的箭头函数等同于：
	var f=function(v) {
		return v;
	};

>如果箭头函数不需要参数或需要多个参数，就使用一个圆括号代表参数部分。

	var f=()=>5;
	//等同于
	var f=function (){ return 5; }

	var sum=(num1,num2) => num1+num2;
	//等同于
	var sum=function(num1,num2) {
		return num1+num2;
	};
>如果箭头函数的代码块部分多于一条语句，就要使用大括号将它们括起来，并且使用return语句返回。

	var sum=(num1,num2)=>{return num1+num2;}

>由于大括号被解释为代码块，所以如果箭头函数直接返回一个对象，必须在对象外面加上括号。

	var getTempItem=id=>({id:id,name:"Temp"});
>箭头函数可以与变量解构结合使用。

	const full=({first,last}) => first+' '+last;

	//等同于
	function full(person) {
		return person.first+' '+person.name;
	}

>箭头函数使得表达更加简洁。
	
	const isEven=n=>n%2==0;
	const square=n=>n*n;

>箭头函数的一个用处是简化回调函数。
	
	//正常函数写法
	[1,2,3].map(function(x) {
		return x*x;
	});

	//箭头函数写法
	[1,2,3].map(x=>x*x);

>rest参数与箭头函数结合的例子:
	
	const numbers=(...nums) => nums;

	console.log(numbers(1,2,3,4,5));//[1,2,3,4,5]

	const headAndTail=(head,...tail) => [head,tail];
	
	console.log(headAndTail(1,2,3,4,5));//[1,[2,3,4,5]]
	
_使用注意点_
>箭头函数有几个使用注意点：

	(1)函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。
	(2)不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误。
	(3)不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用Rest参数代替。
	(4)不可以使用yield命令，因此箭头函数不能用作Generator函数。

>`this`对象的指向是可变的，但是在箭头函数中，它是固定的。

	function foo() {
		setTimeout(() => {
			console.log("id:",this.id);
		},100);
	}

	foo.call({id:42});//id:42

>`this`指向的固定化，并不是因为箭头函数内部有绑定`this`的机制，实际原因是箭头函数根本没有自己的`this`，导致内部的`this`就是外层代码块的`this`。正是因为它没有`this`，所以也就不能用作构造函数。

	function foo() {
	   return () => {
	      return () => {
	         return () => {
	            console.log("id:", this.id);
	         };
	      };
	   };
	}

	foo.call( { id: 42 } )()()();// id: 42
>上面代码之中，只有一个`this`，就是函数`foo`的`this`。因为所有的内层函数都是箭头函数，都没有自己的`this`，所以它们的`this`其实都是最外层`foo`函数的`this`。

>除了`this`,以下三个变量在箭头函数之中也不存在的，指向外层函数的对应变量：`arguments`、`super`、`new.target`。

	function foo() {
		setTimeout(()=>{
			console.log("args:",arguments);
		},100);
	}

	foo(2,4,6,8);//args: { '0': 2, '1': 4, '2': 6, '3': 8 }--箭头函数内部的变量arguments，其实是函数foo的arguments变量。
	
>由于箭头函数没有自己的`this`,所以当然也就不能用`call()`、`apply()`、`bind()`这些方法去改变`this`的指向。

	(function() {
	  console.log( [
	    (() => this.x).bind({ x: 'inner' })()
	  ]);
	}).call({ x: 'outer' });//['outer']

>JavaScript语言的this对象一直是一个令人头痛的问题，在对象方法中使用`this`，必须非常小心。箭头函数”绑定”`this`，很大程度上解决了这个困扰。

###嵌套的箭头函数
>箭头函数内部，还可以再使用箭头函数。

	function insert(value) {
		return {into:function(array) {
			return {after:function(afterValue) {
				array.splice(array.indexOf(afterValue)+1,0,value);
				return array;
			}};
		}};
	}

	console.log(insert(2).into([1,3]).after(1));//[1,2,3]
>改写成箭头函数：

	let insert=(value) => ({into:(array) => ({after:(afterValue) => {
		array.splice(array.indexOf(afterValue)+1,0,value);
		return array;
	}})});


	console.log(insert(2).into([1,3]).after(1));//[1,2,3]

>部署管道机制，即前一个函数的输出是后一个函数的输入:

	const pipeline=(...funcs) =>
		val => funcs.reduce((a,b) => b(a),val);

	const plus1=a => a + 1;
	const mult2=a => a * 2;
	const addThenMult = pipeline(plus1,mult2);

	console.log(addThenMult(5));//12

>箭头函数还有一个功能，就是可以很方便地改写λ演算。

	//λ演算的写法
	fix = λf.(λx.f(λv.x(x)(v)))(λx.f(λv.x(x)(v)));

	//ES6的写法
	var fix=f=>(x => f(v => x(x)(v)))
               (x => f(v => x(x)(v)));

##6、函数绑定
>箭头函数可以绑定`this`对象，大大减少了显式绑定`this`对象的写法(`call`,`apply`,`bind`)。但是，箭头函数并不适用于所有场合，所以ES7提出了"函数绑定"运算符，用来取代`call`、`apply`、`bind`调用。虽然该语法还是ES7的一个提案，但是Babel转码器已经支持。

>函数绑定运算符是并排的两个双冒号(::)，双冒号左边是一个对象，右边是一个函数。该运算符会自动将左边的对象，作为上下文环境(即this对象)，绑定到右边的函数上面。

	foo::bar;
	// 等同于
	bar.bind(foo);
	
	foo::bar(...arguments);
	// 等同于
	bar.apply(foo, arguments);
	
	const hasOwnProperty = Object.prototype.hasOwnProperty;

	function hasOwn(obj, key) {
	  return obj::hasOwnProperty(key);
	}

>如果双冒号左边为空，右边是一个对象的方法，则等于将该方法绑定在该对象上面。

	var method = obj::obj.foo;
	// 等同于
	var method = ::obj.foo;
	
	let log = ::console.log;
	// 等同于
	var log = console.log.bind(console);
>由于双冒号运算符返回的还是原对象，因此可以采用链式写法。

	// 例一
	import { map, takeWhile, forEach } from "iterlib";
	
	getPlayers()
	::map(x => x.character())
	::takeWhile(x => x.strength > 100)
	::forEach(x => console.log(x));
	
	// 例二
	let { find, html } = jake;
	
	document.querySelectorAll("div.myClass")
	::find("p")
	::html("hahaha");

##7、尾调用优化
###什么是尾调用？
>尾调用(Tail Call)是函数式编程的一个重要概念，本身非常简单，一句话就能说清楚，就是指某个函数的最后一步是调用另一个函数。

	function f(x){
	  return g(x);
	}

>以下三种情况，都不属于尾调用：

	//情况一
	function f(x) {
		let y=g(x);
		return y;
	}

	//情况二
	function f(x) {
		return g(x)+1;
	}

	//情况三
	function f(x) {
		g(x);
	}

>尾调用不一定出现在函数尾部，只要是最后一步操作即可。

	function f(x) {
	  if (x > 0) {
	    return m(x)
	  }
	  return n(x);
	}
	//上面代码中，函数m和n都属于尾调用，因为它们都是函数f的最后一步操作。

###尾调用优化
>尾调用之所以与其他调用不同，就在于它的特殊的调用位置。

>函数调用会在内存形成一个“调用记录”，又称“调用帧”(`call frame`)，保存调用位置和内部变量等信息。如果在函数A的内部调用函数B，那么在A的调用帧上方，还会形成一个B的调用帧。等到B运行结束，将结果返回到A，B的调用帧才会消失。如果函数B内部还调用函数C，那就还有一个C的调用帧，以此类推。所有的调用帧，就形成一个“调用栈”(`call stack`)。

>尾调用由于是函数的最后一步操作，所以不需要保留外层函数的调用帧，因为调用位置、内部变量等信息都不会再用到了，只要直接用内层函数的调用帧，取代外层函数的调用帧就可以了。


>“尾调用优化”(`Tail call optimization`)，即只保留内层函数的调用帧。如果所有函数都是尾调用，那么完全可以做到每次执行时，调用帧只有一项，这将大大节省内存。这就是“尾调用优化”的意义。
	
>只有不再用到外层函数的内部变量，内层函数的调用帧才会取代外层函数的调用帧，否则就无法进行“尾调用优化”。

	function addOne(a){
	  var one = 1;
	  function inner(b){
	    return b + one;
	  }
	  return inner(a);
	}
>上面的函数不会进行尾调用优化，因为内层函数inner用到了，外层函数addOne的内部变量one。

###尾递归
>函数调用自身，称为递归。如果尾调用自身，就称为尾递归。

>递归非常耗费内存，因为需要同时保存成千上百个调用帧，很容易发生“栈溢出”错误(`stack overflow`)。但对于尾递归来说，由于只存在一个调用帧，所以永远不会发生“栈溢出”错误。

	function factorial(n) {
	  if (n === 1) return 1;
	  return n * factorial(n - 1);
	}
	
	console.log(factorial(5)); // 120

>如果改写成尾递归，只保留一个调用记录，复杂度 O(1) 。

	function factorial(n, total) {
	  if (n === 1) return total;
	  return factorial(n - 1, n * total);
	}
	
	console.log(factorial(5, 1)); // 120
>“尾调用优化”对递归操作意义重大，所以一些函数式编程语言将其写入了语言规格。ES6也是如此，第一次明确规定，所有ECMAScript的实现，都必须部署“尾调用优化”。这就是说，在ES6中，只要使用尾递归，就不会发生栈溢出，相对节省内存。

>只有开启严格模式，尾调用优化才会生效。由于一旦启用尾调用优化，`func.arguments`和`func.caller`这两个函数内部对象就失去意义了，因为外层的帧会被整个替换掉，这两个对象包含的信息就会被移除。严格模式下，这两个对象也是不可用的。

	function restricted() {
	  "use strict";
	  restricted.caller;    // 报错
	  restricted.arguments; // 报错
	}
	restricted();
###递归函数的改写
>尾递归的实现，往往需要改写递归函数，确保最后一步只调用自身。做到这一点的方法，就是把所有用到的内部变量改写成函数的参数。
>

_方法一:在尾递归函数之外，再提供一个正常形式的函数。_

	function tailFactorial(n, total) {
	  if (n === 1) return total;
	  return tailFactorial(n - 1, n * total);
	}
	
	function factorial(n) {
	  return tailFactorial(n, 1);
	}
	
	console.log(factorial(5)); // 120

_方法二：柯里化(currying)，意思是将多参数的函数转换成单参数的形式。_

	function currying(fn,n) {
		return function (m) {
			return fn.call(this,m,n);
		};
	}

	function tailFactorial(n,total) {
		if(n===1) return total;
		return tailFactorial(n-1,n*total);	
	}

	const factorial=currying(tailFactorial,1);

	console.log(factorial(5));//120

_方法三：采用ES6的函数默认值。_

	function factorial(n, total = 1) {
	  if (n === 1) return total;
	  return factorial(n - 1, n * total);
	}
	
	console.log(factorial(5)); // 120

##8、函数参数的尾逗号
>ES7有一个提案，允许函数的最后一个参数有尾逗号(trailing comma)。目前，函数定义和调用时，都不允许有参数的尾逗号。

	





	















