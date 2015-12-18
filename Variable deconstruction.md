#变量的解构赋值
#
##1、数组的解构赋值
###基本用法
>ES6允许按照一定模式，从数组和对象中提取值，对变量进行赋值，称为解构。

	var [a,b,c]=[1,2,3];
	//分别为a,b,c赋值1，2，3

>本质上，此写法类似于"模式匹配",只要等号两边的模式相同，左边的变量会被赋予对应的值。

	let [,,third]=["foo","bar","baz"];
	console.log(third);//"baz"

	let [head,...tail]=[1,2,3,4];
	console.log(tail);//[2,3,4]

	let [x,y,...z]=['a'];
	console.log(x);//"a"
	console.log(y);//undefined
	console.log(z);//[]
>如果解构不成功，变量的值就等于`undefined`.

	var [foo]=[];
	console.log(foo);//undefined
	var [bar,foo]=[1];
	console.log(foo);//undefined
>不完全解构，即等号左边的模式，只匹配一部分的等号右边的数组。这种情况下，解构依然成功。

	let [x,y]=[1,2,3];
	console.log(x);//1
	console.log(y);//2

	let [a,[b],c]=[1,[2,3],4];
	console.log(b);//2
>等号右边不是数组(不是可遍历的结构)，将会报错。

	let [foo] = 1;
	let [foo] = false;
	let [foo] = NaN;
	let [foo] = undefined;
	let [foo] = null;
	let [foo] = {};
	//等号右边的值，要么转为对象后不具备Iterator接口，要么本身不具备Iterator接口。
>解析赋值适用于`var`,`let`,`const`命令,对`Set`结构，也可以使用数组的解构赋值。

	var [v1,v2,...,vn]=array;
	let [v1,v2,...,vn]=array;
	const [v1,v2,...,vn]=array;
	let [x,y,z]=new Set(["a","b","c"]);

>只要某种数据结构具有Iterator接口，都可以采用数组形式的解构赋值。

	function* fibs() {
	  var a = 0;
	  var b = 1;
	  while (true) {
	    yield a;
	    [a, b] = [b, a + b];
	  }
	}
	
	var [first, second, third, fourth, fifth, sixth] = fibs();
	console.log(sixth); // 5
	//fibs是一个Generator函数，原生具有Iterator接口。解构赋值会依次从这个接口获取值。

###默认值
>解构赋值允许指定默认值。

	
	var [foo=true]=[];
	console.log(foo);//true
	
	var [x,y='b']=['a',undefined];
	console.log(x);//'a'
	console.log(y);//'b'

>ES6内部使用严格相等运算符（`===`）,判断一个位置是否有值。如果一个数组成员不严格等于`undefined`，默认值是不会生效的。

	var [x=1]=[undefined];
	console.log(x);//1

	var [x=1]=[null];
	console.log(x);//null

>如果默认值是一个表达式，那么表达式是惰性求值的，即只有在用到的时候，才会求值。

	function f() {
		console.log('aaa');
	}
	
	let [x=f()]=[1];
	console.log(x);//1


>默认值可以引用解构赋值的其他变量，但该变量必须已经声明。

	let [x=1,y=x]=[];
	console.log(x);//1
	console.log(y);//1

	let [x=1,y=x]=[1,2];
	console.log(x);//1
	console.log(y);//2

	let [x=y,y=1]=[];
	console.log(x);//ReferenceError
	console.log(y);

##2、对象的解构赋值
>解构不仅可以用于数组，还可以用于对象。

	var {foo,bar}={foo:"abc",bar:"def"};
	console.log(foo);//"abc"
	console.log(bar);//"def"
>对象的解构和数组有一个重要的不同。数组的元素是按次序排列的，变量的取值由它的位置决定。而对象的属性没有次序，变量必须与属性同名，才能取到正确的值。

	var {bar,foo}={foo:"aaa",bar:"bbb"};
	console.log(foo);//"aaa"
	console.log(bar);//"bbb"

>对象的解构赋值的内部机制，是先找到同名属性，然后再赋给对应的变量。真正被赋值的是后者，而不是前者。如果变量名与属性名不一致，必须写成下面这样：

	var {foo:baz}={foo:"aaa",bar:"bbb"};
	console.log(baz);//"aaa"
	console.log(foo);//ReferenceError: foo is not defined

>对于`let`和`const`来说，变量不能重新声明，所以一旦赋值的变量以前声明过，就会报错。


	let foo;
	let {foo}={foo:1};//Duplicate declaration "foo"
	
>如果没有第二个`let`命令，上述代码将成功运行。

	let foo;
	({foo}={foo:1});
	console.log(foo);//1
>和数组一样，解构也可用于嵌套结构的对象。


	var obj={
		p:[
		"Hello",
		{y:"world"}
		]
	};

	var  {p:[x,{y}]}=obj;//p是模式，不是变量，不会被赋值
	console.log(x);//"hello"
	console.log(y);//"world"
	
>对象的解构也可以指定默认值。默认值生效的条件是，对象的属性值严格等于`undefined`。

	var {x,y=5}={x:1};
	console.log(x);//1
	console.log(y);//5

	var {x=3}={x:undefined};
	console.log(x);//3
	
	var {x=3}={x:null};
	console.log(x);//null
>如果解构失败，变量的值等于`undefined`。

	var {foo}={baz:'baz'};
	console.log(foo);//undefined
>如果解构模式是嵌套的对象，而且子对象所在的父属性不存在，那么会报错。
	
	var {foo:{bar}}={baz:'baz'};// TypeError: Cannot read property 'bar' of undefined

	var _tmp={baz:'baz'};
	console.log(_tmp.foo.baz);// TypeError: Cannot read property 'baz' of undefined

>JavaScript引擎会将`{x}`理解成一个代码块。只有不将大括号写在行首，避免JavaScript将其解释为代码块。因此将一个已经声明的变量用于解构赋值时，必须非常小心。

	//error style
	var x;
	{x}={x:1};//SyntaxError
	
	
	//correct style
	var x;
	({x}={x:1});
	console.log(x);//1 

>解构赋值允许，等号左边的模式之中，不放置任何变量名。

	({}=[true,false]);
	({}='abc');
	({}=[]);

##3、字符串的解构赋值
>字符串也可以解构赋值。此时字符串被转换成了一个类似数组的对象。

	const [a,b,c,d,e]='hello';
	console.log(a);//'h'
	console.log(b);//'e'
	console.log(c);//'l'
	console.log(d);//'l'
	console.log(e);//'o'

>类数组都有一个`length`属性，因此还可以对这个属性解构赋值。

	let {length:len}='hello';
	console.log(len);//5

##4、数值和布尔值的解构赋值
> 解构赋值时，如果等号右边是数值和布尔值，则会先转为对象。

	let {toString:s}=123;
	console.log(s===Number.prototype.toString);//true

	let {toString:s}=true;
	console.log(s===Boolean.prototype.toString);//true
>`undefined`和`null`无法转为对象，所以对它们进行解构赋值，都会报错。

	let {prop:x}=undefined;//TypeError
	let {prop:y}=null;//TypeError

##5、函数参数的解构赋值
>函数参数也可以使用解构赋值。

	var arr=[[1,2],[3,4]].map(([a,b]) => a+b);
	console.log(arr);//[3,7]
>函数参数的解构也可以使用默认值。

	//以下为fn的参数中的x,y指定默认值	
	function fn({x=0,y=0}={}) {
		return [x,y];
	}

	console.log(fn({x:3,y:8}));//[3,8]
	console.log(fn({x:3}));//[3,0]
	console.log(fn());//[0,0]

	//以下为fn的参数指定默认值
	function fn({x,y}={x:0,y:0}) {
		return [x,y];
	}

	console.log(fn({x:3,y:8}));//[3,8]
	console.log(fn({x:3}));//[3,undefined]
	console.log(fn());//[0,0]
	
	//undefined会触发函数参数的默认值
	var arr=[1,undefined,3].map((x='yes')=>x);
	console.log(arr);//[1,"yes",3]

##6、圆括号
>对于编译器来说，一个式子到底是模式，还是表达式，没有办法从一开始就知道，必须解析到（或解析不到）等号才能知道。ES6的规则是，只要有可能导致解构的歧义，就不得使用圆括号。

###不能使用圆括号的情况
_(1) 变量声明语句中，模式不能带圆括号_
	
	var [(a)]=[1];//SyntaxError
	var {x:(c)}={};//SyntaxError
	var {o:({p:p})}={o:{p:2}};//SyntaxError

_(2) 函数参数中，模式不能带有圆括号。_
>函数声明也属于变量声明，不能带有圆括号。

	function fn([(z)]) {return z;}//SyntaxError
_(3) 不能将整个模式，或嵌套模式中的一层，放在圆括号中。_
	
	({p:a})={p:42};//SyntaxError
	([a])=[5];//SyntaxError
	[{ p:(x:c)}] = [{}];//SyntaxError
###可以使用圆括号的情况
>可以使用圆括号的情况只有一种：赋值语句的非模式部分，可以使用圆括号。
	
	[(b)]=[3];
	({p:(c)}={});
	[(parseInt.prop)]=[3];

##7、解构的用途
###(1) 交换变量的值
	[x,y]=[y,x];
###(2) 从函数返回多个值
>函数只能返回一个值，如果要返回多个值，只能将它们放在数组或对象里返回。使用解构赋值，取出这些值就非常方便。

	//return an array
	function example() {
	  return [1, 2, 3];
	}
	var [a, b, c] = example();
	console.log([a,b,c]);//[1,2,3]

	//return an Object
	function example() {
	  return {
	    foo: 1,
	    bar: 2
	  };
	}
	var { foo, bar } = example();
	console.log({foo,bar});//{foo: 1, bar: 2}

###(3) 函数参数的定义
>解构赋值可以方便地将一组参数与变量名对应起来。

	// 参数是一组有次序的值
	function f([x, y, z]) { //code here }
	f([1, 2, 3])
	
	// 参数是一组无次序的值
	function f({x, y, z}) { //code here }
	f({z: 3, y: 2, x: 1})
###(4) 提取JSON数据
>解构赋值对提取JSON对象中的数据，尤其有用

	var jsonData = {
	  id: 42,
	  status: "OK",
	  data: [1, 2]
	}
	
	let { id, status, data: number } = jsonData;
	console.log(id, status, number)// 42, "OK", [1, 2]
###(5) 函数参数的默认值
	
	var fx=function (url,{
		async=true,
		method:'GET'
		cache=true,
		//more config
	}){
		//do stuff
	};
###(6) 遍历Map结构
>任何部署了`Iterator`接口的对象，都可以用`for...of`循环遍历。`Map`结构原生支持`Iterator`接口，配合变量的解构赋值，获取键名和键值就非常方便。
	
	var map = new Map();
	map.set('first', 'hello');
	map.set('second', 'world');
	
	for (let [key, value] of map) {
	  console.log(key + " is " + value);
	}// first is hello // second is world
	
	// get the key
	for (let [key] of map) {
	  // code here
	}
	
	// get the key-value
	for (let [value] of map) {
	  // code here
	}

###(7) 输入模块的指定方法
>加载模块时，往往需要指定输入那些方法。解构赋值使得输入语句非常清晰。


	const { SourceMapConsumer, SourceNode } = require("source-map");