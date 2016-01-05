#数值的扩展
#
##1、二进制和八进制表示法
>ES6提供了二进制和八进制数值的新的写法，分别用前缀`0b`(或`0B`)和`0o`(或`0O`)表示。

	console.log(0b111110111===503);//true
	console.log(0o767===503);//true

>从ES5开始，在严格模式之中，八进制就不再允许使用前缀`0`表示，ES6进一步明确，要使用前缀`0o`表示。

	// 非严格模式
	(function(){
	  console.log(0o11 === 011);
	})() // true

	// 严格模式
	(function(){
	  'use strict';
	  console.log(0o11 === 011);
	})() // Uncaught SyntaxError: Octal literals are not allowed in strict mode.

>如果要将`0b`和`0x`前缀的字符串数值转为十进制，要使用`Number`方法。

	console.log(Number('0b111'));//7
	console.log(Number('0o10'));//8

##2、Number.isFinite(),Number.isNaN()
>ES6在Number对象上，新提供了`Number.isFinite()`和`Number.isNaN()`两个方法，用来检查`Infinite`和`NaN`这两个特殊值。

>`Number.isFinite()`用来检查一个数值是否非无穷。

	console.log(Number.isFinite(15));//true
	console.log(Number.isFinite(0.8));//true
	console.log(Number.isFinite(NaN));//false
	console.log(Number.isFinite(Infinity));//false
	console.log(Number.isFinite(-Infinity));//false
	console.log(Number.isFinite('foo'));//false
	console.log(Number.isFinite('15'));//false
	console.log(Number.isFinite(true));//false

>ES5可以通过下面的代码来部署`Number.isFinite`方法。

	(function(global){
		var global_isFinite=global.isFinite;

		Object.defineProperty(Number,'isFinite',{
			value: function isFinite(value) {
				return typeof value==='number' && global_isFinite(value);
			},
			configurable: true,
			enumerable: false,
			writable: true
		});
	})(this);

>`Number.isNaN()`用来检查一个值是否为NaN。

	console.log(Number.isNaN(NaN));//true
	console.log(Number.isNaN(15));//false
	console.log(Number.isNaN('15'));//false
	console.log(Number.isNaN(true));//false
	console.log(Number.isNaN(9/NaN));//true
	console.log(Number.isNaN('true'/0));//true
	console.log(Number.isNaN('true'/'true'));//true

>ES5通过上面的代码，部署`Number.isNaN()`。

	(function (global) {
		var global_isNaN=global.isNaN;

		Object.defineProperty(Number,'isNaN',{
			value: function isNaN(value) {
				return typeof value === 'number' && global_isNaN(value);
			},
			configurable: true,
			enumerable: false,
			writable: true
		});
	})(this);
>它们与传统的全局方法`isFinite()`和`isNaN()`的区别在于，传统的方法先调用`Number()`将非数值的值转为数值，再进行判断，而这两个新方法只对数值有效，非数值一律返回`false`。

	console.log(isFinite(25));//true
	console.log(isFinite("25"));//true
	console.log(Number.isFinite(25));//true
	console.log(Number.isFinite("25"));//false

	console.log(isNaN(NaN));//true
	console.log(isNaN("NaN"));//true
	console.log(Number.isNaN(NaN));//true
	console.log(Number.isNaN("NaN"));//false

##3、Number.parseInt(),Number.parseFloat()
>ES6将全局方法`parseInt()`和`parseFloat()`，移植到Number对象上面，行为完全保持不变。
	
	console.log(parseInt('12.34'));//12
	console.log(parseFloat('123.45#'));//123.45


	console.log(Number.parseInt('12.34'));//12
	console.log(Number.parseFloat('123.45#'));//123.45

>逐步减少全局性方法，使得语言逐步模块化。

	console.log(Number.parseInt === parseInt);//true
	console.log(Number.parseFloat === parseFloat);//true

##4、Number.isInteger()
`Number.isInteger()`用来判断一个值是否为整数。需要注意的是，在JavaScript内部，整数和浮点数是同样的储存方法，所以3和3.0被视为同一个值。

	console.log(Number.isInteger(25));//true
	console.log(Number.isInteger(25.0));//true
	console.log(Number.isInteger(25.1));//false
	console.log(Number.isInteger("15"));//false
	console.log(Number.isInteger(true));//false
>ES5可以通过下面的代码，部署Number.isInteger()。

	(function (global) {
		var floor=Math.floor,
		isFinite=global_isFinite;

		Object.defineProperty(Number,'isInteger',{
			value: function isInteger(value) {
				return typeof value === 'number' && isFinite(value) &&
				value > -9007199254740992 && value < 9007199254740992 && floor(value) === value;
			},
			configurable: true,
			enumerable: false,
			writable: true
		});
	})(this);

##5、Number.EPSILON
>ES6在Number对象上面，新增一个极小的常量`Number.EPSILON`。
	console.log(Number.EPSILON);//2.220446049250313e-16
	console.log(Number.EPSILON.toFixed(20));//0.00000000000000022204
>`Number.EPSILON`的实质是一个可以接受的误差范围。

	function withinErrorMargin (left,right) {
		return Math.abs(left-right) < Number.EPSILON
	}

	console.log(withinErrorMargin(0.1+0.2,0.3));//true
	console.log(withinErrorMargin(0.2+0.2,0.3));//false

##6、安全整数和Number.isSafeInteger()
>JavaScript能够准备表示的整数范围在`-2^53`到`2^53`之间(不含两个端点),超过这个范围，无法精确表示这个值。

	console.log(Math.pow(2,53)===Math.pow(2,53)+1);
>ES6引入了`Number.MAX_SAFE_INTEGER`和`Number.MIN_SAFE_INTEGER`这两个常量，用来表示这个范围的上下限。

	console.log(Number.MAX_SAFE_INTEGER === Math.pow(2,53)-1);
	console.log(Number.MAX_SAFE_INTEGER === 9007199254740991);

	console.log(Number.MIN_SAFE_INTEGER === -Number.MAX_SAFE_INTEGER);
	console.log(Number.MIN_SAFE_INTEGER === -9007199254740991);
>`Number.isSafeInteger()`则是用来判断一个整数是否落在这个范围之内。

	console.log(Number.isSafeInteger('a'));//false
	console.log(Number.isSafeInteger(null));//false
	console.log(Number.isSafeInteger(NaN));//false
	console.log(Number.isSafeInteger(Infinity));//false
	console.log(Number.isSafeInteger(-Infinity));//false

	console.log(Number.isSafeInteger(3));//true
	console.log(Number.isSafeInteger(1.2));//false
	console.log(Number.isSafeInteger(9007199254740990));//true
	console.log(Number.isSafeInteger(9007199254740992));//false

	console.log(Number.isSafeInteger(Number.MIN_SAFE_INTEGER - 1));//false
	console.log(Number.isSafeInteger(Number.MIN_SAFE_INTEGER));//true
	console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER));//true
	console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 1));//false

>验证运算结果是否落在安全整数的范围时，不要只验证运算结果，而要同时验证参与运算的每个值。

	console.log(9007199254740993 === 9007199254740992);//true
>9007199254740993不是一个安全整数，但是Number.isSafeInteger会返回结果，显示计算结果是安全的。这是因为，这个数超出了精度范围，导致在计算机内部，以9007199254740992的形式储存。
>如果只验证运算结果是否为安全整数，很可能得到错误结果。下面的函数可以同时验证两个运算数和运算结果。

	function trusty (left,right,result) {
		if(Number.isSafeInteger(left) && Number.isSafeInteger(right) && Number.isSafeInteger(result))
		{
			return result;
		}

		throw new RangeError('Option cannot be trusted!');
	}

	console.log(trusty(9007199254740993, 990, 9007199254740993 - 990));// RangeError: Operation cannot be trusted!

	console.log(trusty(1,2,3));//3

##7、Math对象的扩展 
>ES6在Math对象上新增加了17个与数学相关的方法，所有这些方法都是静态方法，只能在Math对象上调用。

###Math.trunc()
>`Math.trunc`方法用于去除一个数的小数部分，返回整数部分。

	console.log(Math.trunc(4.1));//4
	console.log(Math.trunc(4.9));//4
	console.log(Math.trunc(-4.1));//-4
	console.log(Math.trunc(-4.9));//-4
	console.log(Math.trunc(-0.1234));//-0

>对于非数值，`Math.trunc`内部使用`Number`方法将其先转为数值。

	console.log(Math.trunc('123.456'));//123
>对于空值和无法截取整数的值，返回NaN。

	console.log(Math.trunc(NaN));
	console.log(Math.trunc('foo'));
	console.log(Math.trunc());//NaN
>对于没有部署这个方法的环境，可以用下面的代码模拟。

	Math.trunc=Math.trunc || function(x) {
		return x < 0 ? Math.ceil(x) : Math.floor(x);
	}

###Math.sign()
>`Math.sign`方法用来判断一个数到底是正数、负数、还是零。
>它会返回五种值。
>
	-参数为正常，返回+1;
	-参数为负数，返回-1;
	-参数为0，返回0;
	-参数为-0，返回-0;
	-其他值，返回NaN。

	console.log(Math.sign(-5));//-1
	console.log(Math.sign(5));//+1
	console.log(Math.sign(0));//+0
	console.log(Math.sign(-0));//-0
	console.log(Math.sign(NaN));//NaN
	console.log(Math.sign('foo'));//NaN
	console.log(Math.sign());//NaN
>同有部署这个方法的环境，可以用下面的代码模拟。

	Math.sign=Math.sign || function(x) {
		x=+x;
		if(x===0 || isNaN(x)){
			return x;
		}

		return x >0 ? 1 :-1;
	}

###Math.cbrt()
>`Math.cbrt`方法用来计算一个数的立方根。
	console.log(Math.cbrt(-1));//-1
	console.log(Math.cbrt(0));//0
	console.log(Math.cbrt(1));//1
	console.log(Math.cbrt(2));//1.259921049894873
>对于非数值，`Math.cbrt`方法内部也是先使用`Number`方法将其转换为数值。

	console.log(Math.cbrt('8'));//2
	console.log(Math.cbrt('hello'));//NaN
>没有部署这个方法的环境，可以用下面的代码模拟。

	Math.cbrt=Math.cbrt || function(x) {
		var y=Math.pow(Math.abs(x),1/3);
		return x<0 ? -y : y;
	};
###Math.clz32()
>JavaScript的整数使用32位二进制形式表示，`Math.clz32`方法返回一个数的32位无符号整数形式有多少个前导0.

	console.log(Math.clz32(0));//32
	console.log(Math.clz32(1));//31
	console.log(Math.clz32(1000));//22
	console.log(Math.clz32(0b01000000000000000000000000000000));//1
	console.log(Math.clz32(0b00100000000000000000000000000000));//2

>左移运算符(`<<<`)与`Math.clz32`方法直接相关。

	console.log(Math.clz32(0));//32
	console.log(Math.clz32(1));//31
	console.log(Math.clz32(1<<1));//30
	console.log(Math.clz32(1<<2));//29
	console.log(Math.clz32(1<<29));//2
>对于小数，`Math.clz32`方法只考虑整数部分。

	console.log(Math.clz32(3.2));//30
	console.log(Math.clz32(3.9));//30
>对于空值或其他类型的值，`Math.clz32`方法会将它们转为数值，然后再计算。

	console.log(Math.clz32());//32
	console.log(Math.clz32(NaN));//32
	console.log(Math.clz32(Infinity));//32
	console.log(Math.clz32(null));//32
	console.log(Math.clz32('foo'));//32
	console.log(Math.clz32([]));//32
	console.log(Math.clz32({}));//32
	console.log(Math.clz32(true));//31
###Math.imul()
>`Math.imul`方法返回两个数以32位带符号整数形式相乘的结果，返回的也是一个32位的带符号整数。

	console.log(Math.imul(2,4));//8
	console.log(Math.imul(-1,8));//-8
	console.log(Math.imul(-2,-2));//4
>如果只考虑最后32位，大多数情况下，`Math.imul(a,b)`与`a * b`的结果是相同的，即该方法等同于`(a*b)|0`的效果(超过32位的部分溢出)。之所以需要部署这个方法，是因为JavaScript有精度限制，超过2的53次方的值无法精确表示。这就是说，对于那些很大的数的乘法，低位数值往往都是不精确的，`Math.imul`方法可以返回正确的低位数值。

	console.log((0x7fffffff * 0x7fffffff)|0);//0
>由于这两个二进制数的最低位都是1，所以这个结果肯定是不正确的，因为根据二进制乘法，计算结果的二进制最低位应该也是1。这个错误就是因为它们的乘积超过了2的53次方，JavaScript无法保存额外的精度，就把低位的值都变成了0。`Math.imul`方法可以返回正确的值1。

	console.log(Math.imul(0x7fffffff, 0x7fffffff));//1
###Math.fround()
>Math.fround方法返回一个数的单精度浮点数形式。

	console.log(Math.fround(0));	//0
	console.log(Math.fround(1));     // 1
	console.log(Math.fround(1.337)); // 1.3370000123977661
	console.log(Math.fround(1.5));   // 1.5
	console.log(Math.fround(NaN));   // NaN
	
>对于整数来说，`Math.fround`方法返回结果不会有任何不同，区别主要是那些无法使用64个二进制位精确表示的小数。这时，`Math.fround`方法会返回最近这个小数的单精度浮点数。

	Math.fround=Math.fround || function(x) {
		return new Float32Array([x])[0];
	};

###Math.hypot()
>`Math.hypot`方法返回所有参数的平方和的平方根。

	console.log(Math.hypot(3,4));//5
	console.log(Math.hypot(3,4,5));//7.0710678118654755
	console.log(Math.hypot());//0
	console.log(Math.hypot(NaN));//NaN
	console.log(Math.hypot(3,4,'foo'));//NaN
	console.log(Math.hypot(3,4,'5'));//7.0710678118654755
	console.log(Math.hypot(-3));//3
>如果参数不是数值，`Math.hypot`方法会将其转为数值。只要有一个参数无法转为数值，就会返回NaN。

####对数方法
>ES6新增加了4个对数相关方法。

_(1)Math.expml()_
`Math.expm1(x)`返回ex-1,即`Math.exp(x)-1`。

	console.log(Math.expm1(-1));//-0.6321205588285577
	console.log(Math.expm1(0));//0
	console.log(Math.expm1(1));//1.718281828459045
>没有部署这个方法的环境，可以用下面的代码模拟。

	Math.expml=Math.expml||function(x) {
		return Math.exp(x)-1;
	};

_(2)Math.log1p()_
>`Math.log1p(x)`方法返回`1+x`的自然对数，即`Math.log(1+x)`。如果`x`小于-1，返回`NaN`。

	console.log(Math.log1p(1));//0.6931471805599453
	console.log(Math.log1p(0));//0
	console.log(Math.log1p(-1));//-Infinity
	console.log(Math.log1p(-2));//NaN
>对于没有部署这个方法的环境，可以用下面的代码模拟。

	Math.log1p=Math.log1p||function(x) {
		return Math.log(1+x);
	};

_(3) Math.log10()_
>`Math.log10(x)`返回以10为底的x的对数。如果x小于0，则返回NaN。

	console.log(Math.log10(2));//0.3010299956639812
	console.log(Math.log10(1));//0
	console.log(Math.log10(0));//-Infinity
	console.log(Math.log10(-2));//NaN
	console.log(Math.log10(100000));//5
>如果没有部署这个方法的环境，可以用下面的代码模拟。

	Math.log10=Math.log10 || function(x) {
		return Math.log(x)/Math.LN10;
	};

_(4)Math.log2()_
>`Math.log2(x)`返回以2为底的x的对数。如果x小于0，则返回NaN。

	console.log(Math.log2(3));//1.584962500721156
	console.log(Math.log2(2));//1
	console.log(Math.log2(1));//0
	console.log(Math.log2(0));//-Infinity
	console.log(Math.log2(-2));//NaN
	console.log(Math.log2(1024));//10
	console.log(Math.log2(1<<29));//29
>对于没有部署这个方法的环境，可以用下面的代码模拟。

	Math.log2=Math.log2 || function(x) {
		return Math.log(x)/Math.LN2;
	};

####三角函数方法
ES6新增了6个三角函数方法。
	
	Math.sinh(x) 返回x的双曲正弦（hyperbolic sine）
	Math.cosh(x) 返回x的双曲余弦（hyperbolic cosine）
	Math.tanh(x) 返回x的双曲正切（hyperbolic tangent）
	Math.asinh(x) 返回x的反双曲正弦（inverse hyperbolic sine）
	Math.acosh(x) 返回x的反双曲余弦（inverse hyperbolic cosine）
	Math.atanh(x) 返回x的反双曲正切（inverse hyperbolic tangent）

##8、指数运算符
>ES7新增了一个指数运算符（`**`），目前Babel转码器已经支持。

	console.log(2 ** 2);//4
	console.log(2 ** 3);//8
>指数运算符可以与等号结合，形成一个新的赋值运算符(`**=`)。

	let a=2;
	a **=2;//等同于a=a*a;

	let b=3;
	b **=3;//等同于b=b*b*b;


	