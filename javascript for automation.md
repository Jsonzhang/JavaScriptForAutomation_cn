
#说明
在 OS X Yosemite 之后的OS系统中我们可以使用 Javascript 来完成自动完成一些系统的工作，这篇文章是有关这方面的知识阐述。

#内容

+  [简介](#jianjie)
+  [访问应用程序]()
+  [语法实例]()
+  [获取属性，设置属性]()
+  [元素数组](#arrayData)
+  [过滤数组]()
+  [调用命令]()
+  [创建对象]()
+  [添加脚本]()
+  [Strict 标记]()
+  [Timeout, Considering, 和 Ignoring]()
+  [ObjectSpecifier]()
+  [路径]()
+  [Progress]()
+  [类库]()
+  [Applets(小应用程序)]()
+  [UI 自动化]()
+  [Objective-C 桥接器]()
+  [Automation.getDisplayString()]()

[id]: jianjie
##简介
Javascript 在 Mac 环境下执行可以引用以下这些全局变量来完成一些系统级别的任务：
`Automation`  `Application`  `Library`  `Path`  `Progress`  `ObjectSpecifier`  `delay` `console.log`  `ObjC`  `Ref`  `$`

###OSA 组件

在OS系统中通过调用 Javascript 的 OSA 组件来实现自动完成系统工作的功能。在脚本编辑器，全局脚本菜单，Automator程序中的”运行Javascript”操作，applets/droplets ， osascript 命令行工具， NSUserScriptTask API 等等环境中都会调用这个 OSA 组件，所以在这些环境里你可以像写 Applescript 一样，用 Javascript 来自动完成诸如操作邮件，操作文件夹，地址簿插件，日历闹钟脚本或者发出触发事件的消息这类工作。

###脚本字典

打开查看脚本字典有两种方法，分别是在菜单选项中选择 文件 > 打开字典 和 窗口 > 资源库。     
脚本字典会详细列举每个应用中可被操作调用的类，对象和方法，包括了 AppleScript , Javascript，Objective-C (通过脚本桥接框架实现) 等好几种语言格式，这些可操作的对象都会按规范生成对应的 Javascript 变量列表供开发者操作。

###对象修饰符
在以上提到的这些引用了OSA的环境中，当你获取了一个 `Appication` 对象的 JavaScript 属性时，无论这个属性对象指向的是其他 App，还是其他 App 窗口，或者是外部数据，这个对象都只是一个对象修饰符而不是一个真正意义的实体对象。`Application` 对象上的 Javascript 属性对象都只是对象指针，并不是对应实体的真实值，而只是一个指向而已。如果要从被引用的属性上更新对应的值，必须做一些额外的步骤，这些步骤会在 "获取属性和设置属性" 这一节中提到.

##访问应用程序
应用程序可以被下面这几种方式调用访问到：

1. 通过名称

		Application('Mail')

2. 通过Bundle ID调用：

		Application('com.apple.mail')
3. 通过路径调用：

		Application('/Applications/Mail.app')
4. 通过进程 ID 来调用：

		Application(763)
5. 调用远程机器上的应用程序：
	
		Application('eppc://127.0.0.1/Mail')
6. 调用运行当前脚本的 APP：

		Application.currentApplication()












##语法实例

以下是一些调用对象的语法示例：

1. 获取属性：
	
		Mail.name
2. 获取数组元素：
	
		Mail.outgoingMessages[0]
3. 调用方法：
	
		Mail.open(...)
4. 创建对象：
	
		Mail.OutgoingMessage(...)

##获取属性和设置属性
脚本对象的属性可以作为 JavaScript 属性被点语法调用。像我们前面提到的一样，返回的会是一个对象引用而不是一个真的对象。 如果要获取一个实体的值，可以通过`get`方法来获取，将`get`属性作为方法调用：
	
	subject = Mail.inbox.messages[0].subject()
	
同理我们也可以这样设置一个对象的数据：

	Mail.outgoingMessages[0].subject = 'Hello world'

假设我们要取出邮箱(Mail)应用下的收件箱里所有信息的主题，就是从信息这个数组里面取出每一个元素的subject属性，那么我们可以这样做：
	
	subjects = Mail.inbox.messages.subject()
	
[id]: "arrayData"

##元素数组

可以通过调用指定的方法或者方括号的方式来访问数组的元素，返回的对象也是一个带有属性和值的引用对象，对于一个数组元素可以用以下方法来访问到它的值：

1. 序数：

		window = Mail.windows.at(0)
		window = Mail.windows[0]

2. 名称：

		window = Mail.windows.byName('New Message')
		window = Mail.windows['New Message']

3. ID：

		window = Mail.windows.byId(412)

**注意**: 用 ID 来读取数组中的元素并不是用方括号，而是使用圆括号。

##过滤数组
要从一个数组中过滤出一些特定的元素，应该用数组的 `whose` 方法。 语法如下：
	
	someElementArray.whose({...})
传入一个对象作为参数，这个对象将作为数组筛选的标准，比如：

1. 准确匹配某一个值：
	
		
		{ name: 'JavaScript for Automation' }
		{ name: { _equals: 'JavaScript for Automation' } }
		{ name: { '=': 'JavaScript for Automation' } }

2. 匹配包含某一个值的元素：

		{ name: { _contains: 'script for Auto' } }
3. 匹配以某一个值开头的元素：

		{ name: { _beginsWith: 'JavaScript' } }
4. 匹配以某一个值结尾的元素：

		{ name: { _endsWith: 'Automation' } }
5. 大于某个值：

		{ size: { _greaterThan: 20 } }
		{ size: { '>': 20 } }

6. 大于或等于某个值：

		{ size: { _greaterThanEquals: 20 } }
		{ size: { '>=': 20 } }

7. 小于某个值：

		{ size: { _lessThan: 20 } }
		{ size: { '<': 20 } }
8. 小于等于某个值：
	
		{ size: { _lessThanEquals: 20 } }
		{ size: { '<=': 20 } }
9. 子元素的值匹配：

		{ _match: [ ObjectSpecifier.tabs[0].name, 'Apple' ] }


**注意**: _match 必须对应一个数组作为值。

可以用自由组合这些筛选条件，与或非三种逻辑都支持。

1. 与逻辑：

		{ _and: [
			{ name: 'Apple' },
			{ size: { '<': 20 } },
		]}
注意: 拥有多个属性的对象，也会以与逻辑遍历子属性，均为真才会被选中。

再次注意:  _and 数组需要至少两个元素。

2. 或逻辑：

		{ _or: [
			{ name: 'Apple' },
			{ name: { _contains: 'JavaScript' } },
		]}
注意: _or 数组至少要有两个元素。

3. 非逻辑：

		{ _not: [
			{ name: 'Apple' },
			{ name: { _contains: 'JavaScript' } },
		]}

注意:  _not 数组至少拥有一个元素。

_and, _or, 和 _not 这三个键的对应值只能是数组。

再举一个复杂点的例子，这段代码要从你的收件箱中找出主题包含"Javascript"或者"Automation"的邮件:

	Mail = Application('Mail')
	Mail.inbox.messages.whose({
		_or: [
			{ subject: { _contains: "JavaScript" } }
			{ subject: { _contains: "Automation" } }
		]
	})

##调用命令

命令可以像函数一样被调用。一些命令可以直接传入参数，一些命令则可以支持命名参数（命名参数其实就是一个键值对的对象）。如果命令可以直接传入参数，你可以传入第二个参数作为命名参数。而如果命令不支持直接传入参数，则命名参数只能作为唯一一个参数被传入。如果命令传入的参数是可选的，你可以不传入任何东西或者传入一个null作为第一个参数，再传入一个命名参数作为第二个参数。

1. 无参数的命令：
	
		message.open()
2. 传入直接参数的命令：

		Mail.open(message)
3. 传入命名参数的命令：

		response = message.reply({
			replayAll: true,
			openingWindow: false
		})

4. 同时传入直接参数和命名参数的命令：

		Safari.doJavaScript('alert("Hello world")', {
			in: Safari.windows[0].tabs[0]
		})

##创建对象
你可以通过调用类的构造函数来创建一个对象，也可以选择传入键值对形式的对象来作为参数构造新的对象，同时传入第二个参数来作为新对象创建时所需要的数据。一旦创建完对象你就可以使用对象的 `make` 方法或者将数据数组压入对象，这样对象才会真正在 `Application` 中存在， 在调用 `make` 方法或者把数组压入应用程序之前该对象并不会在应用中存在。

1. 创建一个新对象：
	
		message = Mail.OutgoingMessage().make()
2. 创建一个有属性的新对象：

		message = Mail.OutgoingMessage({
			subject: 'Hello world',
			visible: true
		})
		Mail.outgoingMessages.push(message)

3. 创建一个有数据的新对象：

		para = TextEdit.Paragraph({}, 'Some text')
		TextEdit.documents[0].paragraphs.push(para)
	
一旦对象在应用程序中被创建 (无论是调用`make`方法还是推入了对应的数组),它都能直接和应用中的其他对象进行数据交换。

	message = Mail.OutgoingMessage().make()
	message.subject = 'Hello world'

**注意**: 类的构建函数必须首字母大写。

##添加脚本

添加脚本能够增强拓展应用的功能，OS 系统有一系列的可用的标准脚本，提供了像阅读文本和用户对话框这样的功能。应用必须将`includeStandardAdditions` 标记设置为 `true` 才能使用这些功能。


	app = Application.currentApplication()
	app.includeStandardAdditions = true
	app.say('Hello world')
	app.displayDialog('Please enter your email address', {
		withTitle: 'Email',
		defaultAnswer: 'your_email@site.com'
	})

##Strict 标记

你可以用`strictPropertyScope`,`strictCommandScope`和`strictParameterType`控制自动化程序对Javascript的严谨程度的要求：

1. 用于寻找属性所属的对象：

		app.strictPropertyScope = true

	如果`foo`类有一个`bar`属性，`Baz`类则没有，如果有实例`app`继承自两个类，当`strictPropertyScope`为`false`的时候，如果在实例中找不到对应的属性会自己往父类搜寻是否有相应的属性。

		app.baz[0].bar()
	
2. 寻找对象返回值：

		app.strictCommandScope = true

	如果一个应用有一个`foo`方法，但是`Bar`类没有该方法，当`strictPropertyScope`为`false`的时候，如果在实例中找不到对应的方法会自己往父类搜寻是否有相应的方法。

		app.bar[0].foo()

3. 检查命令中传入的参数的类型：
	
		app.strictParameterType = true

比如, 系统事件里的`keystroke`的第一个参数是`String`类型，如果你把 `application` 属性的`strictParameterType`值设置为`false`，那么你就可以传入一个`Number`类型的值也不会使得系统报错。

`strictPropertyScope` 和 `strictCommandScope` 的默认值为 `false` ，而 `strictParameterType` 的默认值为 `true`。

##Timeout, Considering, 和 Ignoring

你可以在获取和设置属性的时候传入一个事件指向器，在对象上调用命令或者在应用中创建新对象的时候传入一个事件修饰符。这个事件修饰符应该作为调用函数的最后一个参数传入。比如:

执行一条加上timeout的命令：

	app = Application.currentApplication()
	app.includeStandardAdditions = true
	app.displayAlert('This will timeout in 30 seconds', { timeout: 30 })


使得一个`whose`方法忽略大小写，但是考虑空格和连字符。

	app.documents.whose({
		text: { _contains: 'custom-built automation' }
	}, {
		ignoring: 'case',
		considering: ['white space', 'hyphens']
	})
	
可以考虑或者忽略的选项：

* case
* diacriticals
* white space
* hypens
* punctuation
* numeric strings
* application responses
* ObjectSpecifier

`ObjectSpecifier`对象可以用于下面这些场景:


1. 检查一个对象是否对象指引：

		Mail.inbox.messages[0] instanceof ObjectSpecifier == true
2. 获取一个对象指引的类：

		Mail = Application('Mail')
		ObjectSpecifier.classOf(Mail.inbox)

3. 用`whose`建立一个随机的对象指引链：

		firstTabsName = ObjectSpecifier.tabs[0].name
		app.windows.whose({
			_match: [firstTabsName, 'Apple']
		})






















##Paths
当你需要在 TextEdit 中操作文档时你会需要的可能不仅仅是一个路径，而是一整个`Path`对象，这时你需要用`Path`构造函数来将路径实例化。

	TextEdit = Application('TextEdit')
	path = Path('/Users/username/Desktop/foo.rtf')
	TextEdit.open(path)
注意: 调用 toString 方法就可以获取到 Path 对象的对应的路径字符串。

##Progress
可以通过操作脚本的 Progress 对象来操作脚本运行过程中呈现的 UI 。 如下：
设置进度总数：

	Progress.totalUnitCount = 10
设置已完成的进度：

	Progress.completedUnitCount = 1
描述：

	Progress.description = 'Processing Files'
**注意**：设置 progress 对象的描述会替代掉默认的 “运行中…" 这个提示。
**补充描述**：

	Progress.additionalDescription = 'This may take a little while'
**注意**：设置 progress 对象的补充描述意味着进度的总量和已完成的进度都会被显示出来。
##类库
你可以使用储存在 ~/Library/Script Libraries/ 这个脚本库的脚本来更方便快捷达成目的，比如你可以调用 “toolbox.scpt" 这个库文件:

	function log(message) {
	    TextEdit = Application('TextEdit')
    	doc = TextEdit.documents['Log.rtf']
    	doc.text = message
	}

调用之后你可以这么写:

	toolbox = Library('toolbox')
	toolbox.log('Hello world')

**注意**: 代码库里任何暴露在函数之外的代码都会在调用的时候立刻被执行。

##Applets(小应用程序)

你可以通过在脚本编辑器写出脚本并保存为应用程序，这样你就相当于创造了一个独立的，可双击运行的应用程序。小应用程序内置了以下这些处理方法：

程序何时运行:

	function run() {...}

**注意**:  任何暴露在函数之外的代码都会在应用程序启动的时候立刻被执行.

当你使用 Applets(小应用程序) 来打开一个文档时 (文档被小程序读取时):

	function openDocuments(docs) {...}

小程序打印文档时：

	function printDocuments(docs) {...}
注意: `openDocuments` 和 `printDocuments` 方法的参数必须是一个包含文件路径的字符串数组。

可以使用空闲处理函数来做一些定期执行的任务：

	function idle() {...}
	
当小应用程序重新打开的时候：

	function reopen() {...}
当小应用程序退出的时候:

	function quit() {...}

注意: 唯一让小应用程序不退出的方法是在`quit`函数中返回`false`值。没有定义`quit`函数或者返回其他任何不是`false`的值都会让小程序正常退出。

##UI Automation

你可以使用系统应用程序来编写应用的接口脚本。在脚本编辑器中的脚本字典查看系统事件(更具体地说是对应的具体进程的Suite)来看看具体是如何调用应用的事件。

举个例子，现在要编写一段UI脚本来在备忘录中创建一条新笔记：

	Notes = Application('Notes')

	Notes.activate()

	delay(1)
	SystemEvents = Application('System Events')
	Notes = SystemEvents.processes['Notes']

	Notes.windows[0].splitterGroups[0].groups[1].groups[0].buttons[0].click()
##Objective-C Bridge

系统层级运行的Javascript有一个内建的Objective-C桥接器，通过这个桥接器可以提供一些比较强大的功能，比如访问文件系统和创建 Cocoa 应用程序。

对于Objective-C桥接器最主要的使用方式是访问全局变量 `ObjC` 和 `$`.

###框架

默认情况下基础框架的标识符都是可用的。你可以使用`ObjC.import()`这个方法引用更多的库和框架。引入框架时需要参考 BridgeSupport 这个文件。 比如,` NSBeep()` 函数不是基础框架中的函数，那么你可以引入`Cocoa`框架然后就可以使用这个函数了:


	ObjC.import('Cocoa')

	$.NSBeep()

除了系统框架，其他系统级别的库文件中的功能也已经可以直接通过头文件（不用 .h 后缀）声明即可使用，包括： `arpa/inet`, `asl`, `copyfile`, `dispatch`, `dyld`, `errno`, `getopt`,`glob`,`grp`, `ifaddrs`, `launch`, `membership`, `netdb`, `netinet/in`, `notify`, `objc`, `paths`, `pwd`, `readline`, `removefile`, `signal`, `spawn`, `sqlite3`, `stdio`, `stdlib`, `string`, `sys/fcntl`, `sys/file`, `sys/ioctl`, `sys/mount`, `sys/param`, `sys/resource`, `sys/socket`, `sys/stat`, `sys/sysctl`,`sys/time`, `sys/times`, `sys/types`, `sys/wait`, `sys/xattr`, `syslog`, `time`, `unistd`, `uuid/uuid`, `vImage`, `vecLib`, `vmnet`, `xpc`, 和 `zlib`.

除了内建的框架和库，你也可以引入任何支持通过传入路径来完成桥接的框架。举个例子，如果你有一个 框架，可以通过 `/Library/Frameworks/Awesome.framework` 来进行引用。那么你可以这么做来引入你的框架：

	ObjC.import('/Library/Frameworks/Awesome.framework')
	
###数据类型

原始的 Javascript 数据类型都会被转换为对应的C语言数据类型。比如一个 Javascript 字符串就会被转化为 char * 类型，而Javascript中的整型就会转化为对应的 int。 当你使用了一个返回值为 char *的 ObjC API，你会获得的返回值其实是一个JS字符串。注意：长整型(Long)的数字会转化为Javascript中的字符串。

当你把一个 JavaScript 的原始数据类型的值传入到一个 ObjC 的方法时，这个值会先被自动转化为方法所期望的对应的 ObjC 对象类型。 比如, 如果一个方法的期望传入参数是一个 NSString 值，而你传入了一个 JS String 对象，那么这个对象就会被转换为 NSString 。

无论如何请千万记得 ObjC 方法不会自动将返回的 ObjC 对象类型转换为 Javascript 原始类型值。

###类的实例化和调用方法

所有的类都是作为`$`对象的一个属性值被定义的。ObjC对象的方法可以根据是否传入参数被下列两种方法其中一种调用

如果ObjC方法不带参数，则可以用这个名字作为Javascript属性来调用它。比如下面这句代码就实例化了一个可变的空字符串：

	str = $.NSMutableString.alloc.init

如果一个 ObjC 方法需要传参调用，你可以根据`JSExport`约定的命名的Javascript方法(函数类型的属性值)来调用它；命名规范就是冒号后的字母必须大写，然后移除掉所有的冒号；比如，以下代码将Javascript `string` 类型转换为 `NSString`类型并写入到一个文件里：
If the ObjC method does take arguments, then you invoke it by calling the JavaScript method (function-typed property) named according to the  convention; the letter following each ":" is capitalized, and then the ":"s are removed. For instance, this instantiates an  from a JavaScript string and writes it to a file:


	str = $.NSString.alloc.initWithUTF8String('foo')

	str.writeToFileAtomically('/tmp/foo', true)

如果你调用一个类似`intValue`这样的方法，返回的是一个C的数据类型而不是一个对象，然后你会获取到的返回值其实是一个原始的Javascript数据类型。举个例子，下面的代码返回的是原始的Javascript数据整型，99。

	$.NSNumber.numberWithInt(99).intValue
###访问 ObjC 属性

就像调用无参方法一样，ObjC的属性也能通过Javascript属性来调用。

访问一个桥接对象的属性时，ObjC属性列表会被首先查询到。如果该名称的属性存在，那么该属性相应的getter或setter会被调用。而如果类中对应的ObjC属性名称不存在，那么这个属性名称会被当做方法选择器来使用。

如果使用一个自定义的名字来定义一个属性，你可以使用属性名或者自定义名来获取这个属性：


	task = $.NSTask.alloc.init

	task.running == task.isRunning
同理, 不像无参函数,桥接对象属性(假设该属性可以读写)会自动映射为对应的ObjC属性。接下来这两行代码作用是一样的，因为`launchPath` 已经作为ObjC属性被定义好了:

	task.launchPath = '/bin/sleep'

	task.setLaunchPath('/bin/sleep')
###Wrapping and unwrapping

`ObjC.wrap()` 方法可以用于将一个原始的Javascript数据类型转换为ObjC数据类型，并且用一个Javascript包裹器包裹住这个ObjcC对象。

`ObjC.unwrap()` 方法可以将一个被包裹的`ObjC`对象转化为Javascript原始数据类型。

`ObjC.deepUnwrap()` 方法能递归转换将被包裹的Objc对象中所有的子对象转换为JavaScript原始数据类型。

`$()` 操作符是 `ObjC.wrap()` 的一个语法糖,和ObjC中的`@()`运算法作用一致。比如` $("foo")`就会返回一个被包裹的`NSString`实例，而 $(1) 会返回一个被包裹的`NSNumber`实例。

ObjC对象的 `.js` 属性是 `ObjC.unwrap()` 的一个语法糖，比如 `$("foo").js` 会返回 "foo".

###常量和枚举
常量和枚举同样是被定义为 `$` 对象属性之一:

	$.NSNotFound

	$.NSFileReadNoSuchFileError

	$.NSZeroRect
###函数
函数也是`$`对象的属性之一：

	$.NSBeep()

	$.NSMakeSize(33, 55)
###可变参数

每一个函数的可变参数都要和后续声明的类型一致。比如`printf`函数必须在格式化字符串这个参数后面加上一个`%s`类型的参数。


	ObjC.import('stdio')

	$.printf('%s %s %s', 'JavaScript', 'for', 'Automation')
`NSLog` 有一个最后的声明参数 has a final declared parameter type of %@ and must receive %@ arguments after the initial format string.


	$.NSLog('%@ %@ %@', $('JavaScript'), $('for'), $('Automation'))

当你用`arrayWithObjects`创建一个`NSArray`对象时，终止应该由控制器来做而不是由你来自己终止这个程序。

	$.NSArray.arrayWithObjects($('JavaScript'), $('for'), $('Automation'))
###Structs

Structs are represented as generic JavaScript objects, where the struct field names are the property names:
结构是标准的Javascript对象，结构区域的名称就是属性名称：

	$.NSMakeRect(0, 0, 1024, 768)
运行结果: {"origin":{"x":0,"y":0},"size":{"width":1024,"height":768}}

###Nil

在 ObjC, nil 是一个有效的消息对象.向`nil`对象传递信息仍然会返回一个`nil`对象。这个和Javascript不同，Javascript允许不允许`undefined`调用方法。

你可以不传参调用下面这个函数就能得到一个Javascript意义上的`nil`对象。

	nil = $()

如果桥接对象是被一个`nil`指针包裹着，你可以调用`isNil()`方法：

	if (o.isNil()) { ... }
	
###隐形引用传参

能够作为一个参数传入需要传入引用参数的ObjC方法。运行环境会自动将这个内部的空指针换成引用的指针。

	error = $()
	fm = $.NSFileManager.defaultManager
	fm.attributesOfItemAtPathError('/DOES_NOT_EXIST', error)
	if (error.code == $.NSFileReadNoSuchFileError) {
		// ...
	}
###显性引用传参

`Ref` 类传入参数如果是ObjC的话则传入的参数需要是一个引用。在`Ref`函数中传入`0`就能够获取到引用的值。

	function testExplicitPassByReference(path) {
		ref = Ref()
		fm = $.NSFileManager.defaultManager
		exists = fm.fileExistsAtPathIsDirectory(path, ref)
		if (exists) {
			isDirectory = ref[0]
			return (path + ' is ' + (isDirectory ? '' : 'not ') + 'a directory; ')
		}else {
	
			return (path + ' does not exist\n')
		}
	}

	output = testExplicitPassByReference('/System') 
	+ testExplicitPassByReference('/mach_kernel')
	+ testExplicitPassByReference('/foo')
	
输出结果: /System is a directory; /mach_kernel is not a directory; /foo does not exist

###Ref

获取到`Ref`对象的字节值，`Ref`对象的值能够像数组一样被排序：

	ref = Ref('id*')

	ref[0] = $('foo')

	if (ref[0] == $('foo')) { ... }

当你需要传递一个不透明的指针，并检查是否有不透明的指针返回， 你可以实例化一个空的`Ref`对象并使用`equals`方法，如下所示

	CONTEXT = Ref('void');
	ObjC.registerSubclass({
		name: 'MyObject',
		properties: { foo: 'id' },
		methods: {
			'startObserving': {
				types: ['void', []],
				implementation: function (obj) {
					this.addObserverForKeyPathOptionsContext(this,'foo',($.NSKeyValueObservingOptionNew | $.NSKeyValueObservingOptionOld),CONTEXT);
				},
			},
			'observeValueForKeyPath:ofObject:change:context:': {
				types: ['void', ['id', 'id', 'id', 'void *']],
				implementation: function(keyPath, observedObject, change, context) {
					$.NSLog(Automation.getDisplayString(Ref.equals(context, CONTEXT)))
				}
			}
		}
	})

###绑定函数
引入框架时，只有 BridgeSupport 中特别指定的 C 函数才会生效。 调用其他 APP 运行环境下的 C 函数 , 脚本能够将这些函数绑定到Javascript运行环境中。如果你想这么做，你必须特别制定函数的返回类型和参数类型：

	ObjC.bindFunction('pow', ['double', ['double', 'double']])

	$.pow(2,10)

运行结果: 1024


###类型
可以特别指定某些属性，方法，函数参数或者返回值为某个类型的值，以下是可以传入作为类型的字符串标示 ： `void`, `bool`, `float`, `double`, `unsigned char`,` char`,` unsigned short`,` short`, `unsigned int`, `int`, `unsigned long`, `long`, `long double`, `class`, `selector`,`string`, `pointer`, `block`, `function` 和 `id`.
注意: 结构化类型的名字，比如 `NSRect`, 是被允许的, 但诸如 `id` 这样的must be used for objects, not actual class names.
###子类
Javascript 可以注册实现 Objective-C 对象的子类：

	ObjC.registerSubclass({
		name: 'AppDelegate',
		superclass: 'NSObject',
		protocols: ['NSApplicationDelegate'],
		properties: {
			window: 'id'
		},
		methods: {
			'applicationDidFinishLaunching:': function (notification) {
				$.NSLog('Application finished launching');
			}
		}
	})
如果没有任何父类，对象会自动继承自 NSObject 这个父类. 当你指定一个重写的方法的类型为可选，但是如果类型已经存在了，那么他们就必须和父类的方法一致。
这个一个常规的 `init` 初始方法和一个声明了类型的常规方法：

	ObjC.registerSubclass({
		name: 'MyClass',
		properties: {
			foo: 'id'
		},
		methods: {
			init: function () {
				var _this = ObjC.super(this).init;
				if (_this != undefined) {
				_this.foo = 'bar'
				}
				return _this
			},
			baz: {
				types: ['void', []],
				implementation: function () {
					this.foo = 'quux'
				}
			}
		}
	})
注意: **千万不要**重写 init(初始化) 方法中 this 变量或者其他属性值。 使用一个和已有变量不重复的变量名（如上面使用的`_this`）, 然后使用这个变量去初始化并返回属性 。在所有其他方法中都会使用到一个标准的`this`变量，在 JavaScript 中就更不应该去重新定义`this`这个变量 . Objective-C 类可以根据不同的调用从初始构造方法中返回一个不同的对象。 

##Automation.getDisplayString()
`Automation` 对象下的 getDisplayString 方法允许你传入一个对象，输出该对象的源文或者转化为可读的字符串。

	foo = $('bar')
	Automation.getDisplayString(foo)
将 true 参数作为第二个参数传入到 getDisplayString 函数中就可以将传入的对象打印为可读的字符串。
	
	Automation.getDisplayString(foo, true)



原文链接 [https://developer.apple.com/library/mac/releasenotes/InterapplicationCommunication/RN-JavaScriptForAutomation/](https://developer.apple.com/library/mac/releasenotes/InterapplicationCommunication/RN-JavaScriptForAutomation/) 