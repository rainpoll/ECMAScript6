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