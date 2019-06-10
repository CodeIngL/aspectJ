### AspectJ附录A ###

### 切点 ###


- Methods and Constructors
	- call(Signature) 
		- 每次调用对调用站点上与Signature匹配的任何方法或构造函数的每次调用
	- execution(Signature) 
		- 执行（签名）每次执行任何与Signature匹配的方法或构造函数
- 字段
	- get(Signature) 
		- 获取（签名）每一个与签名相匹配的字段的引用
	- set(Signature) 
		- 设置（签名）每个分配到匹配签名的任何字段。指定的值可以使用参数切入点来显示
- 异常处理程序
	- handler(TypePattern) 
		- 处理程序（TypePattern）用于TypePattern中任何Throwable类型的每个异常处理程序。异常值可以使用参数切入点来显示
- 忠告
	- adviceexecution（） 
		- 每次执行任何建议
- 初始化
	- staticinitialization(TypePattern) 
		- 静态初始化（TypePattern）对TypePattern中的任何类型执行静态初始化程序
	- initialization(Signature)
		- 当在类型中调用的第一个构造函数与Signature匹配时，初始化（签名）每次初始化对象，包含从超级构造函数调用返回到第一个构造函数的返回值
	- preinitialization(Signature)
		- 预先初始化（Signature）每当在类型中调用的第一个构造函数与Signature匹配时对对象进行每个预初始化，包含将第一个构造函数输入到对超级构造函数的调用
- 词汇
	- within(TypePattern)
		- 在（TypePattern）内的TypePattern类型中定义的代码的每个连接点内
	- withincode(Signature)
		- 在与Signature匹配的方法或构造函数中定义的代码中添加每个连接点的代码（签名）

- 实例检查和上下文暴露
	- this(Type or Id)
		- 当当前执行的对象是Type或Id的类型的实例时，此（Type或Id）每个连接点
	- target(Type or Id)
		- 当目标执行对象是Type或Id类型的实例时，每个连接点都有一个目标（Type或Id）
	- args(Type or Id, ...)
		- 参数是类型的实例或Id的类型时，参数（类型或Id，...）的每个连接点
- 控制流
	- cflow（Pointcut）
		- 由Pointcut挑选的每个连接点P的控制流中的每个连接点，包括P本身
	- cflowbelow（Pointcut）
		- 每个连接点低于Pointcut选出的每个连接点P的控制流;不包括P本身
- 条件
	- if(Expression)
		- 当布尔表达式为真时的每个连接点
- 组合
	- ! Pointcut
		- 未选取的每个连接点
	- Pointcut0 && Pointcut1
		- 由Pointcut0和Pointcut1选取的每个连接点
	- Pointcut0 || Pointcut1
		- 由Pointcut0或Pointcut1选取的每个连接点
	- ( Pointcut )
		- （切入点）由Pointcut挑出的每个连接点



### 类型模式 ###

类型模式是其中之一

- TypeNamePattern 
	- TypeNamePattern中的所有类型
- SubtypePattern 
	- SubtypePattern中的所有类型，带有+的模式。
- ArrayTypePattern 
	- ArrayTypePattern中的所有类型，包含一个或多个[] s的模式。
- ！TypePattern
	- 不在TypePattern中的所有类型
- TypePattern0 && TypePattern1 
	- TypePattern0和TypePattern1中的所有类型
- TypePattern0 || TypePattern1 
	- TypePattern0或TypePattern1中的所有类型
- （TypePattern）
	- TypePattern中的所有类型

其中TypeNamePattern可以是普通类型名称，通配符*（表示所有类型），也可以是带有嵌入式*和通配符的标识符。

标识符中的嵌入*与任何字符序列匹配，但与包（或内部类型）分隔符“。”不匹配。

标识符中嵌入的..匹配以包（或内部类型）分隔符“。”开始和结束的任何字符序列。



### 忠告 ###

每条建议都是这种形式

	[strictfp] AdviceSpec [throws TypeList]：Pointcut {Body}

AdviceSpec是其中之一

- before( Formals )
	- 在每个连接点之前运行
- after( Formals ) returning [ ( Formal ) ]
	- 在每个正常返回的连接点之后运行。可选的形式可以访问返回的值
- after( Formals ) throwing [ ( Formal ) ]
	- 在投掷Throwable的每个连接点之后运行。如果存在可选的形式，则仅在每个引发正规类型的Throwable的连接点之后运行，并且Formal允许访问Throwable异常值
- after( Formals )
	- 在每个连接点之后运行，而不管它是否正常返回或抛出Throwable
- Type around( Formals )
	- 代替每个连接点运行。连接点可以通过调用proceed来执行，它与around通知采用相同数量和类型的参数。
	
咨询机构内部有三个特殊变量：

- thisJoinPoint
	- 一个org.aspectj.lang.JoinPoint类型的对象，表示通知正在执行的连接点。
- thisJoinPointStaticPart
	- 相当于thisJoinPoint.getStaticPart（），但可能会使用更少的运行时资源。
- thisEnclosingJoinPointStaticPart
	- 动态封闭连接点的静态部分。


### 类型间成员声明 ###

每个类型间成员都是其中之一

- Modifiers ReturnType OnType . Id ( Formals ) [ throws TypeList ] { Body }
	- a method on OnType.
- abstract Modifiers ReturnType OnType . Id ( Formals ) [ throws TypeList ] ;
	- an abstract method on OnType.
- Modifiers OnType . new ( Formals ) [ throws TypeList ] { Body }
	- a constructor on OnType.
- Modifiers Type OnType . Id [ = Expression ] ;
	- a field on OnType.


### 其他声明 ###

- declare parents : TypePattern extends Type ;
	- TypePattern中的类型扩展类型。
- declare parents : TypePattern implements TypeList ;
- TypePattern中的类型实现TypeList中的类型。
- declare warning : Pointcut : String ;
	- 如果Pointcut中的任何连接点可能存在于程序中，则编译器将发出警告字符串。
- declare error : Pointcut : String ;
	- 如果Pointcut中的任何连接点都可能存在于程序中，编译器会发出错误String。
- declare soft : Type : Pointcut ;
	- 在Pointcut挑选的任何连接点处引发的任何Type异常都包装在org.aspectj.lang.SoftException中。
- declare precedence : TypePatternList ;
	- 在应用多条建议的任何连接点处，该连接点处的建议优先级均为TypePatternList顺序。


#### 切面 ####

每个方面都是这种形式

	[ privileged ] Modifiers aspect Id [ extends Type ] [ implements TypeList ] [ PerClause ] { Body }

PerClause定义了方面如何实例化和关联（默认情况下，issingleton（））：


	| PerClause        | Description    |  Accessor  |
    | --------   | -----:   | :----: |
    | [ issingleton() ]        | 这方面的一个实例。 这是默认设置。      |   aspectOf() at all join points    |
    | perthis(Pointcut)        | 一个实例与Pointcut中任何连接点上当前正在执行的对象的每个对象都关联      |   aspectOf(Object) at all join points    |
    | pertarget(Pointcut)        | 一个实例与Pointcut中任何连接点上的目标对象的每个对象都关联。      |   aspectOf(Object) at all join points    |
    | percflow(Pointcut)        | percflow（Pointcut）对于由Pointcut定义的连接点的控制流的每个入口都定义了aspect。      |   aspectOf() at join points in cflow(Pointcut)    |
    | percflowbelow(Pointcut)        | percflowbelow（Pointcut）对于Pointcut定义的连接点下的控制流的每个入口都定义了aspect。      |   aspectOf() at join points in cflowbelow(Pointcut)    |

