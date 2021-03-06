#字符串扩展
#
##1、字符串的Unicode表示法
>`JavaScript`允许采用`\uxxxx`形式表示一个字符，其中"xxxx"表示字符的码点。

	var vname="\u0061"
	console.log(vname);//"a"
>这种表示法只限于`\u0000`——`\uFFFF`之间的字符。超出这个范围的字符，必须用两个双字节的形式表达。

	var vname="\uD842\uDFB7"
	console.log(vname);// "₻"

	var v="\u20BB7"
	console.log(v);// " 7"

>如果直接在“\u”后面跟上超过0xFFFF的数值（比如\u20BB7），JavaScript会理解成“\u20BB+7”。由于\u20BB是一个不可打印字符，所以只会显示一个空格，后面跟着一个7。
>
>ES6对这一点做出了改进，只要将码点放入大括号，就能正确解读该字符。

	var a="\u{20BB7}"
	console.log(a);// "₻"
	
	var b="\u{41}\u{42}\u{43}"
	console.log(b);// "ABC"
	
	let hello = 123;
	console.log(hell\u{6F}); // 123
	
	console.log('\u{1F680}' === '\uD83D\uDE80');// true

>因此，JavaScript共有6种方法可以表示一个字符。

	'\z' === 'z'  // true
	'\172' === 'z' // true
	'\x7A' === 'z' // true
	'\u007A' === 'z' // true
	'\u{7A}' === 'z' // true

##2、codePointAt()
>JavaScript内部，字符以UTF-16的格式储存，每个字符固定为2个字节。对于那些需要4个字节储存的字符（Unicode码点大于0xFFFF的字符），JavaScript会认为它们是两个字符。

	var s = "𠮷";
	
	console.log(s.length); // 2
	console.log(s.charAt(0)); // ''
	console.log(s.charAt(1)); // ''
	console.log(s.charCodeAt(0)); // 55362
	console.log(s.charCodeAt(1)) // 57271

>ES6提供了`codePointAt`方法，能够正确处理4个字节储存的字符，返回一个字符的码点。
	
	var s=`₻a`;
	
	console.log(s.codePointAt(0));
	console.log(s.codePointAt(1));
	
	console.log(s.charCodeAt(2));

>`codePointAt`方法返回的是码点的十进制值，如果想要十六进制的值，可以使用`toString`方法转换一下。
	
	var s=`₻a`;
	
	console.log(s.codePointAt(0).toString(16));
	console.log(s.charCodeAt(2).toString(16));

>`codePointAt`方法是测试一个字符由两个字节还是由四个字节组成的最简单的方法。

	function is32Bit(c) {
		return c.codePointAt(0)>0xFFFF;	
	}

	console.log(is32Bit("𠮷"));//true
	console.log(is32Bit("a"));//false
	
##3、String.fromCodePoint()
>ES5提供的`String.fromCharCode`方法，用于从码点返回对应字符，但是这个方法不能识别32位的UTF-16字符(Unicode编号大于`0xFFFF`)。

	console.log(String.fromCharCode(0x20BB7));//ஷ
	
	String.fromCharCode不能识别大于0xFFFF的码点，所以0x20BB7就发生了溢出，最高位2被舍弃了，最后返回码点U+0BB7对应的字符，而不是码点U+20BB7对应的字符。
>ES6提供了`String.fromCodePoint`方法，可以识别`0xFFFF`的字符，弥补了`String.fromCharCode`方法的不足。在作用上，正好与`codePointAt`方法相反。

	console.log(String.fromCodePoint(0x20BB7));
	// "𠮷"
	console.log(String.fromCodePoint(0x78, 0x1f680, 0x79) === 'x\uD83D\uDE80y');
	// true

>`fromCodePoint`方法定义在`String`对象上，而`codePointAt`方法定义在字符串的实例对象上。

##4、字符串的遍历器接口
>ES6为字符串添加了遍历器接口，使得字符串可以被`for...of`循环遍历。

	for (let codePoint of `foo`)
	{
		console.log(codePoint);
	}
>除了遍历字符串，这个遍历器最大的优点是可以识别大于0xFFFF的码点，传统的for循环无法识别这样的码点。

	var text = String.fromCodePoint(0x20BB7);

	for (let i = 0; i < text.length; i++) {
	  console.log(text[i]);
	}
	// " "
	// " "
	
	for (let i of text) {
	  console.log(i);
	}
	// "𠮷"

##5、at()
>ES5对字符串对象提供`charAt`方法，返回字符串给定位置的字符。该方法不能识别码点大于`oxFFFF`的字符。

	console.log('abc'.charAt(0));
	console.log('𠮷'.charAt(0));

>ES7提供了字符串实例的at方法，可以识别Unicode编号大于0xFFFF的字符，返回正确的字符。Chrome浏览器已经支持该方法。

	console.log('abc'.at(0)); // "a"
	console.log('𠮷'.at(0)); // "𠮷"

##6、normalize()
>为了表示语调和重音符号，Unicode提供了两种方法。一种是直接提供带重音符号的字符，比如Ǒ（\u01D1）。另一种是提供合成符号（combining character），即原字符与重音符号的合成，两个字符合成一个字符，比如O（\u004F）和ˇ（\u030C）合成Ǒ（\u004F\u030C）。

	console.log('\u01D1'==='\u004F\u030C'); //false

	console.log('\u01D1'.length); // 1
	console.log('\u004F\u030C'.length); // 2

	JavaScript将合成字符视为两个字符，导致两种表示方法不相等。

>ES6提供字符串实例的normalize()方法，用来将字符的不同表示方法统一为同样的形式，这称为Unicode正规化。

	console.log('\u01D1'.normalize() === '\u004F\u030C'.normalize());//true

>normalize方法可以接受四个参数。

	NFC，默认参数，表示“标准等价合成”（Normalization Form Canonical Composition），返回多个简单字符的合成字符。所谓“标准等价”指的是视觉和语义上的等价。
	NFD，表示“标准等价分解”（Normalization Form Canonical Decomposition），即在标准等价的前提下，返回合成字符分解的多个简单字符。
	NFKC，表示“兼容等价合成”（Normalization Form Compatibility Composition），返回合成字符。所谓“兼容等价”指的是语义上存在等价，但视觉上不等价，比如“囍”和“喜喜”。（这只是用来举例，normalize方法不能识别中文。）
	NFKD，表示“兼容等价分解”（Normalization Form Compatibility Decomposition），即在兼容等价的前提下，返回合成字符分解的多个简单字符。

>例如：NFC参数返回字符的合成形式，NFD参数返回字符的分解形式。

	console.log('\u004F\u030C'.normalize('NFC').length); // 1
	console.log('\u004F\u030C'.normalize('NFD').length); // 2
>normalize方法目前不能识别三个或三个以上字符的合成。这种情况下，还是只能使用正则表达式，通过Unicode编号区间判断。

##7、includes(),startsWidth(),endsWidth()

>JavaScript只有indexOf方法，可以用来确定一个字符串是否包含在另一个字符串中。ES6又提供了三种新方法。

>includes()：返回布尔值，表示是否找到了参数字符串。
startsWith()：返回布尔值，表示参数字符串是否在源字符串的头部。
endsWith()：返回布尔值，表示参数字符串是否在源字符串的尾部。

	var s = 'Hello world!';

	console.log(s.startsWith('Hello')); // true
	console.log(s.endsWith('!')); // true
	console.log(s.includes('o')); // true

>三个方法都支持第二个参数，表示开始搜索的位置。

##8、repeat()
>`repeat`方法返回一个新字符串，表示将原字符串重复`n`次。

	console.log('x'.repeat(3)); // "xxx"
	console.log('hello'.repeat(2)); // "hellohello"
	console.log('na'.repeat(0)); // ""
	
	console.log('na'.repeat(Infinity));
	// RangeError
	console.log('na'.repeat(-1));
	// RangeError
>如果参数是0到-1之间的小数，则等同于0，这是因为会先进行取整运算。0到-1之间的小数，取整以后等于-0，repeat视同为0。

	console.log('na'.repeat(-0.9));
	//""
	console.log('na'.repeat(NaN));
	//""
>如果repeat的参数是字符串，则会先转换成数字。

	console.log('na'.repeat('na')); // ""
	console.log('na'.repeat('3')); // "nanana"


##9、padStart(),padEnd()

>ES7推出了字符串补全长度的功能。如果某个字符串不够指定长度，会在头部或尾部补全。`padStart`用于头部补全，`padEnd`用于尾部补全。

	console.log('x'.padStart(5, 'ab')); // 'ababx'
	console.log('x'.padStart(4, 'ab')); // 'abax'

	console.log('x'.padEnd(5, 'ab')); // 'xabab'
	console.log('x'.padEnd(4, 'ab')); // 'xaba'

>如果原字符串的长度，等于或大于指定的最小长度，则返回原字符串。

	console.log('xxx'.padStart(2, 'ab')); // 'xxx'
	console.log('xxx'.padEnd(2, 'ab')); // 'xxx'
>如果省略第二个参数，则会用空格补全长度。

	console.log('x'.padStart(4)); // '   x'
	console.log('x'.padEnd(4)); // 'x   '

##10、模板字符串
>ES6引入了模板字符串解决传统的JavaScript输出模板语言。

	$("#result").append(`
	  There are <b>${basket.count}</b> items
	   in your basket, <em>${basket.onSale}</em>
	  are on sale!
	`);

>模板字符串（template string）是增强版的字符串，用反引号（`）标识。它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量。

	console.log(`string text line 1
	string text line 2`);
	//string text line 1
	//string text line 2
	
	// 字符串中嵌入变量
	var name = "Bob", time = "today";
	console.log(`Hello ${name}, how are you ${time}?`);
	//'Hello Bob, how are you today?'

>如果使用模板字符串表示多行字符串，所有的空格和缩进都会被保留在输出之中。模板字符串中嵌入变量，需要将变量名写在${}之中。

	function authorize(user, action) {
	  if (!user.hasPrivilege(action)) {
	    throw new Error(
	      // 传统写法为
	      // 'User '
	      // + user.name
	      // + ' is not authorized to do '
	      // + action
	      // + '.'
	      `User ${user.name} is not authorized to do ${action}.`);
	  }
	}

>大括号内部可以放入任意的JavaScript表达式，可以进行运算，以及引用对象属性。

	var x = 1;
	var y = 2;
	
	console.log(`${x} + ${y} = ${x + y}`);
	// "1 + 2 = 3"
	
	console.log(`${x} + ${y * 2} = ${x + y * 2}`);
	// "1 + 4 = 5"
	
	var obj = {x: 1, y: 2};
	console.log(`${obj.x + obj.y}`);

	//模板字符串中还能调用函数
	function fn() {
	  return "Hello World";
	}
	
	console.log(`foo ${fn()} bar`);
	// foo Hello World bar
	
>如果大括号中的值不是字符串，将按照一般的规则转为字符串。比如，大括号中是一个对象，将默认调用对象的`toString`方法。

>如果模板字符串中的变量没有声明，将报错。由于模板字符串的大括号内部，就是执行JavaScript代码，因此如果大括号内部是一个字符串，将会原样输出。
	
	//变量place没有声明
	var msg=`Hello,${place}`;
	//ReferenceError

	console.log(`Hello ${`World`}`);
>引用模板字符串本身，可以像下面这样写：

	// 写法一
	let str = 'return ' + '`Hello ${name}!`';
	let func = new Function('name', str);
	func('Jack') // "Hello Jack!"
	
##11、实例：模板编译
>通过模板字符串，生成正式模板的实例：


	var template = `
	<ul>
	  <% for(var i=0; i < data.supplies.length; i++) {%>
	    <li><%= data.supplies[i] %></li>
	  <% } %>
	</ul>
	`;

>编译模板字符串:

	function compile(template){
	  var evalExpr = /<%=(.+?)%>/g;
	  var expr = /<%([\s\S]+?)%>/g;
	
	  template = template
	    .replace(evalExpr, '`); \n  echo( $1 ); \n  echo(`')
	    .replace(expr, '`); \n $1 \n  echo(`');
	
	  template = 'echo(`' + template + '`);';
	
	  var script =
	  `(function parse(data){
	    var output = "";
	
	    function echo(html){
	      output += html;
	    }
	
	    ${ template }
	
	    return output;
	  })`;
	
	  return script;
	}

>`compile`函数的用法如下:

	var parse = eval(compile(template));
	div.innerHTML = parse({ supplies: [ "broom", "mop", "cleaner" ] });
	//   <ul>
	//     <li>broom</li>
	//     <li>mop</li>
	//     <li>cleaner</li>
	//   </ul>

##12、标签模板
>模板字符串的功能，不仅仅是上面这些。它可以紧跟在一个函数名后面，该函数将被调用来处理这个模板字符串。这被称为“标签模板”功能（tagged template）。

>标签模板其实不是模板，而是函数调用的一种特殊形式。“标签”指的就是函数，紧跟在后面的模板字符串就是它的参数。

	var a = 5;
	var b = 10;
	
	console.log(tag`Hello ${ a + b } world ${ a * b }`);
	//`tag`是一个函数，整个表达式的返回值，就是`tag`函数处理模板字符串后的返回值。

>函数`tag`依次会接收到多个参数:

	function tag(stringArr,value1,value2){
		// code here...
	}

##13、String.raw()
>ES6还为原生的String对象，提供了一个`raw`方法。

>`String.raw`方法，往往用来充当模板字符串的处理函数，返回一个斜杠都被转义(即斜杠前面再加一个斜杠)的字符串，对应于替换变量后的模板字符串。

	console.log(String.raw`Hi\n${2+3}!`);
	// "Hi\\n5!"
	
	console.log(String.raw`Hi\u000A!`);
	// 'Hi\\u000A!'

	//String.raw的代码基本如下：
	String.raw=function(strings,...values) {
		var output="";
		for(var index=0;index<values.length;index++) {
			output+=strings.raw[index]+values[index];
		}

		output+=strings.raw[index];
		return output;
	}


>`String.raw`方法可以作为处理模板字符串的基本方法，它会将所有变量替换，而且对斜杠进行转义，方便下一步作为字符串来使用。
>`String.raw`方法也可以作为正常的函数使用。这时，它的第一个参数，应该是一个具有`raw`属性的对象，且`raw`属性的值应该是一个数组。 

	String.raw({raw:`test`},0,1,2);
	//`t0e1s2t`

	//等同于
	String.raw({raw:[`t`,`e`,`s`,`t`]},0,1,2);

