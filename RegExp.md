#正则的扩展 
#
##1、RegExp构造函数
>在ES5中，RegExp构造函数只能接受字符串作为参数。

	var regex=new RegExp("xyz","i");

	//等价于
	var regex=/xyz/i;

>ES6允许RegExp构造函数接受正则表达式作为参数，这是会返回一个原有正则表达式的拷贝。

	var regex = new RegExp(/xyz/i);
>如果使用RegExp构造函数的第二个参数指定修饰符，则返回的正则表达式会忽略原有的正则表达式的修饰符，只使用新指定的修饰符。

	var regflag=new RegExp(/abc/ig,"i").flags;
	console.log(regflag);//i

##2、字符串的正则方法
>字符串对象共有4个方法，可以使用正则表达式：match()、replace()、search()和split()。

>ES6将这4个方法，在语言内部全部调用RegExp的实例方法，从而做到所有与正则相关的方法，全都定义在RegExp对象上。

-`String.prototype.match`调用`RegExp.prototype[Symbol.match]`
-`String.prototype.replace`调用`RegExp.prototype[Symbol.replace]`
-`String.prototype.search`调用`RegExp.prototype[Symbol.search]`
-`String.prototype.split`调用`RegExp.prototype[Symbol.split]`

##3、u修饰符
>ES6对正则表达式添加了`u`修饰符，含义为"Unicode模式",用来正解处理大于`\uFFFF`的Unicode字符。也就是说，会正确处理四个字节的UTF-16编码。

	console.log(/^\uD83D/u.test('\uD83D\uDC2A'));
	// false
	console.log(/^\uD83D/.test('\uD83D\uDC2A'));
	// true
>"\uD83D\uDC2A"是一个四个字节的UTF-16编码，代表一个字符。但是，ES5不支持四个字节的UTF-16编码，会将其识别为两个字符，导致第二行代码结果为true。加了u修饰符以后，ES6就会识别其为一个字符，所以第一行代码结果为`false`。

>一旦加上u修饰符号，就会修改下面这些正则表达式的行为。

_(1)点字符_
>点(.)字符在正则表达式中，含义是除了换行符以外的任意单个字符。对于码点大于`oxFFFF`的Unicode字符，点字符不能识别，必须加上u修饰符。

	var s = "𠮷";

	console.log(/^.$/.test(s)); // false
	console.log(/^.$/u.test(s)); // true
_(2)Unicode字符表示法_
>ES6新增了使用大括号表壳Unicode字符，这种表示法在正则表达式中必须加上u修饰符，才能识别。

	console.log(/\u{61}/.test('a'));//false
	console.log(/\u{61}/u.test('a'));//true
	console.log(\u{20BB7}/u.test('𠮷'));//true
>上面代码表示，如果不加u修饰符，正则表达式无法识别`\u{61}`这种表示法，只会认为这匹配61个连续的u。

_(3)量词_
>使用u修饰符后，所有量词都会正确识别大于码点大于`0xFFFF`的Unicode字符。
	
	console.log(/a{2}/.test('aa')); //true
	console.log(/a{2}/u.test('aa')); //true
	console.log(/𠮷{2}/.test('𠮷𠮷')); // false
	console.log(/𠮷{2}/u.test('𠮷𠮷')); // true
>另外，只有在使用u修饰符的情况下，Unicode表达式中的大括号才会被正确解读，否则会被解读为量词。

	console.log(/^\u{3}$/.test('uuu'));//true

_(4)预定义模式_
>u修饰符也影响到预定义模式，能否正确识别码点大于`0xFFFF`的Unicode字符。

	console.log(/^\S$/.test('𠮷'));//false
	console.log(/^\S$/u.test('𠮷'));//true
>`\S`是预定义模式，匹配所有不是空格的字符。只有加了u修饰符，它才能正确匹配码点大于0xFFFF的Unicode字符。
	
_正解返回字符串长度的函数:_
	
	function codePointLength(text) {
		var result=text.match(/[\s\S]/gu);
		return result ? result.length : 0;
	}

	var s="𠮷𠮷";
	console.log(s.length);//4
	console.log(codePointLength(s));//2

_(5)i修饰符_
>有些Unicode字符的编码不同，但是字型很相近，如，\u004B与\u212A都是大写的K。

	//不加u修饰符，就无法识别非规范的K字符。
	console.log(/[a-z]/i.test('\u212A')); // false
	console.log(/[a-z]/iu.test('\u212A')); // true	

##4、y修饰符
>除了`u`修饰符，ES6还为正则表达式添加了`y`修饰符，叫做"粘连"(sticky)修饰符。

>y修饰符的作用与g修饰符类似，也是全局匹配，后一次匹配都从上一次匹配成功的下一个位置开始。不同之处在于，g修饰符只要剩余位置中存在匹配就可，而y修饰符确保匹配必须从剩余的第一个位置开始，这也就是"粘连"的涵义。

	var s="aaa_aa_a";
	var r1=/a+/g;
	var r2=/a+/y;

	console.log(r1.exec(s));
	console.log(r2.exec(s));

	console.log(r1.exec(s));
	console.log(r2.exec(s));

>如果改一下正则表达式，保证每次都能头部匹配，y修饰符就会返回结果。

	var s="aaa_aa_a";
	var r=/a+_/y;

	console.log(r.exec(s));//["aaa_"]
	console.log(r.exec(s));//["aa_"]

>使用`lastIndex`属性，可以更好地说明`y`修饰符。

	const REGEX = /a/g;

	// 指定从2号位置（y）开始匹配
	REGEX.lastIndex = 2;

	// 匹配成功
	const match = REGEX.exec('xaya');

	// 在3号位置匹配成功
	console.log(match.index); // 3

	// 下一次匹配从4号位开始
	console.log(REGEX.lastIndex); // 4

	// 4号位开始匹配失败
	console.log(REGEX.exec('xaxa')); // null
>y修饰符同样遵守`lastIndex`属性，但是要求必须在`lastIndex`指定的位置发现匹配。

	const REGEX=/a/y;

	// 指定从2号位置（y）开始匹配
	REGEX.lastIndex = 2;

	// 不是粘连，匹配失败
	console.log(REGEX.exec('xaya')); // null

	//指定从3号位置开始匹配
	REGEX.lastIndex=3;

	// 匹配成功
	const match = REGEX.exec('xaya');

	// 在3号位置匹配成功
	console.log(match.index); // 3

	// 下一次匹配从4号位开始
	console.log(REGEX.lastIndex); // 4
>`y`修饰符号隐含了头部匹配的标志^。y修饰符的设计本意，就是让头部匹配的标志ˆ在全局匹配中都有效。

	console.log(/b/y.exec("aba"));//null
>在split方法中使用y修饰符，原字符串必须以分隔符开头。这也意味着，只要匹配成功，数组的第一个成员肯定是空字符串。

	//没有找到匹配
	console.log('x##'.split(/#/y));//[ 'x##' ];
	
	//找到两个匹配
	console.log('##x'.split(/#/y));//['','','x']
>后续的分隔符只有紧跟前面的分隔符，才会被识别。

	console.log('#x#'.split(/#/y));//['','x#']

	console.log('##'.split(/#/y));//['','','']
>字符串对象的replace方法。

	const REGEX=/a/gy;
	console.log('aaxa'.replace(REGEX,'-'));//'--xa'
>`y`修饰符的一个应用， 是从字符中提取token(词元),`y`修饰符确保了匹配之间不会有漏掉的字符。

	const TOKEN_Y=/\s*(\+|[0-9]+)\s/y;
	const TOKEN_G=/\s*(\+|[0-9]+)\s*/g;

	console.log(tokenize(TOKEN_Y,'3+4'));//["3","+","4"]
	console.log(tokenize(TOKEN_G,'3+4'));//["3","+","4"]

	console.log(tokenize(TOKEN_Y,'3x+4'));//["3"]
	console.log(tokenize(TOKEN_G,'3x+4'));//["3","+","4"]

	function tokenize(TOKEN_REGEX,str) {
		let result=[];
		let match;
		while(match=TOKEN_REGEX.exec(str)) {
			result.push(match[1]);
		}
		return result;
	}

##5、sticky属性

>与y修饰符相匹配，ES6的正则对象多了sticky属性，表示是否设置了y修饰符。

	var r=/hello\d/y;
	console.log(r.sticky);//true
##6、flags属性
>ES6为正则表达式新增了flags属性，会返回正则表达式的修饰符。

	//ES5的source属性
	//返回正则表达式的正文
	console.log(/abc/ig.source);//"abc"

	//ES6的flags属性
	//返回正则表达式的修饰符
	console.log(/abc/ig.flags);//'gi'

##7、RegExp.escape()
>字符串必须转义，才能作为正则表达式。

	function escapeRegExp(str) {
		return str.replace(/[\-\[\]\/\{\}\(\)\*\+\?\.\\\^\$\|]/g, "\\$&");
	}
	
	let str='/path/to/resource.html?search=query';
	console.log(escapeRegExp(str));// "\/path\/to\/resource\.html\?search=query"


	

	


	

