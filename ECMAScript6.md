#ECMAScript 6运行环境
#
##1、Babel转码器(ES6转码器)
>Babel是一个广泛使用的ES6转码器，可以将ES6代码转为ES5代码，从而在浏览器或其他环境中执行。

	// 转码前
	input.map(item => item + 1);
		
	// 转码后
	input.map(function (item) { return item + 1;
	});

###命令行环境
>Babel的安装命令如下:

	$ npm install --global babel-cli
	$ npm install --save babel-preset-es2015
>当前目录下，需新建一个配置文件`.babelrc`

	//.babelrc
	{
		"presets":['es2015']
	}
>Babel自带一个`babel-node`命令，提供支持ES6的REPL环境。可直接运行ES6代码。

	$ babel-node
	console.log([1,2,3].map(x => x*x))
>`babel-node`命令也可以直接运行ES6脚本。
	
	$ babel-node ex_es6.js
>`babel`命令可以将ES6代码转换成ES5代码。

	$babel es_es6.js
	"use strict";

	console.log([1,2,3].map(function (x) {
		return x*x;
	}));

>`-o`参数将转换后的代码，从标准输出导入文件。

	$ babel ex_es6.js -o ex_es5.js
	# or
	$ babel ex_es6.js --out-file ex_es5.js

>`-d`参数用于转换整个目录。

	//如果希望生成source map文件，要加上`-s`参数
	$ babel -d bulid-dir source-dir -s

###浏览器环境
>Babel也可以用于浏览器。但是，从Babel 6.0开始，不再直接提供浏览器版本，而是要用构建工具构建出来。如果不想使用构建工具，只有通过安装5.x版本的babel-core模块获取。

	$ npm install babel-core@5
>运行上面的命令后，可以在当前目录的`node_modules/babel-core/`子目录里面，找到`babel`的浏览器的版本`browser.js`。

>具体在网页中的使用如下:

	<script src="node_modules/babel-core/browser.js"></script>
	<script type="text/babel">
	// ES6 code
	</script>
>以上写法是实时将ES6代码转为ES5，对网页性能会有影响。
>下面是`Babel`配合`Browserify`一起使用，可以生成浏览器能够直接加载的脚本。
>安装`babelify`模块：

	$npm install --save-dev babelify babel-preset-es2015
>命令行转换ES6脚本：

	$ browserify script.js -o boudle.js -t [ babelify --persets [es2015 react]]
>可以在`package.json`设置下面的代码，这样就不用每次在命令行输入参数。

	{
		"browserify":{
		"transform":[["babelify",{"presets":["es2015"]}]]	
	}
	}

###Node环境
>Node脚本中，需要转换ES6脚本，可以采用如下方法。
>先安装`babel-core`和`babel-preset-es2015`:

	$ npm install --save-dev babel-core babel-preset-es2015
>然后在项目根目录下，新建一个`.babelrc`文件:

	{
		"presets":["es2015"]
	}
>然后在脚本中调用`babel-core`的`transform`方法:

	var es5Code = 'let x = n => n + 1';
	var es6Code = require('babel-core')
  	.transform(es5Code, {presets: ['es2015']})
  	.code;	

	//transform方法的第一个参数是一个字符串，表示需要转换的ES5代码，第二个参数是转换的配置对象。

>Node脚本还有一种特殊的`babel`用法，即把`babel`加载为`require`命令的一个钩子。安装`babel-core`和`babel-preset-es2015`以后，先在项目的根目录下，设置一个配置文件`.babelrc`

	//.babelrc	
	{
		"presets":["es2015"]
	}
>然后在应用的入口脚本头部加入下面的语句：

	require("babel-core/register");
	//有了上面这行语句，后面所有通过require命令加载的后缀名为.es6、.es、.jsx和.js的脚本，都会先通过babel转码后再加载。
>Babel默认不会转换Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法（比如Object.assign）。如果用到了这些功能，当前的运行环境又不支持。就需要安装babel-polyfill模块:

	$ npm install babel-polyfill --save
>然后在所有脚本头部加上:

	require('babel-polyfill');
	# or
	import 'babel-polyfill';

###在线转换
>Babel提供一个REPL在线编译器，可以在线将ES6代码转为ES5代码。转换后的代码，可以直接作为ES5代码插入网页运行。

##2、Traceur转码器
>Google公司的Traceur转码器，也可以将ES6代码转为ES5代码。

###直接插入网页
>Traceur允许将ES6代码直接插入网页。首先，必须在网页头部加载Traceur库文件。

	<!-- 加载Traceur编译器 -->
	<script src="http://google.github.io/traceur-compiler/bin/traceur.js"
	        type="text/javascript"></script>
	<!-- 将Traceur编译器用于网页 -->
	<script src="http://google.github.io/traceur-compiler/src/bootstrap.js"
	        type="text/javascript"></script>
	<!-- 打开实验选项，否则有些特性可能编译不成功 -->
	<script>
	        traceur.options.experimental = true;
	</script>
>接下来，就可以把ES6代码放入上面这些代码的下方：

	<script type="module">
	  class Calc {
	    constructor(){
	      console.log('Calc constructor');
	    }
	    add(a, b){
	      return a + b;
	    }
	  }
	
	  var c = new Calc();
	  console.log(c.add(4,5));
	</script>
	# or
	<script type="module" src="calc.js" >
	</script>
>`script`标签的`type`属性的值是`module`，而不是`text/javascript`。这是Traceur编译器识别ES6代码的标识，编译器会自动将所有`type=module`的代码编译为ES5，然后再交给浏览器执行。

###在线转换
>Traceur也提供一个在线编译器，可以在线将ES6代码转为ES5代码。转换后的代码，可以直接作为ES5代码插入网页运行。

	<script src="http://google.github.io/traceur-compiler/bin/traceur.js"
	        type="text/javascript"></script>
	<script src="http://google.github.io/traceur-compiler/src/bootstrap.js"
	        type="text/javascript"></script>
	<script>
	        traceur.options.experimental = true;
	</script>
	<script>
	$traceurRuntime.ModuleStore.getAnonymousModule(function() {
	  "use strict";
	
	  var Calc = function Calc() {
	    console.log('Calc constructor');
	  };
	
	  ($traceurRuntime.createClass)(Calc, {add: function(a, b) {
	    return a + b;
	  }}, {});
	
	  var c = new Calc();
	  console.log(c.add(4, 5));
	  return {};
	});
	</script>

###命令行转换
>作为命令行工具使用时，Traceur是一个Node.js的模块，首先需要用npm安装。

	$ npm install -g traceur
>traceur直接运行es6脚本文件，会在标准输出显示运行结果，以前面的calc.js为例。
	
	$ traceur calc.js
	Calc constructor
	9

>如果要将ES6脚本转为ES5保存，要采用下面的写法

	$ traceur --script calc.es6.js  --out calc.es5.js
>上面`--script`选项表示指定输入文件，`--out`选项表示指定输出文件。
>为了防止有些特性编译不成功，最好加上`--experimental`选项。

	$ traceur --script calc.es6.js --out  calc.es5.js --experimental
>命令行下转换的文件，就可以放在浏览器中运行。

###Node.js环境的用法(已安装traceur模块)
>Traceur的Node.js用法如下：

	var traceur = require('traceur');
	var fs = require('fs');
	
	// 将ES6脚本转为字符串
	var contents = fs.readFileSync('es6-file.js').toString();
	
	var result = traceur.compile(contents, {
	  filename: 'es6-file.js',
	  sourceMap: true,
	  // 其他设置
	  modules: 'commonjs'
	});
	
	if (result.error)
	  throw result.error;
	
	// result对象的js属性就是转换后的ES5代码
	fs.writeFileSync('out.js', result.js);
	// sourceMap属性对应map文件
	fs.writeFileSync('out.js.map', result.sourceMap);
