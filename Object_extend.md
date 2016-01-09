#对象的扩展
#
##1、属性的简洁表示法
>ES6允许直接写入变量和函数，作为对象的属性和方法。这样的书写更加简洁。

	var foo='bar';
	var baz={foo};
	console.log(bar);//{foo:"bar"}

	//等同于 
	var baz={foo:foo};
>除了属性可以简写，方法也可以简写。

	var o={
		method() {
			return "Hello!";
		}
	};

	console.log(o.method());//"Hello!"

	//等同于
	var o={
		method:function() {
			return "Hello!";
		}
	};

>简写例子：

	var birth='2000/01/01';
	
	var Person={
		name:'张三',
		//等同于birth:birth
		birth,
		//等同于hello:function()...
		hello() {
			console.log('我的名字是',this.name);
		}
	};

	console.log(Person.name);//张三
	console.log(Person.birth);//2000/01/01
	Person.hello();//我的名字是 张三

>这种写法对于函数的返回值，将会非常方便。

	function getPoint() {
		var x=1;
		var y=10;
		return {x,y};
	}

	console.log(getPoint());//{x:1,y:10}
>CommonJS模块输出变量，就非常合适使用简洁写法。

	var ms={};

	function getItem(key) {
		return key in ms ? ms[key] :null;
	}

	function setItem(key,value) {
		ms[key]=value;
	}

	function clear() {
		ms={};
	}


	module.exports={getItem,setItem,clear};

	//等同于
	module.exports={
		getItem:getItem,
		setItem:setItem,
		clear:clear
	};

>属性的赋值器和取值器，事实上也是采用这种写法。

	var cart={
		_wheels:4,

		get wheels () {
			return this._wheels;
		},

		set wheels (value) {
			if(value < this._wheels) {
				throw new Error('数值太小了!');
			}
			this._wheels=value;
		}
	}

>如果某个方法的值是一个Generator函数，前面需要加上星号。

	var obj={
		* m(){
			yield 'hello world';
		 }
	}

##2、属性名表达式
>JavaScript语言定义对象的属性，有两种方法。

	//方法一
	obj.foo=true;

	//方法二
	obj['a'+'bc']=123;

>如果使用字面量方式定义对象，ES5中只能使用方法一(标识符)定义属性。

	var obj={
		foo:true,
		abc:123
	};

>ES6允许字面量定义对象时，用方法二(表达式)作为对象的属性名，即把表达式放在括号内。

	let propKey='foo';

	let obj={
		[propKey]:true,
		['a'+'bc']:123
	};

>另一个例子:

	var lastWord='last word';

	var a={
		'first word':'hello',
		[lastWord]:'world'
	};

	console.log(a['first word']);//'hello'
	console.log(a[lastWord]);//'world'
	console.log(a['last word']);//'world'

>表达式还可以用于定义方法名。
	
	let obj={
		['h'+'ello']() {
			return 'hi';
		}
	};

	console.log(obj.hello());//'hi'

>属性名表达式与简洁表示法，不能同时使用，会报错。

	//报错
	var foo='bar';
	var bar='abc';
	var baz={[foo]};

	//正确
	var foo='bar';
	var baz={[foo]:'abc'};
	console.log(baz);//{ bar: 'abc' }

##3、方法的name属性
>函数的`name`属性，返回函数名。对象方法也是函数，因此也有`name`属性。

	var person={
		sayName() {
			console.log(this.name);
		},
		get firstName() {
			return 'Nicholas';
		}
	}


	console.log(person.sayName.name);//"sayName"
	console.log(person.firstName.name);//"get firstName"
>上面代码中，方法的`name`属性返回函数名。如果使用了取值函数，则会在方法名前加上`get`。如果是存值函数，方法名的前面会加上`set`。


>有两种特殊情况：`bind`方法创造的函数，`name`属性返回"bound"加上原函数的名字；`Function`构造函数创造的函数，`name`属性返回"anonymous"。

	console.log((new Function()).name);//"anonymous"

	var doSomething=function() {
		//...
	};

	console.log(doSomething.bind().name);//"bound doSomething"

>如果对象的方法是一个Symbol值，那么`name`属性返回的是这个Symbol值的描述。

	const key1=Symbol('descripgion');
	const key2=Symbol();
	let obj={
		[key1](){},
		[key2](){},
	};

	console.log(obj[key1].name);//"[description]"
	console.log(obj[key2].name);//""

##4、Object.is()
>ES5比较两个值是否相等，只有两个运算符：相等运算符(`==`)和严格相等运算符(`===`)。它们都有缺点，前者会自动转换数据类型，后者的`NaN`不等于自身，以及`+0`等于`-0`。JavaScript缺乏一种运算，在所有环境中，只要两个值是一样的，它们就应该相等。


>ES6提出"Same-value equality"(同值相等)算法，用来解决这个问题。`Object.is`就是部署这个算法的新方法。它用来比较两个值是否严格相等，与严格比较运算符(===)的行为基本一致。

	console.log(Object.is('foo','foo'));//true

	console.log(Object.is({},{}));//false

>不同之处有两个:一是`+0`不等于`-0`,二是`NaN`等于自身。

	console.log(+0===-0);//true
	console.log(NaN===NaN);//false

	console.log(Object.is(+0,-0));//false
	console.log(Object.is(NaN,NaN));//true

>ES5可以如下部署`Object.is`:

	Object.defineProperty(Object,'is',{
		value:function(x,y) {
			if(x===y) {
				//针对+0不等于-0的情况
				return x!==0||1/x==1/y;
			}
			//针对NaN的情况
		},
		configurable:true,
		enumerable:false,
		writable:true
	});

##5、Object.assign()
>`Object.assign`方法用来将源对象(source)的所有可枚举属性，复制到目标对象(target)。它至少需要两个对象作为参数，第一个参数是目标对象，后面的参数都是源对象。只要有一个参数不是对象，就会抛出TypeError错误。

	var target={a:1};
	
	var source1={b:2};
	var source2={c:3};

	Object.assign(target,source1,source2);
	console.log(target);//{a:1,b:2,c:3}

>如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性会覆盖前面的属性。

	var target={a:1,b:1};
	
	var source1={b:2,c:2};
	var source2={c:3};

	Object.assign(target,source1,source2);
	console.log(target);//{a:1,b:2,c:3}

>`Object.assign`只拷贝自身属性，不可枚举的属性(`enumerable`为false)和继承的属性不会被拷贝。

	Object.assign({b:'c'},
		Object.defineProperty({},'invisible',{
			enumerable:false,
			value:'hello'
		})
	);//{b:'c'}

>属性名为Symbol值的属性，也会被`Object.assign`拷贝。

	Object.assign({a:'b'},{[Symbol('c')]:'d'});//{ a: 'b', Symbol(c): 'd' }

>对于嵌套的对象，`Object.assign`的处理方法是替换，而不是添加。

	var target={a:{b:'c',d:'e'}};
	var source={a:{b:'hello'}};
	Object.assign(target,source);//{ a: { b: 'hello' } }
>注意，`Object.assign`可以用来处理数组，但是会把数组视为对象。

	Object.assign([1,2,3],[4,5]);//[4,5,3]	
	//Object.assign把数组视为属性名为0、1、2的对象，因此目标数组的0号属性4覆盖了原数组的0号属性1。

__(1)为对象添加属性__
	
	class Point {
		constructor(x,y) {
			Object.assign(this,{x,y});
		}
	}
	//通过assign方法，将x属性和y属性添加到Point类的对象实例。

__(2)为对象添加方法__
	
	Object.assign(SomeClass.prototype,{
		someMethod(arg1,arg2) {
			//...
		},
		anotherMethod() {
			//...
		}
	});

	//等同于下面的写法
	SomeClass.prototype.someMethod=function(arg1,arg2) {
		//...
	};

	SomeClass.prototype.anotherMethod=function() {
		//...
	};

__(3)克隆对象__
	
	function clone(origin) {
		return Object.assign({},origin);
	}

>采用这种方法克隆，只能克隆原始对象自身的值，不能克隆它继承的值。如果想要保持继承链，可以采用下面的代码。

	function clone(origin) {
		let originProto=Object.getPrototypeOf(origin);
		return Object.assign(Object.create(originProto),origin);
	}

	
__(4)合并多个对象__
>将多个对象合并到某个对象。

	const merge=(target,...source) => Object.assign(target,...sources);

>如果希望合并后返回一个新对象，可以改写上面函数，对一个空对象合并。

	const merge=(...sources) => Object.assign({},...sources);

__(5)为属性指定默认值__
	
	const DEFAULTS={
		logLevel:0,
		outputFormat:'html'
	};

	function processContent(options) {
		let options=Object.assign({},DEFAULTS,options);
	}

>上面代码中，`DEFAULTS`对象是默认值，`options`对象是用户提供的参数。`Object.assign`方法将`DEFAULTS`和`options`合并成一个新对象，如果两者有同名属性，则`option`的属性值会覆盖`DEFAULTS`的属性值。

>由于存在深拷贝的问题，`DEFAULTS`对象和`options`对象的所有属性的值，都只能是简单类型，而不能指向另一个对象。否则，将导致`DEFAULTS`对象的该属性不起作用。
	
##6、属性的可枚举性
>对象的每个属性都有一个描述对象，用来控制该属性的行为。`Object.getOwnPropertyDescriptor`方法可以获取该属性的描述对象。

	let obj={foo:123};
	Object.getOwnPropertyDescriptor(obj,'foo');

	//{ value: 123,
	// writable: true,
	// enumerable: true,
	// configurable: true }

>描述对象的`enumerable`属性，称为"可枚举性"，如果该属性为`false`,就可以某些操作会忽略当前属性。

>ES5有三个操作会忽略`enumerable`为`false`的属性。
		
	-for...in循环:只遍历对象自身的和继承的可枚举的属性
	-Object.keys():返回对象自身的所有可枚举的属性的键名
	-JSON.stringify():只串行化对象自身的可枚举的属性

>ES6新增了两个操作,会忽略`enumerable`为`false`的属性。

	-Object.assign():只拷贝对象自身的可枚举的属性
	-Reflect.enumerate():返回所有`for...in`循环会遍历的属性
>这五个操作之中，只有`for...in`和`Reflect.enumerate()`会返回继承的属性。引入`enumerable`的最初目的，就是让某些属性可以规避掉`for...in`操作。比如，对象原型的`toString`方法，以及数组的`length`属性，就通过这种手段，不会被`for...in`遍历到。

	console.log(Object.getOwnPropertyDescriptor(Object.prototype,'toString').enumerable);//false


	console.log(Object.getOwnPropertyDescriptor([],'length').enumerable);//false

>另外，ES6规定，所有Class的原型的方法都是不可枚举的。

	console.log(Object.getOwnPropertyDescriptor(class {foo() {}}.prototype,'foo').enumerable);//false

##7、属性的遍历
>ES6一共有6种方法可以遍历对象的属性。

__(1)for...in__
>`for...in`循环遍历对象自身的和继承的可枚举属性(不含Symbol属性)。

__(2)Object.keys(obj)__
>`Object.keys`返回一个数组，包括对象自身的(不含继承的)所有可枚举属性(不含Symbol属性)。

__(3)Object.getOwnPropertyNames(obj)__
>`Object.getOwnPropertyNames`返回一个数组，包含对象自身的所有属性(不含Symbol属性，但是包括不可枚举属性)。

__(4)Object.getOwnPropertySymbols(obj)__
>`Object.getOwnPropertySymbols`返回一个数组，包含对象自身的所有Symbol属性。

__(5)Reflect.ownKeys(obj)__
>`Reflect.ownKeys`返回一个数组，包含对象自身的所有属性，不管是属性名是Symbol或字符串，也不管是否可枚举。

__(6)Reflect.enumerate(obj)__
>`Reflect.enumerate`返回一个Iterator对象，遍历对象自身的和继承的所有可枚举属性(不含Symbol属性)，与`for...in`循环相同。

>以上的6种方法遍历对象的属性，都遵守同样的属性遍历的次序规则。

	-首先遍历所有属性名为数值的属性，按照数字排序。
	-其次遍历所有属性名为字符串的属性，按照生成时间排序。
	-最后遍历所有属性名为Symbol值的属性，按照生成时间排序。

	console.log(Reflect.ownKeys({[Symbol()]:0,b:0,10:0,2:0,a:0}));// ['2', '10', 'b', 'a', Symbol()]

##8、__proto__属性，Object.setPrototypeOf(),Object.getPrototypeOf()

__(1)`__proto__`__属性
>`__proto__`属性，用来读取或设置当前对象的`prototype`对象。目前，所有浏览器都部署了这个属性。

	//ES6的写法
	var obj={
		method:function() {//...}
	}

	obj.__proto__=someOtherObj;

	//ES5写法
	var obj=Object.create(someOtherObj);
	obj.method=function() {//...}

>标准明确规定，只有浏览器必须部署这个属性，其他运行环境不一定需要部署，而且新的代码最好认为这个属性是不存在的。因此，无论从语义的角度，还是从兼容性的角度，都不要使用这个属性，而是使用下面的`Object.setPrototypeOf()`(写操作)、`Object.getPrototypeOf()`(读操作)、`Object.create()`(生成操作)代替。

>实现上，`__proto__`调用的是`Object.prototype.__proto__`,具体实现如下：

	Object.defineProperty(Object.prototype,'__proto__',{
		get() {
			let _thisObj=Object(this);
			return Object.getPrototypeOf(_thisObj);
		},
		set(proto) {
			if(this === undefined || this===null) {
				throw new TypeError();
			}

			if(!isObject(this)) {
				return undefined;
			}
			
			if(!isObject(proto)) {
				return undefined;
			}
			
			let status=Reflect.setPrototypeOf(this,proto);
			if(!status) {
				throw new TypeError();		
			}
		},
	});

	function isObject(value) {
		return Object(value) ===value;
	}

>如果一个对象本身部署了`__proto`属性，则该属性的值就是对象的原型。

	console.log(Object.getPrototypeOf({__proto__:null}));//null

__(2)Object.setPrototypeOf()__
>`Object.setPrototypeOf`方法的作用与`__proto__`相同，用来设置一个对象的`prototype`对象。

	//格式
	Object.setPrototypeOf(object,prototype);

	//用法
	var o=Object.setPrototypeOf({},null);
>该方法等同于下面的函数：

	function (obj,proto) {
		obj.__proto__=proto;
		return obj;
	}
>下面的例子：

	let proto={};
	let obj={x:10};
	Object.setPrototypeOf(obj,proto);

	proto.y=20;
	proto.z=40;

	console.log(obj.x);//10
	console.log(obj.y);//20
	console.log(obj.z);//40

__(3)Object.getPrototypeOf()__
>该方法与setPrototypeOf方法配套，用于读取一个对象对prototype对象。
	
	Object.getPrototypeOf(obj);

>下面是个例子：

	function Rectangle() {
		
	}

	var rec=new Rectangle();

	console.log(Object.getPrototypeOf(rec)===Rectangle.prototype);//true

	console.log(Object.setPrototypeOf(rec,Object.prototype));//{}

	console.log(Object.getPrototypeOf(rec)===Rectangle.prototype);//false

##9、对象的扩展运算符
>ES7有一个提案，将rest参数/扩展运算符(...)引入对象。Babel转码器已经支持这项功能。

__(1)Rest参数__
>Rest参数用于从一个对象取值，相当于将所有可遍历的、但尚未被读取的属性，分配指定的对象上面。所有的键和它们值，都会拷贝到新对象上面。

	let {x,y,...z}={x:1,y:2,a:3,b:4};
	console.log(x);//1
	console.log(y);//2
	console.log(z);//Object {a:3,b:4}

>Rest参数的拷贝是浅拷贝，即如果一个键的值是复合类型的值(数组、对象、函数)，那么Rest参数拷贝的是这个值的引用，而不是这个值的副本。

	let obj={a:{b:1}};
	let {...x}=obj;
	obj.a.b=2;
	console.log(x.a.b);//2
	
>Rest参数不会拷贝继承自原型对象的属性。

	let o1={a:1};
	let o2={b:2};
	o2.__proto__=o1;
	let o3={...o2};
	console.log(o3);//Object {b: 2}

__(2)扩展运算符__
>扩展运算符用于取出参数对象的所有可遍历属性，拷贝到当前对象之中。

	let z={a:3,b:4};
	let n={...z};
	console.log(n);//Object {a:3,b:4}
	
>这等同于使用`Object.assign`方法：

	let aClone={...a};

	//等同于
	let aClone=Object.assign({},a);
>扩展运算符可以用于合并两个对象。

	let ab={...a,...b};
>扩展运算符还可以用自定义属性，会在新对象之中，覆盖掉原有参数。 

	let aWithOverrides={...a,x:1,y:2};

	//等同于
	let aWithOverrides={...a,...{x:1,y:2}};

	//等同于
	let x=1,y=2,aWithOverrides={...a,x,y};

	//等同于
	let aWithOverrides=Object.assign({},a,{x:1,y:2});

>如果把自定义属性放在扩展运算符前面，就变成了设置新对象的默认属性值。

	let aWithDefaults={x:1,y:2,...a};

	//等同于	
	let aWithDefaults=Object.assign({},{x:1,y:2},a);
	
	//等同于
	let aWithDefaults=Object.assign({x:1,y:2},a);

>扩展运算符的参数对象之中，如果有取值函数`get`,这个函数是会执行的。
	
	// 并不会抛出错误，因为x属性只是被定义，但没执行
	let aWithXGetter = {
	  ...a,
	  get x() {
	    throws new Error('not thrown yet');
	  }
	};
	
	// 会抛出错误，因为x属性被执行了
	let runtimeError = {
	  ...a,
	  ...{
	    get x() {
	      throws new Error('thrown now');
	    }
	  }
	};	

>如果扩展运算符的参数是null或undefined，这个两个值会被忽略，不会报错。

	let emptyObject = { ...null, ...undefined }; // 不报错

	
	
	
		






	








	