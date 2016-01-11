#Symbol
#
##1、概述
>ES5的对象属性名都是字符串，这容易造成属性名的冲突。比如，使用了一个他人提供的对象，但又想为这个对象添加新的方法，新方法的名字就有可能与现有方法产生冲突。如果有一种机制，保证每个属性的名字都是独一无二的就好了，这样就从根本上防止属性名的冲突。

>ES6引入了一种新的原始数据类型Symbol,表示独一无二的值。它是JavaScript语言的第七种数据类型，前六种是：Undefined、Null、布尔值、字符串、数值、对象。

>Symbol值通过`Symbol`函数生成。对象的属性名现在可以有两种类型，一种是原来就有的字符串，另一种就是新增的Symbol类型。凡是属性名属于Symbol类型，就都是独一无二的，可以保证不会与其他属性名产生冲突。

	let s=Symbol();

	console.log(typeof s);//"symbol"
>`typeof`运算符的结果，表明变量`s`是Symbol数据类型，而不是字符串之类的其他类型。

>`Symbol`函数前不能使用`new`命令，否则会报错。这是因为生成的Symbol是一个原始类型的值，不是对象。由于Symbol值不是对象，所以不能添加属性。基本上，它是一种类似于字符串的数据类型。

>`Symbol`函数可以接受一个字符串作为参数，表示对Symbol实例的描述，主要是为了在控制台显示，或者转为字符串时，比较容易区分。

	var s1=Symbol('foo');
	var s2=Symbol('bar');

	console.log(s1);//Symbol(foo)
	console.log(s2);//Symbol(bar)

	console.log(s1.toString());//Symbol(foo)
	console.log(s2.toString());//Symbol(bar)

>`Symbol`函数的参数只是表示对当前Symbol值的描述，因此相同参数的`Symbol`函数的返回值是不相等的。

	//没有参数的情况
	var s1=Symbol();
	var s2=Symbol();

	console.log(s1===s2);//false

	//有参数的情况
	var s1=Symbol("foo");
	var s2=Symbol("foo");

	console.log(s1===s2);//false

>Symbol值不能与其他类型的值进行运算，会报错。

	var sym=Symbol('My symbol');

	console.log("your symbol is "+sym);//TypeError

>Symbol值可以显式转为字符串。

	var sym=Symbol('My symbol');

	console.log(String(sym));//'Symbol(My symbol)'
	console.log(sym.toString());// 'Symbol(My symbol)'

>Symbol值可以转为布尔值，但是不能转为数值。

	var sym=Symbol();
	console.log(Boolean(sym));//true
	console.log(!sym);//false

	if(sym) {
		//...
	}

	console.log(Number(sym));//TypeError
	console.log(sym+2);//TypeError

##2、作为属性名的Symbol
>由于每个Symbol值都是不相等的，这意味着Symbol值可以作为标识符，用于对象的属性名，就能保证不会出现同名的属性。这对于一个对象由多个模块构成的情况非常有用，能防止某一个健被不小心改成或覆盖。

	var mySymbol=Symbol();

	//第一种写法
	var a={};
	a[mySymbol]='Hello!';

	//第二种写法
	var a={
		[mySymbol]:'Hello!'
	};

	//第三种写法
	var a={};
	Object.defineProperty(a,mySymbol,{value:'Hello!'});

	//以上写法都得到同样结果
	console.log(a[mySymbol]);//"Hello!"

>Symbol值作为对象属性名时，不能用点运算符。

	var mySymbol=Symbol();
	var a={};

	a.mySymbol='Hello!';
	console.log(a[mySymbol]);//undefined
	console.log(a['mySymbol']);//"Hello!"

>在对象的内部，使用Symbol值定义属性时，Symbol值必须放在方括号之中。

	let s=Symbol();

	let obj={
		[s]:function (arg) {//...}
	};

	//增加的对象写法
	let obj={
		[s](arg) {//...}
	};

	console.log(obj[s](123));

>如果`s`不放在方括号中，该属性的键名就是字符串`s`,而不是`s`所代表的那个Symbol值。

>Symbol类型还可以用于定义一组常量，保证这组常量的值都是不相等的。

	const COLOR_RED    = Symbol();
	const COLOR_GREEN  = Symbol();

	function getComplement(color) {
	  switch (color) {
	    case COLOR_RED:
	      return COLOR_GREEN;
	    case COLOR_GREEN:
	      return COLOR_RED;
	    default:
	      throw new Error('Undefined color');
	    }
	}
>常量使用Symbol值最大的好处，就是其他任何值都不可能有相同的值了，因此可以保证上面的switch语句会按设计的方式工作。还有一点需要注意，Symbol值作为属性名时，该属性还是公开属性，不是私有属性。

##3、实例：消除魔术字符串
>魔术字符串指的是，在代码之中多次出现、与代码形成强耦合的某一个具体的字符串或者数值。风格良好的代码，应该尽量消除魔术字符串，该由含义清晰的变量代替。

	function getArea(shape,options) {
		var area=0;

		switch(shape) {
			case 'Triangle'://魔术字符串
			area=.5*options.width*options.height;
			break;
		/*...more code...*/
		}
		
		return area;
	}

	console.log(getArea('Triangle',{width:100,height:100}));//魔术字符串-5000

>常用的消除魔术字符串的方法，就是把它写成一个常量。

	var shapeType={
		triangle:'Triangle'
	};

	function getArea(shape,options) {
		var area=0;
		switch(shape) {
			case shapeType.triangle:
				area=.5*options.width*options.height;
				break;
		}
		return area;
	}

	console.log(getArea(shapeType.triangle,{width:100,height:100}));
>`shapeType.triangle`等于哪个值并不重要，只要确保不会跟其他`shapeType`属性的值冲突即可。因此，这里就很适合改用Symbol值。

	const shapeType={
		triangle:Symbol()
	};
	//除了将shapeType.triangle的值设为一个Symbol，其他地方都不用修改。

##4、属性名的遍历
>Symbol作为属性名，该属性不会出现在`for...in`、`for...of`循环中，也不会被`Object.keys()`、`Object.getOwnPropertyNames()`返回。但是，它也不是私有属性，有一个`Object.getOwnPropertySymbols`方法，可以获取指定对象的所有Symbol属性名。

>`Object.getOwnPropertySymbols`方法返回一个数组，成员是当前对象的所有用作属性名的Symbol值。

	var obj={};
	var a=Symbol('a');
	var b=Symbol.for('b');

	obj[a]='Hello';
	obj[b]='World';

	var objectSymbols=Object.getOwnPropertySymbols(obj);
	
	console.log(objectSymbols);//[Symbol(a), Symbol(b)]

>`Object.getOwnPropertySymbols`方法与`for...in`循环、`Object.getOwnPropertyNames`方法进行对比的例子。

	var obj={};

	var foo=Symbol('foo');

	Object.defineProperty(obj,foo,{
		value:'foobar'
	});

	for(var i in obj) {
		console.log(i);
	}

	console.log(Object.getOwnPropertyNames(obj));//[]

	console.log(Object.getOwnPropertySymbols(obj));//[Symbol(foo)]

>`Reflect.ownKeys`方法可以返回所有类型的键名，包括常规键名和Symbol键名。

	let obj={
		[Symbol('my_key')]:1,
		enum:2,
		nonEnum:3
	};

	console.log(Reflect.ownKeys(obj));// [Symbol(my_key), 'enum', 'nonEnum']
>由于以Symbol值作为名称的属性，不会被常规方法遍历得到。可以利用这个特性，为对象定义一些非私有的，但又希望只用于内部的方法。

	var size=Symbol('size');

	class Collection {
		constructor() {
			this[size]=0;
		}

		add(item) {
			this[this[size]]=item;
			this[size]++;
		}

		static sizeOf(instance) {
			return instance[size];
		}
	}

	var x=new Collection();
	console.log(Collection.sizeOf(x));//0

	x.add('foo');
	console.log(Collection.sizeOf(x));//1

	console.log(Object.keys(x));//['0']
	console.log(Object.getOwnPropertyNames(x));//['0']
	console.log(Object.getOwnPropertySymbols(x));//[Symbol(size)]

>对象x的size属性是一个Symbol值，`Object.keys(x)`、`Object.getOwnPropertyNames(x)`都无法获取它。

##5、Symbol.for(),Symbol.keyFor()
>重新使用同一个Symbol值，`Symbol.for`方法可以做到。它接受一个字符串作为参数，然后搜索有没有参数作为名称的Symbol值。如果有，就返回这个Sybmol值，否则就新建并返回一个以字符串为名称的Symbol值。

	var s1=Symbol.for('foo');
	var s2=Symbol.for('foo');

	console.log(s1===s2);//true

>`Symbol.for()`与`Symbol()`这两种写法，都会生成新的Symbol。它们的区别是，前者会被登记在全局环境中供搜索，后者不会。`Symbol.for()`不会每次调用就返回一个新的Symbol类型的值，而是会先检查给定的key是否已经存在，如果不存在才会新建一个值。比如，如果你调用`Symbol.for('cat')`30次，每次都会返回同一个Symbol值，但是调用`Symbol('cat')`30次，会返回30个不同的Symbol值。

	console.log(Symbol.for('bar')===Symbol.for('bar'));//true

	console.log(Symbol('bar')===Symbol('bar'));//false

>Symbol.keyFor方法返回一个已登记的Symbol类型值的key。

	var s1=Symbol.for('foo');
	console.log(Symbol.keyFor(s1));//'foo'

	var s2=Symbol('foo');
	console.log(Symbol.keyFor(s2));//undefined

>`Symbol.for`为Symbol值登记的名字，是全局环境的，可以在不同的iframe或service worker中取到同一个值。

	var iframe = document.createElement('iframe');
	iframe.src = String(window.location);
	document.body.appendChild(iframe);
	
	console.log(iframe.contentWindow.Symbol.for('foo') === Symbol.for('foo'));// true

##6、内置的Symbol值
>除了定义自己使用的Symbol值以外，ES6还提供了11个内置的Symbol值，指向语言内部使用的方法。

__Symbol.hasInstance__
>对象的`Symbol.hasInstance`属性，指向一个内部方法。该对象使用`instanceof`运算符时，会调用这个方法，判断该对象是否为某个构造函数的实例。`for instanceof Foo`在语言内部，实际调用的是`Foo[Symbol.hasInstance](foo)`。

	class MyClass {
		[Symbol.hasInstance](foo) {
			return foo instanceof Array;
		}
	}

	var o=new MyClass();
	console.log(o instanceof Array);//false

__Symbol.isConcatSpreadable__
>对象的`Symbol.isConcatSpreadable`属性等于一个布尔值，表示该对象使用`Array.prototype.concat()`时，是否可以展开。

	let arr1=['c','d'];
	var arr=['a','b'].concat(arr1,'e');
	console.log(arr);//['a', 'b', 'c', 'd', 'e']

	let arr2=['c','d'];
	arr2[Symbol.isConcatSpreadable]=false;
	var arr=['a','b'].concat(arr2,'e');
	console.log(arr);//['a','b',['c','d'],'e']

>`Symbol.isConcatSpreadable`属性必须写成一个返回布尔值的方法。

	class A1 extend Array {
		[Symbol.isConcatSpreadable]() {
			return true;
		}
	}

	class A2 extend Array {
		[Symbol.isConcatSpreadable]() {
			return false;
		}
	}

	let a1=new A1();
	a1[0]=3;
	a1[1]=4;

	let a2=new A2();
	a2[0]=5;
	a2[1]=6;
	console.log([1,2].concat(a1).concat(a2));//[1,2,[3,4],[5,6]]
	//上面代码中，类`A1`是可扩展的，类`A2`是不可扩展的，所以使用`concat`时有不一样的结果

__Symbol.species__
>对象的`Symbol.species`属性，指向一个方法。该对象作为构造函数创造实例时，会调用这个方法。即如果`this.constructor[Symbol.species]`存在，就会使用这个属性作为构造函数，来创造新的实例对象。

>`Symbol.species`属性默认的读取器如下：

	static get [Symbol.species]() {
		return this;
	}

__Symbol.match__
>对象的`Symbol.match`属性，指向一个函数。当执行`str.match(myObject)`时，如果该属性存在，会调用它，返回该方法的返回值。

	String.prototype.match(regexp);
	//等同于
	regexp[Symbol.match](this);

	class MyMatcher {
		[Symbol.match](string) {
			return 'hello world'.indexOf(string)
		}
	}

	console.log('e'.match(new MyMatcher()));//1

__Symbol.replace__
>对象的`Symbol.replace`属性，指向一个方法，当该对象被`String.prototype.replace`方法调用时，会返回该方法的返回值。

	String.prototype.replace(searchValue,replaceValue)
	//等同于
	searchValue[Symbol.replace](this,replaceValue)

__Symbol.search__
>对象的`Symbol.search`属性，指向一个方法，当该对象被`String.prototype.search`方法调用时，会返回该方法的返回值。

	
	String.prototype.search(regexp)
	//等同于
	regexp[Symbol.search](this)

	class MySearch {
		constructor(value) {
			this.value=value;
		}

		[Symbol.search](string) {
			return string.indexOf(this.value);
		}
	}

	console.log('foobar'.search(new MySearch('foo')));//0

__Symbol.split__
>对象的`Symbol.split`属性，指向一个方法，当对象被`String.prototype.split`方法调用时，会返回该方法的返回值。

	String.prototype.split(seperator,limit)
	//等同于
	sepeartor[Symbol.split](this,limit)

__Symbol.iterator__
>对象的Symbol.iterator属性，指向该对象的默认遍历器方法，即该对象进行for...of循环时，会调用这个方法，返回该对象的默认遍历器。

	class Collection {
	  *[Symbol.iterator]() {
	    let i = 0;
	    while(this[i] !== undefined) {
	      yield this[i];
	      ++i;
	    }
	  }
	}
	
	let myCollection = new Collection();
	myCollection[0] = 1;
	myCollection[1] = 2;
	
	for(let value of myCollection) {
	  console.log(value);
	}
	// 1
	// 2

__Symbol.toPrimitive__
>对象的`Symbol.toPrimitive`属性，指向一个方法。该对象被转为原始类型的值时，会调用这个方法，返回该对象对应的原始类型值。

>`Symbol.toPrimitive`被调用时，会接受一个字符串参数，表示当前运算符的模式，一共有三种模式。

	-Number:该场合需要转成数值
	-String:该场合需要转成字符串
	-Default:该场合可以转成数值，也可以转成字符串


	let obj = {
	  [Symbol.toPrimitive](hint) {
	    switch (hint) {
	      case 'number':
	        return 123;
	      case 'string':
	        return 'str';
	      case 'default':
	        return 'default';
	      default:
	        throw new Error();
	     }
	   }
	};

	console.log(2 * obj);	// 246
	console.log(3 + obj); // '3default'
	console.log(obj === 'default'); // true
	console.log(String(obj)); // 'str'

__Symbol.toStringTag__
>对象的`Symbol.toStringTag`属性，指向一个方法。在该对象上面调用`Object.prototype.toString`方法时，如果这个属性存在，它的返回值会出现在`toString`方法返回的字符串之中，表示对象的类型。也就是说，这个属性可以用来定制`[object Object]`或`[object Array]`中object后面的那个字符串。

	console.log({[Symbol.toStringTag]:'Foo'}.toString());//"[object Foo]"

	class Collection {
		get [Symbol.toStringTag]() {
			return 'xxx';
		}
	}

	var x=new Collection();
	console.log(Object.prototype.toString.call(x));//"[object xxx]"

>ES6新增内置对象的`Symbol.toStringTag`属性值如下：

	-`JSON[Symbol.toStringTag]`:'JSON'
	-`Math[Symbol.toStringTag]`:'Math'
	-Module对象`M[Symbol.toStringTag]`:'Module'
	-`ArrayBuffer.prototype[Symbol.toStringTag]`：'ArrayBuffer'
	-`DataView.prototype[Symbol.toStringTag]`：'DataView'
	-`Map.prototype[Symbol.toStringTag]`：'Map'
	-`Promise.prototype[Symbol.toStringTag]`：'Promise'
	-`Set.prototype[Symbol.toStringTag]`：'Set'
	-`%TypedArray%.prototype[Symbol.toStringTag]`：'Uint8Array'等
	-`WeakMap.prototype[Symbol.toStringTag]`：'WeakMap'
	-`WeakSet.prototype[Symbol.toStringTag]`：'WeakSet'
	-`%MapIteratorPrototype%[Symbol.toStringTag]`：'Map Iterator'
	-`%SetIteratorPrototype%[Symbol.toStringTag]`：'Set Iterator'
	-`%StringIteratorPrototype%[Symbol.toStringTag]`：'String Iterator'
	-`Symbol.prototype[Symbol.toStringTag]`：'Symbol'
	-`Generator.prototype[Symbol.toStringTag]`：'Generator'
	-`GeneratorFunction.prototype[Symbol.toStringTag]`：'GeneratorFunction'

__Symbol.unscopables__
>对象的`Symbol.unscopables`属性，指向一个对象。该对象指定了使用`with`关键字时，哪些属性会被`with`环境排除。

	Array.prototype[Symbol.unscopables]
	// {
	//   copyWithin: true,
	//   entries: true,
	//   fill: true,
	//   find: true,
	//   findIndex: true,
	//   keys: true
	// }

	console.log(Object.keys(Array.prototype[Symbol.unscopables]));// ['copyWithin', 'entries', 'fill', 'find', 'findIndex', 'keys']

>数组有6个属性，会被with命令排除。

	//没有unscopables时
	class MyClass {
		foo() { return 1; }
	}

	var foo=function () { return 2; }

	with (MyClass.prototype) {
		foo();//1
	};	

	//有unscopables时
	class MyClass {
		foo() { return 1; }
		get [Symbol.unscopables]() {
			return { foo:true };
		}
	}

	var foo=function () { return 2; }

	with (MyClass.prototype) {
		foo();//2
	}








