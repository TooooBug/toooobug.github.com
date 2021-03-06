Title: [译]Shadow DOM第二课
Date: 2013-07-21 16:50:58
Tags: "Shadow DOM" "Web Components"

本文将会讨论Shadow DOM的更多神奇之处。本文是在[《Shadow DOM第一课》](http://www.toobug.net/article/shadow_dom_101.html)的基础之上讨论的，如果你需要基础介绍，请参看那篇。

## 介绍

无法否认，没有样式的结构是很无趣的。幸好，[Web Components](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/explainer/index.html#acknowledgements)的制定者们很早就意识到了这个问题，并为我们提供了好几种为Shadow DOM中的结构指定样式的方法。

## 样式封装

Shadow DOM中有一个很核心的特性叫“shadow边界”（shadow boundary），它有很多好用的特点，其中的一点就是提供了样式封装。换句话说：默认情况下，在Shadow DOM中定义的样式会被限制在Shadow Root的范围中。

$$solo_more$$

下面是一个例子。如果你的浏览器支持Shadow DOM的话，将会看到“Shadow DOM Title”。

	<div><h3>Host title</h3></div>
	<script>
	var root = document.querySelector('div').webkitCreateShadowRoot();
	root.innerHTML = '<style>h3{ color: red; }</style>' + 
					'<h3>Shadow DOM Title</h3>';
	</script>

<div id="example1"><h3>Host title</h3></div>
<script>
var root = document.querySelector('div#example1').webkitCreateShadowRoot();
root.innerHTML = '<style>h3{ color: red; }</style>' + 
				'<h3>Shadow DOM Title</h3>';
</script>

<div class="helperimg" style="display:none;">
	<p>您的浏览器不支持Shadow DOM，这个例子的正确样子是这样的：</p>
	<p><img alt="不支持shadow dom的同学看这个图片" src="/images/shadow_dom_201_1.png" /></p>
</div>

在这个例子中，有两个值得注意的点：

- 在页面上有<a href="javascript:alert('页面上一共有' + document.querySelectorAll('h3').length + '个<h3>。')">其它的h3</a>，但唯一一个被上面的样式选中并变成红色的只有Shadow Root中的。也就是说，默认情况下，样式的作用范围被限制了。
- 页面中用于其它h3的样式没有被应用到Shadow DOM中去，因为选择器无法穿越shadow边界。

很神奇吧？感谢Shadow DOM，让我们可以将外部样式封装起来。

## 为shadow host元素指定样式

`@host`是一种[at-rule](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#host-at-rule)，它可以选择那些挂载了Shadow DOM的元素（即shadow host）。

	<button class="bigger">My Button</button>
	<script>
	var root = document.querySelector('button').webkitCreateShadowRoot();
	root.innerHTML = '<style>' + 
		'@host{' + 
			'button { text-transform: uppercase; }' +
			'.bigger { padding: 20px; }' +
		'}' +
		'</style>' + 
		'<content select=""></content>';
	</script>
	<button class="bigger">My Button</button>

<button id="example2" class="bigger">My Button</button>
<script>
var root = document.querySelector('button#example2').webkitCreateShadowRoot();
root.innerHTML = '<style>' + 
	'@host{' + 
		'button { text-transform: uppercase; }' +
		'.bigger { padding: 20px; }' +
	'}' +
	'</style>' + 
	'<content select=""></content>';
</script>

<div class="helperimg" style="display:none;">
	<p>您的浏览器不支持Shadow DOM，这个例子的正确样子是这样的：</p>
	<p><img alt="不支持shadow dom的同学看这个图片" src="/images/shadow_dom_201_2.png" /></p>
</div>

需要特别注意的是，包裹在`@host`中的样式规则比页面中的任何选择器中的样式优化先都要高，但比shadow host元素上的`style`属性优先级低。`@host`仅仅在Shadow Root所在的范围中才有效，不可以在Shadow DOM之外使用。

使用`@host`的典型场景是在创建自定义元素时，比如对不同的用户态（:hover，:focus，:active等）应用不同的样式。

	<style>
	@host {
		* {
			opacity: 0.4;
			+transition: opacity 420ms ease-in-out;
		}
		*:hover {
			opacity: 1;
		}
		*:active {
			position: relative;
			top: 3px;
			left: 3px;
		}
	}
	</style>

<button id="example3">My Button</button>
<script>
var root = document.querySelector('button#example3').webkitCreateShadowRoot();
root.innerHTML = '<style>' + 
	'@host{' + 
		'* {' +
			'opacity: 0.4;' +
			'+transition: opacity 420ms ease-in-out;' +
		'}' +
		'*:hover {' +
			'opacity: 1;' +
		'}' +
		'*:active {' +
			'position: relative;' +
			'top: 3px;' +
			'left: 3px;' +
		'}' +
	'}' +
	'</style>' + 
	'<content select=""></content>';
</script>

<div class="helperimg" style="display:none;">
	<p>您的浏览器不支持Shadow DOM，这个例子的正确样子是这样的：</p>
	<p><img alt="不支持shadow dom的同学看这个图片" src="/images/shadow_dom_201_3.gif" /></p>
</div>

在这个例子中，我使用了`*`选择器来选择承载我的Shadow DOM的元素。也就是“我不关心你是什么元素，只要按这样的样式显示就行了。”

另一种需要使用`@host`的场景是你想从Shadow DOM内部为不同类型的shadow host指定不同的样式，比如shadow host是自定义元素时就需要这样。当然你还可以根据shadow host元素的类型来创建不同的皮肤。

	@host {
		g-foo { 
			/* Applies if the host is a <g-foo> element.*/
		}

		g-bar {
			/* Applies if the host is a <g-bar> element. */
		}

		div {
			/* Applies if the host element is a <div>. */
		}

		* {
			/* Applies to any type of element hosting this ShadowRoot. */
		}
	}

## 创建样式勾子

某些场景下让用户自定义是极好的，此时你可能希望打破样式封装，留出一些勾子给其它人来指定样式。

### 使用自定义伪元素

Webkit和Firefox都定义了一些伪元素，用于给原生的元素指定样式。比如`input[type=range]`，你可以使用`::-webkit-slider-thumb`来给滑块定义样式。

	input[type=range].custom::-webkit-slider-thumb {
		-webkit-appearance: none;
		background-color: blue;
		width: 10px;
		height: 40px;
	}

与原生元素提供的这些样式勾子一样，Shadow DOM的作者也可以指定一些可供外部指定样式的元素。具体的方法是使用[自定义伪元素](http://www.w3.org/TR/shadow-dom/#custom-pseudo-elements)。

你可以通过使用`pseudo`属性来指定某个元素作为自定义伪元素，它的值（名字）需要加上前缀“x-”。这样就可以让外部通过伪元素名穿过shadow边界，选择到这个Shadow DOM中的元素。

下面是一个创建自定义滑块挂件的例子，可以通过伪元素让滑块变成蓝色：

	<style>
		#host::x-slider-thumb {
			background-color: blue;
		}
	</style>
	<div id="host"></div>
	<script>
	var root = document.querySelector('#host').webkitCreateShadowRoot();
	root.innerHTML = '<div>' +
						'<div pseudo="x-slider-thumb"></div>' + 
					'</div>';
	</script>

> 看起来很不错是么？你可以通过外部的CSS来指定样式，但不可以通过外部的JS来访问元素。shadow边界仍然对JS有效，但对CSS中使用自定义伪元素例外了。

### 使用CSS变量

> CSS变量功能可以在Chrome的about:flags中通过“启用实验性 WebKit 功能”开启。

另外一个可以用于“换肤”的强大方式就是[CSS变量](http://dev.w3.org/csswg/css-variables/)。本质上，就是创建一个“样式占位符”，让其它人来填充这占位符。

一种可能遇到的场景就是自定义元素的作者在Shadow DOM内部使用了一些“样式占位符”，比如一个用于改变按钮字体的变量和一个用于改变颜色的变量：

	button {
		color: +var (button-text-color, pink); /* default color will be pink */
		font: +var (button-font) ;
	}

然后，这个元素的使用者就可以根据他们的喜好来定义这些值，比如让它使用页面上的“Comic Sans”风格：

	#host {
		+var-button-text-color: green;
		+var-button-font: "Comic Sans MS", "Comic Sans", cursive;
	}

因为有CSS变量的继承机制，一切都很完美！完整代码如下：

	<style>
		#host {
			+var-button-text-color: green;
			+var-button-font: "Comic Sans MS", "Comic Sans", cursive;
		}
	</style>
	<div id="host">Host node</div>
	<script>
	var root document.querySelector('#host').webkitCreateShadowRoot();
	root.innerHTML = '<style>' + 
			'button {' + 
				'color: +var (button-text-color, pink);' + 
				'font: +var (button-font) ;' + 
			'}' +
			'</style>' +
			'<content></content>';
	</script>

> 在这篇文章中，已经有好处提到了[自定义元素](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/custom/index.html#the-element-element)。我不会去讲自定义元素的知识，从现在起，你只需要知道Shadow DOM的结构基础和样式概念是部分来自自定义元素即可。

## 继承和重设样式

有某些情况下，你可能希望让外部样式影响到Shadow DOM内部。一个常见的例子是评论挂件。大部分作者在嵌入评论挂件的时候都希望它长得像自己的页面，至少我是这样。我们需要一种方法来统一挂件的风格，比如通过继承字体、颜色、行高等。

为了灵活性考虑，Shadow DOM允许我们在样式封装的边界上打出更多的“洞”。有两个属性可以控制来自外部的样式：

- `.resetStyleInheritance`
	- `false` 默认值，[http://www.impressivewebs.com/inherit-value-css/](这里)可以了解更多关于CSS继承的内容
	- `true` 在边界处将所有可以被继承的属性都置为`initial`
- `.applyAuthorStyles`
	- `true` 使用作者在文档中定义的样式，可理解为“外部样式可以渗透到Shadow DOM内部”
	- `false` 默认值，外部样式不会被应用到Shadow DOM内部

下面是一个demo，用于演示改变这两个值时Shadow DOM会有怎样的变化。

	<div><h3>Host title</h3></div>
	<script>
	var root = document.querySelector('div').webkitCreateShadowRoot();
	root.applyAuthorStyles = true;
	root.resetStyleInheritance = false;
	root.innerHTML = '<style>h3{ color: red; }</style>' + 
									 '<h3>Shadow DOM Title</h3>' + 
									 '<content select="h3"></content>';
	</script>


<div id="example4"><h3>Host title</h3></div>
<button id="example4_applyAuthorStyles" data-boolean="true">applyAuthorStyles=true</button>
<button id="example4_resetStyleInheritance" data-boolean="false">resetStyleInheritance=false</button>
<script>
~function(){
	var root = document.querySelector('div#example4').webkitCreateShadowRoot();
	root.applyAuthorStyles = true;
	root.resetStyleInheritance = true;
	root.innerHTML = '<style>h3{ color: red; }</style>' + 
									 '<h3>Shadow DOM Title</h3>' + 
									 '<content select="h3"></content>';
	function example4(options){
		if(typeof options.applyAuthorStyles === 'boolean'){
			root.applyAuthorStyles = options.applyAuthorStyles;
			$('#example4_applyAuthorStyles').text('applyAuthorStyles='+options.applyAuthorStyles);
		}
		if(typeof options.resetStyleInheritance === 'boolean'){
			root.resetStyleInheritance = options.resetStyleInheritance;
			$('#example4_resetStyleInheritance').text('resetStyleInheritance='+options.resetStyleInheritance);
		}
	}
	$('#example4_applyAuthorStyles').click(function(){

		var $this = $(this);
		var targetBoolean = !$this.data('boolean');
		example4({
			applyAuthorStyles:targetBoolean
		});
		$this.data('boolean',targetBoolean);

	});
	$('#example4_resetStyleInheritance').click(function(){

		var $this = $(this);
		var targetBoolean = !$this.data('boolean');
		example4({
			resetStyleInheritance:targetBoolean
		});
		$this.data('boolean',targetBoolean);

	});
}();
</script>

<div class="helperimg" style="display:none;">
	<p>您的浏览器不支持Shadow DOM，这个例子的正确样子是这样的：</p>
	<p><img alt="不支持shadow dom的同学看这个图片" src="/images/shadow_dom_201_4.gif" /></p>
</div>

上面的例子可以很容易地看到`.applyAuthorStyles`是怎样工作的。它使得Shadow DOM中的h3可以继承页面中h3元素的样式。（译注：你可能会奇怪为何这两个h3的样式不一样呢？不是继承了文档中的样式了吗？这是因为文档中的h3样式前面还有其它的选择器，不是直接写的`h3{...}`，而是类似`#container h3{...}`，这样的话因为h3在Shadow DOM中，而前面的选择器在文档中，导致CSS选择器产生跨越边界的行为，因此选择不到，见下段。）

> 即使设置了`apply-author-styles`属性，在文档中定义的CSS选择器仍然无法超过shadow边界。*只有完全在Shadow DOM内部或者外部的样式规则才会被匹配。*

理解`.resetStyleInheritance`有点麻烦，主要是因为它只在可以被继承的属性上才有效果。它的含义是：当浏览器在往上寻找某个可以继承的属性值时（比如`color`），在页面和Shadow Root之间的边界上，这些值不再被继承，而是使用`initial`代替（根据CSS标准）。（译注：举个例子，文档中定义了`body{color:red}`，此时如果是文档中有一个`p`元素，计算`color`属性值时就会往上寻找到`body`的`color`并继承，但如果是Shadow DOM中有一个`p`元素，则`color`值在浏览器寻找到shadow边界时被置为`initial`，而不是继续到Shadow DOM外部寻找继承。）

如果你不确定哪个属性在CSS中会被继承，可以查看这个[手册](http://www.impressivewebs.com/inherit-value-css/)，或者在开发工具中的Element面板中切换“Show inherited”选项。

![在调试工具中查看继承的样式](/images/shadow_dom_201_showinheritance.gif)

### 小抄

为了更好地理解这两个属性，下面有一个表格。赶紧把它收藏进口袋吧，很金贵的哦！

<table>
	<thead>
		<tr>
			<th>场景</th>
			<th>applyAuthorStyles</th>
			<th>resetStyleInheritance</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				<p>“我有自己的样式，但是希望继承一些基础样式，比如文本颜色”<p>
				<p><cite>比如，在写一个挂件</cite></p>
			</td>
			<td>false</td>
			<td>false</td>
		</tr>
		<tr>
			<td>
				<p>“让页面滚蛋去吧，我有自己的主题”<p>
				<p><cite>你仍然需要一个组件内部使用的css reset，因为被分配到内部的内容会带上它在页面中拥有的样式</cite></p>
			</td>
			<td>false</td>
			<td>true</td>
		</tr>
		<tr>
			<td>
				<p>“我希望从页面样式中获得我的主题”<p>
			</td>
			<td>true</td>
			<td>true</td>
		</tr>
		<tr>
			<td>
				<p>“我希望让页面样式尽可能地渗透给我”<p>
				<p><cite>记住，CSS选择器无法跨越shadow边界</cite></p>
			</td>
			<td>true</td>
			<td>false</td>
		</tr>
	</tbody>
</table>

## 给被分配的节点指定样式

`.applyAuthorStyles`/`.resetStyleInheritance`只会影响定义在Shadow DOM内部的节点的样式。

被分配的节点可以算是“异类”，从逻辑上讲，它们不属于Shadow DOM，它们仍然是Shadow host元素的子元素，只是在渲染的时候被移到另外一个地方去了。很自然地，它们会带上从文档中获取到的样式。唯一的例外规则是它们可以在渲染后的新的地方（Shadow DOM中）获取新的样式。

### ::distributed() 伪元素

如果被分配的节点是Shadow host元素的子元素，那怎样从Shadow DOM内部来找到它们并给它们写新的样式呢？答案是`::distributed()`伪元素。这是第一个“函数式”（functional）伪元素，它接受一个CSS选择器作为它的参数。

我们来看一个简单的例子：

	<div><h3>Host title</h3></div>
	<script>
	var root = document.querySelector('div').webkitCreateShadowRoot();
	root.innerHTML = '<style>' + 
					   'h3{ color: red; }' + 
					   'content::-webkit-distributed(h3) { color: green; }' + 
					 '</style>' + 
					 '<h3>Shadow DOM Title</h3>' +
					 '<content select="h3"></content>';
	</script>

<div id="example5"><h3>Host title</h3></div>
<script>
var root = document.querySelector('div#example5').webkitCreateShadowRoot();
root.innerHTML = '<style>' + 
				   'h3{ color: red; }' + 
				   'content::-webkit-distributed(h3) { color: green; }' + 
				 '</style>' + 
				 '<h3>Shadow DOM Title</h3>' +
				 '<content select="h3"></content>';
</script>

<div class="helperimg" style="display:none;">
	<p>您的浏览器不支持Shadow DOM，这个例子的正确样子是这样的：</p>
	<p><img alt="不支持shadow dom的同学看这个图片" src="/images/shadow_dom_201_5.png" /></p>
</div>

你应该可以看到红色的“Shadow DOM Title”和绿色的“Host Title”。同时注意到，“Host Title”仍然保留了从文档（页面）中带来的样式。

### 在插入点（insertion points）重设样式

在创建ShadowRoot的时候，你可以选择重设继承的样式。`<content>`和`<shadow>`插入点也可以进行同样的选择。当我们使用这些元素的时候，可以通过在JS中设置`.resetStyleInheritance`布尔属性，或者在元素本身上设置`reset-style-inheritance`属性。

- 对ShadowRoot或者是`<shadow>`插入点来说：`reset-style-inheritance`意味着，可继承的CSS属性在到达Shadow DOM之前，在shadow host处被置为`initial`。这个位置也就是熟知的“上边界”（upper boundary）。
- 对`<content>`插入点来说：`reset-style-inheritance`意味着，可继承的CSS属性在shadow host的子元素被分配之前会被置为`initial`。这个位置也就是熟知的“下边界”（lower boundary）。

（译注：其实没懂上下边界的差异在哪里，求指教……）

> 特别注意：在文档中定义的样式会继续应用到它们选择到的那些节点，即使这些节点被分配到Shadow DOM内部。节点跑到Shadow DOM内部并不会改变那些已经应用的样式。

## 总结

对自定义元素的作者来说，可以有N多种控制样式的办法。Shadow DOM成为了这些办法中最基础的部分。

Shadow DOM为我们提供了样式封装（scoped style），以及一种可以选择性让外部样式渗透的方法。通过自定义伪元素和CSS变量，作者可以提供给第三方使用者一些样式勾子，以便使用者进一步自定义他们的内容。最终，web的作者们仍然可以完全控制内容的呈现。

> 原文地址<http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/?redirect_from_locale=zh>

<script>
	/*var css = document.createElement('link');
	css.setAttribute('rel','stylesheet');
	css.setAttribute('href','/attachments/shadow_dom_101_style.css');
	document.head.appendChild(css);*/

	if(!window.WebKitShadowRoot){
		$('.helperimg').css({

			border:'1px solid #ccc',
			background:'#eee',
			padding:'20px'

		}).show();
	}
</script>
