## 附录B ##

### 简介 ###

AspectJ通过将连接点的概念覆盖到现有的Java语义并向Java添加一些新的程序元素来扩展Java：

一个连接点是程序执行过程中明确的一点。 这些包括方法和构造函数的调用，字段访问和其他下文描述。

一个切点选择连接点，并公开这些连接点的执行中上下文的一些值。 有几个基本的切点指示器，其他可以由`pointcut`声明来命名和定义。

一处通知是在切点中每个连接点执行的代码。 通知可以访问切点公开的值。 通知由**before**, **after**和**around**声明定义。

**Inter-type**声明构成了AspectJ的静态横切功能，也就是，通过用新的字段，构造函数或方法添加或扩展接口和类，从而改变改变程序的类型结构的代码。 一些**Inter-type**声明是通过常规方法，字段和构造函数声明的扩展来定义的，其他的声明是使用新的声明关键字`declare`进行的。

一个切面是封装切点，通知和静态横切特征的横切类型。 按类型，我们指Java的概念：一个模块化的代码单元，它有一个定义良好的接口，可以在编译时进行推理。 切面由`aspect`声明定义。

---

### 连接点 ###

虽然切面定义了横切的类型，但AspectJ系统不允许完全任意的横切。 相反，切面定义了在程序执行过程中切入规范点的类型。 这些规范性的点被称为连接点。

一个连接点是程序执行过程中明确的一点。 由AspectJ定义的连接点是：

**Method call（方法调用）**

- 当一个方法被调用时，不包括非静态的super方法调用。

**Method execution（方法执行）**

- 当实际方法的代码体执行时。

**Constructor call（构造函数调用）**

- 当一个对象构建并该对象的初始构造函数被调用（即，不是`"super"`或`"this"`形式的构造函数调用）。正在构造的对象在`constructor call`连接点处返回，因此它的返回类型被认为是该对象的类型，并且可以在`after returning`通知中访问该对象。

**Constructor execution（构造函数执行）**

- 当实际构造函数的代码体执行时，即在其this或者super构造函数调用后执行。正在构造的对象就是当前正在执行的对象，因此可以使用`this`切点来访问。构造函数的`constructor execution`连接点就是构造器包含调用super构造以及封装类的任何非静态初始化。没有值从`constructor execution`连接点返回，所以它的返回类型被认为是`void`。

**Static initializer execution(静态初始化器执行)**

- 当一个类的静态初始化器执行时。没有值从`static initializer execution`连接点返回，所以它的返回类型被认为是`void`。


**Object pre-initialization(对象预初始化)**

- 在特定类的对象初始化代码运行之前。这包含其第一个被调用的构造函数开始和其父构造函数开始之间的时间。因此，这些连接点的执行包含评估`this()`和`super()`构造函数调用的参数的连接点。没有值从`object pre-initialization`连接点返回，所以它的返回类型被认为是`void`。

**Object initialization（对象初始化）**

- 当一个特定类的对象初始化代码运行时。这包含了返回其父构造函数和返回其第一次调用构造函数之间的时间。它包含所有用于创建对象的动态初始化器和构造器。正在构造的对象是当前正在执行的对象，因此可以使用`this`切点来访问。没有值从`Object initialization`连接点返回，所以它的返回类型被认为是`void`。

**Field reference（字段引用）**

- 引用非常量字段时。 [请注意，对常量字段（绑定到常量字符串对象或原始值的`static final`字段）的引用不是连接点，因为Java要求将它们内联。]

**Field set(字段设置)**

- 当一个字段被分配给。`Field set`连接点被认为有一个参数，该字段被设置为的值。没有值从`field set`连接点返回，所以它的返回类型被认为是`void`。 [请注意，初始化常量字段（初始化程序为常量字符串对象或原始值的`static final`字段）不是连接点，因为Java要求将其引用内联。]

**Handler execution(处理程序执行)**

- 当一个异常处理程序执行。 `Handler execution`连接点被认为有一个参数，正在处理异常。 没有值从`Handler execution`连接点返回，所以它的返回类型被认为是`void`。


**Advice execution(通知执行)**

- 当一处通知的代码体执行时。

每个连接点可能有三个与之关联的状态：当前正在执行的对象，目标对象和参数对象数组。 它们分别由三个状态暴露的切点，`this`，`target`和`args`暴露。

非正式地，当前正在执行的对象是在挑选的连接点的`this`表达式对象。 目标对象是在连接点控制或关注被传递的对象。 参数值是在控制或关注传递中传递的那些值。

| Join Point	        | Current Object	    |  Target Object	  |  Arguments |
| --------   | :-----   | :-----   | :----: |
| Method Call        | executing object*      |   target object**   | method arguments |
| Method Execution	        | executing object*	      |   executing object*	    | method arguments  |
| Constructor Call	        | executing object*	      |   None    | constructor arguments |
| Constructor Execution	        | executing object	      |   executing object	    | constructor arguments |
| Static initializer execution	        | None      |   None    | None |
| Object pre-initialization	        | None      |   None    | constructor arguments |
| Object initialization	        | executing object	      |   executing object	    | constructor arguments |
| Field reference	        | executing object*	      |   target object**	    | None |
| Field assignment	        | executing object*	      |   target object**	    | assigned value |
| Handler execution	        | executing object*	      |   executing object*	    | caught exception |
| Advice execution		        | executing aspect	      |   executing aspect    | advice arguments |



>*静态上下文中没有执行对象，例如静态方法或静态初始化程序。

>**没有与静态方法或字段关联的连接点的目标对象。

>### 对应表格中的的*或者** ###


### Pointcuts ###


切点是一个程序元素，用于挑选连接点并从这些连接点的执行上下文中暴露数据。 切点主要由通知使用。 他们可以用布尔运算符来组成其他切点。 该语言提供的基本切点和组合是：

**call(MethodPattern)**

- 选择签名与**MethodPattern**匹配的每个`method call`连接点。

**execution(MethodPattern)**

- 选择签名与**MethodPattern**匹配的每个`method execution`连接点。

**get(FieldPattern)**

- 选择签名与**FieldPattern**匹配的每个`field reference`连接点。 [请注意，对常量字段（绑定到常量字符串对象或原始值的`static final`字段）的引用不是连接点，因为Java要求将它们内联。]

**set(FieldPattern)**

- 选择签名与**FieldPattern**匹配的每个`field set`连接点。 [请注意，初始化常量字段（初始化程序为常量字符串对象或原始值的`static final`字段）不是连接点，因为Java要求将其引用内联。]

**call(ConstructorPattern)**

- 选择签名与**ConstructorPattern**匹配的每个`constructor call`连接点。

**execution(ConstructorPattern)**

- 选择签名与**ConstructorPattern**匹配的每个`constructor execution`连接点。

**initialization(ConstructorPattern)**

- 选择签名与**ConstructorPattern**匹配的每个`object initialization`连接点。

**preinitialization(ConstructorPattern)**

- 选择签名与**ConstructorPattern**匹配的每个`object pre-initialization`连接点。

**staticinitialization(TypePattern)**

- 选择签名与**TypePattern**相匹配的每个`static initializer execution`连接点。

**handler(TypePattern)**

- 选择签名与**TypePattern**匹配的每个`exception handler`程序连接点。

**adviceexecution()**

- 选择所有`advice execution`连接点。

**within(TypePattern)**

- 选择与**TypePattern**类型匹配中执行的代码中的每个连接点。

**withincode(MethodPattern)**

- 选择与**MethodPattern**签名匹配中执行的代码中的每个连接点。

**withincode(ConstructorPattern)**

- 选择与**ConstructorPattern**构造函数匹配中执行的代码的每个连接点。

**cflow(Pointcut)**

- 选择`Pointcut`选取的连接点`P`的控制流中的每个连接点，包括`P`本身。

**cflowbelow(Pointcut)**

- 选择`Pointcut`选取的任何连接点`P`的控制流中的每个连接点，但不包含`P`本身。

**this(Type or Id)**

- 选择当前正在执行的对象（绑定到`this`对象）是`Type`的实例或标识符`Id`的类型（必须在封闭通知或切点定义中绑定的）的每个连接点。不会匹配来自静态上下文的任何连接点。

**target(Type or Id)**

- 选择目标对象（应用了调用或字段操作的对象）是`Type`的实例或标识符`Id`的类型（必须在封闭通知或切点定义中绑定）的每个连接点。不匹配任何静态成员的`calls`，`gets`或`sets`。

**args(Type or Id, ...)**

- 选择参数是适当类型的实例（或使用该类型的标识符形式）的每个连接点。如果参数的静态类型（声明的参数类型或字段类型）与指定的参数类型相同或为其子类型，则会匹配`null`参数。

**PointcutId(TypePattern or Id, ...)**

- 选择由`PointcutId`命名的用户子定义的切点指示器选取的每个连接点。

**if(BooleanExpression)**

- 选择布尔表达式计算结果为true的每个连接点。所使用的布尔表达式只能访问静态成员，切点或通知的参数以及`thisJoinPoint`形式。特别是，它不能调用切面的非静态方法，或者使用`after advice`返回的值或异常。

**!Pointcut**

-选择**Pointcut**未选取的每个连接点。

**Pointcut0 && Pointcut1**

- 选择`Pointcut0`和`Pointcut1`选取的每个连接点。 

**Pointcut0 || Pointcut1**

- 选择`Pointcut0`或`Pointcut1`选取的每个连接点。 。

**( Pointcut )**

- 选择`Pointcut`选取的每个连接点。

#### 切点定义 ####

切点由程序员用`pointcut`声明来定义和命名。

	  pointcut publicIntCall(int i):
	      call(public * *(int)) && args(i);

一个已命名的切点可以在一个类或切面中定义，并被视为发现它的类或切面的成员。 作为成员，它可能具有访问修饰符，如`public`或`private`。

	  class C {
	      pointcut publicCall(int i):
		  call(public * *(int)) && args(i);
	  }
	
	  class D {
	      pointcut myPublicCall(int i):
		  C.publicCall(i) && within(SomeType);
	  }

不是`final`的切点可以被声明为抽象的，并且没有正文定义。 抽象切点只能在抽象切面中声明。

	  abstract aspect A {
	      abstract pointcut publicCall(int i);
	  }

在这种情况下，继承切面可能会覆盖抽象切点。

	 aspect B extends A {
	      pointcut publicCall(int i): call(public Foo.m(int)) && args(i);
	  }

为了完整性，带有声明的切点可以声明为`final`。

虽然命名切点声明看起来有点像方法声明，并且可以在子切面中覆盖，但它们不能被重载。 两个切点在相同的类或切面声明中使用相同的名称命名是错误的。

命名切点的范围是封闭类声明。 这与其他成员的范围不同; 其他成员的范围是封闭的类体。 这意味着下面的代码是合法：

	  aspect B percflow(publicCall()) {
	      pointcut publicCall(): call(public Foo.m(int));
	  }


#### 上下文暴露 ####

切点有一个接口; 他们暴露了他们挑选的连接点的执行上下文的某些部分。 例如，上面的`PublicIntCall`暴露了所有公开的一元整数方法接收的第一个参数。 通过为命名切点和通知提供类型化的形式参数（如Java方法的形式参数）来暴露此上下文。 这些形式参数受名称匹配的约束。

在通知或切点声明的右侧，在某些切点设计器中，允许使用Java标识符来代替类型或类型集合。 允许这样做的切点指示符是`this`, `target`, 和`args`。 在所有这些情况下，使用标识符而不是类型会做两件事。 首先，它根据形式参数的类型选择连接点。 所以切点

	   pointcut intArg(int i): args(i);

挑选一个int（或一个byte，short或char;任何可分配给int的东西）作为参数传递的连接点。 其次，它使得该参数的值可用于封闭的通知或切点。

值也可以从命名切点公开，所以

	  pointcut publicCall(int x): call(public *.*(int)) && intArg(x);
	  pointcut intArg(int i): args(i);

是一种合法的方式来挑选所有对接受int参数的公共方法的调用，并暴露该参数。

这种曝光有一个特例。 暴露一个Object类型的参数也会匹配基本类型的参数，并且暴露该基本类型的“装箱”版本。 所以，

	  pointcut publicCall(): call(public *.*(..)) && args(Object);

将选择所有以其子类型为唯一参数的一元方法（即，不是像`int`这样的基本类型），但

	  pointcut publicCall(Object o): call(public *.*(..)) && args(o);

将选择所有采用任何参数的一元方法：如果参数为`int`，则传递给`advice`的值将是`java.lang.Integer`类型。

原始值的“装箱”基于原始原始类型。 所以在下面的程序中


	  public class InstanceOf {
	
	    public static void main(String[] args) {
	      doInt(5);
	    }
	
	    static void doInt(int i) {  }
	  }
	
	  aspect IntToLong {
	    pointcut el(long l) : 
	        execution(* doInt(..)) && args(l);
	
	    before(Object o) : el(o) {
	         System.out.println(o.getClass());
	    }
	  }

切点将匹配并公开整数参数，但它会将其作为`Integer`而不是`Long`公开。


#### 原始切点 ####

**Method-related pointcuts**

AspectJ提供了两个原始切点指示器，用于捕获方法调用和执行连接点。

- call(MethodPattern)
- execution(MethodPattern)

**Field-related pointcuts**

AspectJ提供了两个原始切点指示器，用于捕获字段参考和设置连接点：

- get(FieldPattern)
- set(FieldPattern)

所有`set`的连接点都被视为具有一个参数，即该字段设置的值，因此在`set`的连接点处，可以使用`args`切点来访问该值。 因此，守护在类型`T`中声明的静态整数变量`x`的一个切面可能被写为


	  aspect GuardedX {
	      static final int MAX_CHANGE = 100;
	      before(int newval): set(static int T.x) && args(newval) {
			  if (Math.abs(newval - T.x) > MAX_CHANGE)
			      throw new RuntimeException();
		      }
	  }


**Object creation-related pointcuts**

AspectJ提供原始切点指示器，用于捕获对象的初始化程序执行连接点。

- call(ConstructorPattern)
- execution(ConstructorPattern)
- initialization(ConstructorPattern)
- preinitialization(ConstructorPattern)

**Class initialization-related pointcuts**

AspectJ提供了一个基本的切点指示器来挑选静态初始化程序执行连接点。

- staticinitialization(TypePattern)

**Exception handler execution-related pointcuts**

AspectJ提供了一个基本的切点指示器来捕获异常处理程序的执行：

- handler(TypePattern)

所有`handler`连接点都被视为具有一个参数，即正在处理的异常的值。 该值可以使用`args`切点来访问。 因此，在处理`FooException`对象之前，可以将其写为

	  aspect NormalizeFooException {
	      before(FooException e): handler(FooException) && args(e) {
		  	e.normalize();
	      }
	  }

**Advice execution-related pointcuts**

AspectJ提供了一个原始切点指示器来捕获通知的执行

- adviceexecution()

例如，这可以用来过滤来自特定切面的通知控制流中的任何连接点。

	  aspect TraceStuff {
	      pointcut myAdvice(): adviceexecution() && within(TraceStuff);
	
	      before(): call(* *(..)) && !cflow(myAdvice) {
		  		// do something
	      }
	  }

**State-based pointcuts**

当一个特定类型的对象正在执行，正在运行或正在被传递时，许多关注点贯穿在动态时间中。 AspectJ提供了在这些时间捕获连接点的原始切点。 这些切点使用其对象的动态类型来挑选连接点。 它们也可能用于暴露用于区分的对象。

- this(Type or Id)
- target(Type or Id)

`this`切点会选择当前正在执行的对象（与`this`绑定的对象）是特定类型的实例的每个连接点。 `target`切点挑选目标对象（方法被调用的对象或访问的字段）是特定类型的实例的每个连接点。 请注意，`target`应该被理解为当前连接点将控制转移到的对象。 这意味着目标对象与方法执行连接点处的当前对象相同，但可能在方法调用连接点处不同。

- args(Type or Id or "..", ...)

`args`切点选择参数是某些类型的实例的每个连接点。 逗号分隔列表中的每个元素都是四件事情之一。 如果它是一个类型名称，那么该位置的参数必须是该类型的一个实例。 如果它是一个标识符，那么该标识符必须被绑定在封闭的通知或切点声明中，因此该位置的参数必须是标识符类型的实例（或者如果标识符被键入到Object，则为任何类型的实例）。 如果它是“*”通配符，那么任何参数都将匹配，如果它是特殊通配符“..”，那么任何数量的参数都将匹配，就像签名模式中一样。 所以切点

	args(int, .., String)

将选择所有第一个参数为`int`且最后一个为`String`的连接点。

**Control flow-based pointcuts**

一些关注点横切该程序的控制流程。 `cflow`和`cflowbelow`原始切点指示器基于控制流来捕获连接点。

- cflow(Pointcut)
- cflowbelow(Pointcut)

`cflow`切点会选择由`Pointcut`挑选的每个连接点`P`（包括`P`本身）的入口和出口之间发生的所有连接点。 因此，它挑选了由`Pointcut`挑选的连接点的控制流中的连接点。

`cflowbelow`切点会挑选`Pointcut`所挑选的每个连接点P的入口和出口之间发生的所有连接点，但不包括`P`本身。 因此，它会在`Pointcut`挑选的连接点的控制流之下选择连接点。

**来自控制流的上下文暴露**

`cflow`和`cflowbelow`切点可以通过包含`this`，`target`和`args`切点来暴露上下文状态。

任何时候访问此类状态时，都会通过匹配的最新控制流进行访问。 因此，即使在许多控制流程中，由下面的程序打印的"`current arg`"也为零。


	class Test {
	    public static void main(String[] args) {
	        fact(5);
	    }
	    static int fact(int x) {
	        if (x == 0) {
	            System.err.println("bottoming out");
	            return 1;
	        }
	        else return x * fact(x - 1);
	    }
	}
	
	aspect A {
	    pointcut entry(int i): call(int fact(int)) && args(i);
	    pointcut writing(): call(void println(String)) && ! within(A);
	    
	    before(int i): writing() && cflow(entry(i)) {
	        System.err.println("Current arg is " + i);
	    }
	}

通过否定控制流切点来暴露此类状态是错误的，例如`!cflowbelow(P)`。

**Program text-based pointcuts**

尽管许多关注点涉及程序的运行时结构，但有些必须处理词法结构。 AspectJ允许切面根据定义相关代码的位置选择连接点。

- within(TypePattern)
- withincode(MethodPattern)
- withincode(ConstructorPattern)


`within`切点挑选的连接点，其代码的执行定义在`TypePattern`中的某个类型的声明中。 这包括该类的类初始化，对象初始化，方法和构造函数执行连接点以及与该类型的语句和表达式关联的任何连接点。 它还包括与类型的嵌套类型中的代码相关联的任何连接点，以及该类型的默认构造函数（如果有的话）。

`withincode`切点挑选每个连接点，其中代码执行是在特定方法或构造函数的声明中定义的。 这包括方法或构造函数执行连接点以及与方法或构造函数的语句和表达式关联的任何连接点。 它还包括与方法或构造函数的本地或匿名类型中的代码关联的所有连接点。

**Expression-based pointcuts**

- if(BooleanExpression)

`if`切点根据动态属性挑选连接点。 它的语法需要一个表达式，该表达式必须计算为布尔值true或false。 在这个表达式中，`thisJoinPoint`对象是可用的。 所以选择所有呼叫连接点的一种（非常低效的）方法是使用切点
  
	if(thisJoinPoint.getKind().equals("call"))

请注意，连接点处的切点表达式组件的评估顺序未定义。 编写有副作用的`if`切点被认为是不好的风格，并且可能导致潜在的混淆甚至改变测试代码运行时间或行为的行为。


#### 签名 ####

连接点的一个非常重要的属性是它的签名，许多AspectJ的切点指示器用它来选择特定的连接点。

**Methods**

与方法关联的连接点通常具有方法签名，包括方法名称，参数类型，返回类型，声明（受检查）异常的类型以及可以调用方法的某些类型（以下称为“限定类型”）。

在`method call`连接点上，签名是一种方法签名，其限定类型是用于访问该方法的静态类型。 这意味着从调用`((Integer)i).toString()`创建的连接点的签名与调用 `((Object)i).toString()`的签名不同，即使i是同一个变量也是如此。

在`method execution`连接点处，签名是一种方法签名，其限定类型是方法的声明类型。


**Fields**

与字段相关联的连接点通常具有字段签名，由字段名称和字段类型组成。 一个`field reference`连接点有这样一个签名，并且没有参数。 `field reference`连接点具有这样的签名，但具有单个参数，其类型与字段类型相同。

**Constructors**

与构造函数关联的连接点通常具有构造函数签名，包括参数类型，声明（受检查）的异常类型和声明类型。

在`constructor call`连接点处，签名是被调用构造函数的构造函数签名。 在`constructor execution`连接点处，签名是当前正在执行的构造函数的构造函数签名。

在对象`initialization`和`pre-initialization`连接点上，签名是开始此初始化的构造函数的构造函数签名：在此类型初始化此对象期间输入的第一个构造函数。

**Others**

在`handler execution`执行连接点处，签名由处理程序处理的异常类型组成。

在`advice execution`连接点处，签名由切面类型，通知的参数类型，返回类型（除了`around`通知以外的都是`void`）以及声明（受检查）异常的类型组成


#### 匹配 ####


`withincode`，`call`，`execution`，`get`和`set`等基本切点指示器都使用签名模式来确定它们描述的连接点。 签名模式是一个或多个连接点签名的抽象描述。 签名模式旨在与声明个别成员和构造函数时所写的相同类型的事物非常接近。

Java中的方法声明包括方法名称，方法参数，返回类型，类似于静态或私有的修饰符以及throws语句，而构造函数声明省略了返回类型并将方法名称替换为类名称。 例如，类`Test`中的特定方法声明的开始可能是

	  class C {
	      public final void foo() throws ArrayOutOfBoundsException { ... }
	  }

在AspectJ中，方法签名模式具有所有这些特征，但大多数元素都可以用通配符来替换。 所以


	call(public final void C.foo() throws ArrayOutOfBoundsException)

选择该方法的调用连接点和调用切点

	  call(public final void *.*() throws ArrayOutOfBoundsException)

选择匹配方法的所有`call`连接点，只要它们没有参数，没有返回值，是`public`和`final`的，并且声明抛出`ArrayOutOfBounds`异常，无论它们的名称的名字还是在哪个类中定义。

定义类型名称（如果不存在）默认为`*`，所以该切点的另一种方式写法是

	  call(public final void *() throws ArrayOutOfBoundsException)

通配符`..`表示零个或多个参数，所以

	  execution(void m(..))


挑选粗为任意数量的参数的名为m的void方法的`execution`连接点

	  execution(void m(.., int))

选择最后一个参数为int类型的名为m的void方法的`execution`连接点。

修饰符也构成签名模式的一部分。 如果一个AspectJ签名模式应该匹配没有特定修饰符的方法，比如所有的非`public`方法，那么适当的修饰符应该用！操作符。 所以，

	withincode(!public void foo())

选择与名为`foo`的非public的void方法中的代码相关联的所有连接点


	 withincode(void foo())


挑选与名为foo的`void`方法中的代码相关联的所有连接点，不管其访问修饰符如何。

方法名称可以包含`*`通配符，表示方法名称中的任意数量的字符。 所以


	  call(int *())

选择所有的`call`连接点为int方法而不管名字，但是

	  call(int get*())

挑选所有`call`连接点为以字符"`get`"开头的`int`方法。

AspectJ将`new`关键字用于构造函数签名模式，而不是使用特定的类名。 因此，定义为抛出`ArithmeticException`的类C的私有空构造函数的`execution`连接点可以这样选择

	  execution(private C.new() throws ArithmeticException)


**Matching based on the declaring type**


签名匹配切点都指定了一个声明类型，但是对于每个连接点签名，含义略有不同，符合Java语义。

当匹配`withincode`，`get`和`set`时，声明类型是包含声明的类。

匹配`method-call`连接点时，声明类型是用于访问方法的静态类型。 一个常见的错误是为作为最初声明类型的子类型的`call`pointcut指定一个声明类型。 例如，给定类

	  class Service implements Runnable {
	    public void run() { ... }
	  } 

紧跟着的切点

	call(void Service.run())

将无法选择代码的连接点

	  ((Runnable) new Service()).run();

指定最初声明的类型是正确的，但会选择任何此类调用（这里是对任何Runnable的`run()`方法的调用）。 在这种情况下，请考虑选择目标类型：

	  call(void run()) && target(Service)

匹配`method executions`连接点时，如果`execution`切点方法签名指定了一个声明类型，那么切点只会匹配在该类型中声明的方法，或者覆盖在该类型中声明或继承的方法的方法。 所以切点


	  execution(public void Middle.*())


选择所有返回void的public无参方法不管是在Middle中声明还是继承来的，即使这些方法在Middle的子类中被重写。 因此，切点将在此代码中为Sub.m()选定为`method-execution`连接点：


	  class Super {
	    protected void m() { ... }
	  }
	  class Middle extends Super {
	  }
	  class Sub extends Middle {
	    public void m() { ... }
	  }


**Matching based on the throws clause**

类型模式可以用来根据它们的throws子句选择方法和构造函数。 这允许以下两种非常通配的切点：


	  pointcut throwsMathlike():
	      // each call to a method with a throws clause containing at least
	      // one exception exception with "Math" in its name.
	      call(* *(..) throws *..*Math*);
	
	  pointcut doesNotThrowMathlike():
	      // each call to a method with a throws clause containing no
	      // exceptions with "Math" in its name.
	      call(* *(..) throws !*..*Math*);

`ThrowsClausePattern`是`ThrowsClausePatternItems`的逗号分隔列表，其中

`ThrowsClausePatternItem`:
	
- `[!]TypeNamePattern`

`ThrowsClausePattern`匹配任何代码成员签名的throws子句。 要匹配，每个`ThrowsClausePatternItem`必须匹配成员的throws子句。 如果任何项目不匹配，则整个模式不匹配。

如果`ThrowsClausePatternItem`以"`！`"开头，那么它匹配特定的throws子句，当且仅当throws子句中指定的所有类型都与TypeNamePattern不匹配。

如果`ThrowsClausePatternItem`不以"`！`"开头，那么它与throws子句匹配当且仅当throws子句中存在类型与TypeNamePattern相匹配。

"`！`"的规则 匹配有一个潜在的令人惊讶的属性，因为这两个切点


- call(* *(..) throws !IOException)
- call(* *(..) throws (!IOException))


在调用时会有不同的匹配

	void m() throws RuntimeException, IOException {}

[1]将不匹配方法m（），因为方法m的throws子句声明它抛出IOException。
[2]将匹配方法m（），因为方法m的throws子句声明它会抛出一个与IOException不匹配的异常，即RuntimeException。


### 类型模式 ###

类型模式是一种选择类型集合的方法，并在只使用一种类型的地方使用它们。 使用类型模式的规则很简单。

**Exact type pattern**

首先，所有类型名称也是类型模式。 所以`Object`，`java.util.HashMap`，`Map.Entry`，`int`都是类型模式。

如果一个类型模式是一个确切的类型 - 如果它不包含通配符 - 那么匹配就像Java中的普通类型查找一样工作：

- 与基本类型（如int）具有相同名称的模式与这些基本类型匹配。
- 通过包名称（如java.util.HashMap）限定的模式与其他包中的类型匹配。
- 不合格的模式（如HashMap）匹配Java正常范围规则解决的类型。 因此，例如，HashMap可能会匹配相同包中的包级别类型或者使用java导入形式导入的类型。 但它不匹配java.util.HashMap，除非该切面在java.util中，或者类型已经被导入。

所以确切的类型模式基于通常的Java范围规则进行匹配。


**Type name patterns**

有一个特殊的类型名称`*`，它也是一个类型模式。 `*`选择所有类型，包括原始类型。 所以

	call(void foo(*))

选择所有的`call`连接点名为foo的方法，并有一个任意类型的参数。

包含两个通配符“*”和“..”的类型名称也是类型模式。 *通配符匹配除“.”以外的零个或多个字符，因此可以在类型具有特定命名约定时使用。 所以

	handler(java.util.*Map)

选择java.util.Map和java.util.java.util.HashMap等类型和

	handler(java.util.*)

挑选以“java.util”开头的所有类型。 并且不再有“.”s，即java.util包中的类型，但不包含内部类型（如java.util.Map.Entry）。

“..”通配符匹配以“.”开头和结尾的任何字符序列，因此可用于挑选任何子包或所有内部类型中的所有类型。 所以


	within(com.xerox..*)


挑选代码位于任何名称以“com.xerox”开头的类型的声明中的所有连接点。

带有通配符的类型模式不依赖于Java通常的作用域规则 - 它们与织机中可用的所有类型匹配，而不仅仅是那些被导入到Aspect的声明文件中的类型。

**Subtype patterns**

使用“+”通配符可以选择某个类型的所有子类型（或一组类型）。 “+”通配符紧跟在类型名称模式之后。 所以，同时

	call(Foo.new())

选择所有构造函数调用连接点，其中构建了完全类型为Foo的实例，

	call(Foo+.new())

挑选Foo的任何子类型（包括Foo本身）的实例构建的所有构造函数调用连接点，并且不太可能

	call(*Handler+.new())

挑选所有构造函数调用连接点，其中构造名称以“Handler”结尾的任何类型的任何子类型的实例。

**Array type patterns**

类型名称模式或子类型模式后面可以跟着一组或多组方括号来创建数组类型模式。 所以`Object[]`是一个数组类型模式， `com.xerox..*[][]`也是如此，`Object+[]`也是如此。

**Type patterns**

类型模式由类型名称模式，子类型模式和数组类型模式构建，并由布尔运算符`&&`，`||`和`！`构造。 所以

	  staticinitialization(Foo || Bar)

选择Foo或Bar以及的静态初始化程序执行连接点


	  call((Foo+ && !Foo).new(..))

当Foo的子类型（而不是Foo本身）被构建时挑选构造函数调用连接点。

#### 模式总结 ####

以下是AspectJ中使用的模式语法的摘要：


	MethodPattern = 
	  [ModifiersPattern] TypePattern 
	        [TypePattern . ] IdPattern (TypePattern | ".." , ... ) 
	        [ throws ThrowsPattern ]
	ConstructorPattern = 
	  [ModifiersPattern ] 
	        [TypePattern . ] new (TypePattern | ".." , ...) 
	        [ throws ThrowsPattern ]
	FieldPattern = 
	  [ModifiersPattern] TypePattern [TypePattern . ] IdPattern
	ThrowsPattern = 
	  [ ! ] TypePattern , ...
	TypePattern = 
	    IdPattern [ + ] [ [] ... ]
	    | ! TypePattern
	    | TypePattern && TypePattern
	    | TypePattern || TypePattern
	    | ( TypePattern )  
	IdPattern =
	  Sequence of characters, possibly with special * and .. wildcards
	ModifiersPattern =
	  [ ! ] JavaModifier  ...


---

### Advice ###

每种通知都是这种形式


	[ strictfp ] AdviceSpec [ throws TypeList ] : Pointcut { Body }

AdviceSpec是其中之一

- before( Formals )
- after( Formals ) returning [ ( Formal ) ]
- after( Formals ) throwing [ ( Formal ) ]
- after( Formals )
- Type around( Formals )

并且其中`Formal`指类似于`Type Variable-Name`的变量绑定，如用于方法参数的变量绑定，而`Formals`指的是`Formal`的逗号分隔列表。

通知定义横切行为。 它是根据切点定义的。 一处通知的代码运行在其切点所选择的每个连接点上。 代码的运行方式取决于通知的类型。

AspectJ支持三种通知。 通知的种类决定了它如何与它所定义的连接点进行交互。 因此，AspectJ将通知分为在连接点之前运行的通知，在连接点之后运行的通知，以及代替（或"`around`"）连接点运行的通知。

虽然在`before`通知相对没有问题，但对`after`通知有三种解释：连接点的执行正常完成后，在抛出异常之后或者在执行上述其中任何一个之后。 AspectJ允许在任何这些情况下提供通知。

	  aspect A {
	      pointcut publicCall(): call(public Object *(..));
	      after() returning (Object o): publicCall() {
		  	System.out.println("Returned normally with " + o);
	      }
	      after() throwing (Exception e): publicCall() {
		  	System.out.println("Threw an exception: " + e);
	      }
	      after(): publicCall(){
		  	System.out.println("Returned or threw an Exception");
	      }
	  }

`after return`通知可能不关心其返回的对象，在这种情况下，它可以这样写

	  after() returning: call(public Object *(..)) {
	      System.out.println("Returned normally");
	  }

如果在`after return`确实暴露了它的返回对象，那么该参数的类型被认为是对通知的类似于`instanceof`的约束：只有当返回值是适当的类型时它才会运行。

如果能够将值指定给那个类型的变量，那么值是适当的类型，在Java中。也就是说，一个`byte`值可以分配给一个`short`参数，反之亦然，`int`可以分配给`float `参数，`boolean`值只能分配给`boolean`参数，而`reference`类型可以通过`instanceof`工作。

有两种特殊情况：如果暴露的值的类型输入到`Object`，那么通知不受该类型的约束:实际返回值被转换为通知中的主体对象类型：`int`值表示为`java.lang.Integer`对象等，并且无返回值（例如来自void方法）被表示为`null`。

其次，如果连接点能够返回类型`T`，则可以将`null`赋给参数`T`.

Around通知取代其运行的连接点，而不是在它之前或之后运行。因为`around`允许返回一个值，所以它必须声明返回类型，就像方法一样。

因此，下面一个around通知的简单使用是使指定方法返回常量:

	  aspect A {
	      int around(): call(int C.foo()) {
		  return 3;
	      }
	  }

然而，在`around`通知中，原始连接点的计算可以用特殊的语法来执行

	proceed( ... )

`proceed`形式将获得来自`around`的切点所暴露的上下文中的参数，并返回无论`around`是否声明返回值。 所以，下面的通知会在foo被调用时将第二个参数加倍，然后减半结果：

	 aspect A {
	      int around(int i): call(int C.foo(Object, int)) && args(i) {
			  int newi = proceed(i*2)
			  return newi/2;
	      }
	  }


如果`around`通知的返回值被输入到`Object`，则`proceed`结果被转换为`object`表示，即使它最初是一个基本类型值。 当通知返回一个`Object`值时，该值将被转换回原来的形式。 因此，另一种编写加倍和减半通知的方法是：

	  aspect A {
	      Object around(int i): call(int C.foo(Object, int)) && args(i) {
			  Integer newi = (Integer) proceed(i*2)
			  return new Integer(newi.intValue() / 2);
	      }
	  }

除非指定切面实例以外的其他目标作为调用的接收方，否则任何在`around`通知内部发生的`proceed(..)`都被视为特殊的`proceed`方式（即使切面定义名为`proceed`的方法）。 例如，在以下程序中，第一个`proceed`调用将被视为对ICanProceed实例的方法调用，而第二个要继续的调用被视为特殊`proceed`形式。

	
	  aspect A {
	     Object around(ICanProceed canProceed) : execution(* *(..)) && this(canProceed) {
	        canProceed.proceed();         // a method call
	        return proceed(canProceed);   // the special proceed form
	     }
	     
	     private Object proceed(ICanProceed canProceed) {
	        // this method cannot be called from inside the body of around advice in
	        // the aspect
	     }
	  }	


在各种通知中，通知的参数的行为与方法参数完全相同。 特别是，分配给任何参数只会影响参数的值，而不会影响参数的值来源。 这意味着

	
	  aspect A {
	      after() returning (int i): call(int C.foo()) {
		  	i = i * 2;
	      }
	  }


不会使通知的返回值增加一倍。 相反，它会使本地参数加倍。 可以通过使用around通知来更改连接点的参数值或返回值。

通过`proceed（..）`，可以通过为变量提供不同的值来更改较先例的通知和基础连接点所使用的值。 例如，此切面将替换名为`pointcut privateData`中的s的字符串：


	 aspect A {
	    Object around(String s): MyPointcuts.privateData(s) {
	      return proceed("private data");
	    }
	  }

如果替换参数以`proceed（..）`，则当参数引用实际类型的超类型并且不提供实际类型的引用时，您可以在运行时导致`ClassCastException`。 在以下切面，`around`通知将声明的目标`List`替换为`ArrayList`。 这是类型匹配后编译时的有效代码。


	  import java.util.*;
	
	  aspect A {
	    Object around(List list): call(* List+.*()) && target(list) {
	      return proceed(new ArrayList());
	    }
	  }

但想象一下一个简单的程序，其中实际的目标是LinkedList。 在这种情况下，通知会在运行时导致`ClassCastException`，并且未在`ArrayLis`t中声明`peek()`。

	  public class Test {
	    public static void main(String[] args) {
	      new LinkedList().peek();
	    }
	  }

即使在看起来不需要的情况下，例如，如果程序更改为在`List`中声明的调用`size()`，`ClassCastException`也会发生：

	  public class Test {
	    public static void main(String[] args) {
	      new LinkedList().size();
	    }
	  }


仍然会有一个`ClassCastException`，因为无法证明运行时二进制兼容更改在`LinkedList`层次结构中的或者需要`LinkedList`的连接点上的某些通知。


**Advice modifiers**

`strictfp`修饰符是通知中允许的唯一修饰符，它的作用是使通知中的所有浮点表达式都是`FP-strict`。

**Advice and checked exceptions**

通知声明必须包含一个throws子句，列出主体内可能抛出的已检查的异常。 此检查的异常列表必须与通知的每个目标连接点兼容，否则编译器会发出错误信号。

例如，在以下声明中：


	  import java.io.FileNotFoundException;
	
	  class C {
	      int i;
	
	      int getI() { return i; }
	  }
	
	  aspect A {
	      before(): get(int C.i) {
		  	throw new FileNotFoundException();
	      }
	      before() throws FileNotFoundException: get(int C.i) {
		  	throw new FileNotFoundException();
	      }
	  }

这两处通知都是非法的。 第一个是因为主体抛出未声明的检查异常，第二个是因为`field get`连接点不能抛出`FileNotFoundException`。

AspectJ中每种连接点可能抛出的异常是：

**method call and execution**

由目标方法的throws语句声明的受检查异常。

**constructor call and execution**

由目标构造函数的throws语句声明的受检查异常。

**field get and set**

没有受检查异常能够从这些连接点中抛出。

**exception handler execution**

目标异常处理程序可能抛出的异常。

**static initializer execution**

没有受检查异常能够从这些连接点中抛出。

**pre-initialization and initialization**

初始化类的所有构造函数的throws语句中的任何异常。

**advice execution**

任何在通知的throws语句中的异常。

#### Advice precedence ####

多条通知可能适用于同一个连接点。 在这种情况下，通知的解决顺序基于通知优先级。

**Determining precedence**

有许多规则决定了一处通知在相同连接点时是否优先于另一处通知。

如果这两条通知定义在不同切面，则有三种情况：

- 如果切面A在`declare precedence`形式中比切面B匹配得早，那么当具体切面A中的所有通知在同一个连接点上时，其优先于具体切面B中的所有通知。
- 否则，如果切面A是切面B的子切面，那么在A中定义的所有通知优先于在B中定义的所有通知。因此，除非`declare precedence`另外指定，否则子切面的通知优先于父切面的通知。
- 否则，如果在两个不同切面定义了两条通知，则没有定义哪一条优先。

如果这两条通知是在同一切面定义的，则有两种情况：

- 如果两者都是`after`通知，则在后定义的优先于先定义的。
- 否则，该切面中先定义的优先后定义的.

这些规则可以导致循环，例如

	  aspect A {
	      before(): execution(void main(String[] args)) {}
	      after():  execution(void main(String[] args)) {}
	      before(): execution(void main(String[] args)) {}
	  }

这种循环会导致编译器发出错误信号。

**优先效应**

在特定的连接点，通知按优先级排列。

一条`around`通知控制着通过调用`proceed`来运行较低优先级的通知。`proceed`调用将运行下一个优先级的通知，或者如果没有进一步的通知，则在连接点下执行计算。

一条`before`通知可以通过抛出一个异常来防止运行较低优先级的通知。但是，如果它正常返回，那么将运行下一个优先级的通知，或者在没有进一步通知的情况下在连接点下的计算。

`after returning`通知运行将运行下一个优先级的通知，或运行连接点下的计算。然后，如果该计算正常返回，通知的主体将运行。

`after throwing`通知运行将运行下一个优先级的通知，或者在没有进一步通知的情况下运行连接点下的计算。然后，如果该计算抛出了适当类型的异常，则通知的主体将运行。

`after`通知运行将运行下一个优先级的通知，或者在没有进一步通知的情况下运行连接点下的计算。然后通知的主体将运行。

**反射访问连接点**


在通知主体内和`if()`切点表达式中可见三个特殊变量：`thisJoinPoint`，`thisJoinPointStaticPart`和`thisEnclosingJoinPointStaticPart`。 每个都绑定到一个对象，该对象封装了通知的当前或封闭连接点的一些上下文。 存在这些变量是因为某些切点可能会挑选非常大的连接点集合。 例如，切点

	pointcut publicCall(): call(public * *(..));

选择许多方法的调用。 然而，这个切点的通知主体可能希望访问特定连接点的方法名称或参数。

`thisJoinPoint`绑定到一个完整的连接点对象。

`thisJoinPointStaticPart`绑定到包含更少信息的连接点对象的一部分，但每次执行通知时都不需要分配内存。 它相当于`thisJoinPoint.getStaticPart()`。

`thisEnclosingJoinPointStaticPart`绑定到包含当前连接点的连接点的静态部分。 通过这种机制，只有这个封闭连接点的静态部分是可用的。

标准`Java`反射使用`java.lang.reflect`层次结构中的对象来构建其反射对象。 同样，`AspectJ`连接点对象在类型层次结构中也有类型。 绑定到`thisJoinPoint`的对象的类型是`org.aspectj.lang.JoinPoint`，而`thisStaticJoinPoint`绑定到接口类型为`org.aspectj.lang.JoinPoint.StaticPart`的对象。

---

### 静态横切 ###

静态横切

通知声明改变了它们横切的类的行为，但不改变它们的静态类型结构。 对于通过类型层次结构的静态结构进行操作的横切关注点，AspectJ提供了类型间成员声明和其他`declare`形式。


#### 类型间成员声明 ####

AspectJ允许切面来声明成员来关联到其他类型上

**类型间的方法声明如下**

- [ Modifiers ] Type OnType.Id(Formals) [ ThrowsClause ] { Body }
- abstract [ Modifiers ] Type OnType.Id(Formals) [ ThrowsClause ] ;


这种声明的效果是使`OnType`支持新的方法。 即使`OnType`是`interface`。 即使这种方法不是`public`和`abstract`。 所以下面是合法的AspectJ代码：

	  interface Iface {}
	
	  aspect A {
	      private void Iface.m() {
		  System.err.println("I'm a private method on an interface");
	      }
	      void worksOnI(Iface iface) {
		  // calling a private method on an interface
		  iface.m();
	      }
	  }

**类型间的构造函数声明如下**

- [ Modifiers ] OnType.new ( Formals ) [ ThrowsClause ] { Body }

这种声明的效果是使`OnType`支持新的构造函数。 这种情况下`OnType`为接口是错误的。

不能使用类型间声明的构造函数为在`OnType`中声明的`final`变量赋值。 此限制显着增加了理解和编译`OnType`类和声明切面的能力。

请注意，在Java语言中，无构造函数的类有一个隐含的无参构造函数，仅调用`super()`。 这意味着尝试在这样的类上声明一个无参数的类型间构造函数可能会导致冲突，即使看起来没有定义构造函数。

**类型间字段声明如下**

- [ Modifiers ] Type OnType . Id = Expression;
- [ Modifiers ] Type OnType . Id;

这种声明的效果是使`OnType`支持新字段。 即使`OnType`是一个接口。 即使该字段既不是`public`，也不是`static`，也不是`final`。

类型间字段声明的初始化程序（如果有）在其目标类中定义的类本地初始化程序之前运行。

在类型间构造函数或方法声明的主体中或在类型间字段声明的初始化程序中，任何出现的标识符`this`都指向`OnType`对象而不是指向切面类型; 从`static`类型间成员声明中访问`this`位置是错误的。

#### 访问修饰符 ####

类型间成员声明可以是public或private，或者具有default（程序包保护的）。 AspectJ不提供受保护的类型间成员。

访问修饰符适用于切面，与目标类型无关。因此，只有在声明切面定义的代码中才能看到私有类型间成员。默认可见性类型间成员只能在声明切面的包中定义的代码中可见。

请注意，声明私有类型间方法（AspectJ支持）与将私有方法声明插入另一个类中非常不同。前者只允许从声明切面访问，而后者只允许从目标类型访问。例如，Java序列化使用私有方法`void writeObject（ObjectOutputStream）`来实现`java.io.Serializable`。这个私有类型间声明的方法不符合此要求，因为它对于该切面是私有的，而不是对于目标类型是私有的。

抽象类型间方法的访问修饰符有一个约束：在public接口上声明一个抽象的非public类型间方法是非法的。这是非法的，因为它会说public接口的约束条件是只有非公开的实现者必须履行。这不符合Java的类型系统。

#### 冲突 ####

类型间声明增加了当地声明的成员和类型间成员之间发生冲突的可能性。 例如，假设`otherPackage`不是包含切面`A`的包，则为代码

	  aspect A {
	      private Registry otherPackage.onType.r;
	      public void otherPackage.onType.register(Registry r) {
		    r.register(this);
		    this.r = r;
	      }
	  }

声明在`otherPackage`中的`onType`有一个字段`r`。 然而，这个字段只能从A切面的代码中访问。这个切面还声明了`onType`有一个"`register`"方法，但是这个方法可以从任何地方访问。

如果`onType`已经定义了私有或者包保护字段"`r`"，则不存在冲突：切面不能看到这样的字段，并且`otherPackage`中的任何代码都不能看到类型间的"`r`"。

如果`onType`定义了公共字段"`r`"，则存在冲突：表达式

	  this.r = r

是一个错误，因为应该使用私有的类型间"`r`"还是公共的本地定义的"`r`"是不明确的。

如果`onType`定义了一个"`register(Registry)`"方法，就会产生冲突，因为任何可以看到"`register(Registry)`"方法适用的定义方法的代码都是不明确的。

根据Java的冲突解决规则，尽可能地解决冲突：


- 一个子类可以从它的超类继承多个字段，全部使用相同的名称和类型。 但是，对字段进行模糊引用是错误的。
- 子类只能从其超类中继承具有相同名称和参数类型的多个方法,如果只有零个或其中一个是具体的（即除了一个其他都是抽象的，或者全部是抽象的）


考虑到不同类型的成员声明之间存在潜在的冲突，如果一切面优先于另一切面，则其声明将在没有来自编译器的任何冲突通知的情况下生效。 当`declare precedence`为优先声明以及子切面优先于其super切面时，情况都是如此。

#### 扩展和实现 ####

一个切面可以通过改变一个类型的超类或者将一个超类接口添加到一个类型上来改变系统的继承层次结构，用`declare  parents`形式。

- declare parents: TypePattern extends Type;
- declare parents: TypePattern implements TypeList;

例如，如果某个切面希望使某个类可运行，它可能会定义适当的类型间`void run()`方法，但它也应该声明该类满足`Runnable`接口。 为了实现`Runnable`接口中的方法，类型间`run()`方法必须是`public`：


	  aspect A {
	      declare parents: SomeClass implements Runnable;
	      public void SomeClass.run() { ... }
	  }

#### 接口和成员 ####


通过使用类型间成员，接口现在可以携带（non-public-static-final）字段和类可以继承的（non-public-abstract）方法。 由于不明确地继承超类和实现多个超级接口中的成员，可能会发生冲突。

因为接口可能携带非静态的初始化器，所以每个接口的行为就好像它有一个包含它的初始化器的零参数构造器。 超级接口实例化的顺序是可观察的。 我们使用以下属性修复此顺序：超类型在子类型之前被初始化，初始化的代码只运行一次，而类型超类的初始化器在其超级接口的初始化器之前运行。 考虑下面的层次结构，其中{Object，C，D，E}是类，{M，N，O，P，Q}是接口。


	Object  M   O
		 \ / \ /
		  C   N   Q
		   \ /   /
		    D   P
		     \ /
		      E

当新E被实例化时，初始化器按此顺序运行：

    Object M C O N D Q P E


#### 警告和错误 ####

一个切面可以指定永远不应该达到特定的连接点。

- declare error: Pointcut: String;
- declare warning: Pointcut: String;

如果编译器确定可能会到达Pointcut中的一个连接点，那么它将使用`String`为其消息发出错误或警告，如声明的那样。

#### 软化异常 ####

一个切面可以指定一个特定种类的异常，如果连接点处抛出异常，应该绕过Java的常规静态异常检查系统，而是抛出一个org.aspectj.lang.SoftException，它是RuntimeException的子类型，因此不需要声明。

- declare soft: Type: Pointcut;


例如，该切面

	  aspect A {
	      declare soft: Exception: execution(void main(String[] args));
	  }

在执行连接点处，会捕获任何异常并重新抛出包含原始异常的org.aspectj.lang.SoftException。

这与以下的通知作用类似


	  aspect A {
	      void around() execution(void main(String[] args)) {
			  try { proceed(); }
			  catch (Exception e) {
			      throw new org.aspectj.lang.SoftException(e);
			  }
	      }
	  }

除了包装异常外，它还影响Java的静态异常检查机制。

与通知一样，`declare soft`形式在没有被具体切面继承的抽象切面上没有作用。 所以下面的代码不会编译，除非它是用继承该切面的具体切面编译的:


	  abstract aspect A {
	    abstract pointcut softeningPC();
	
	    before() : softeningPC() {     
	      Class.forName("FooClass"); // error:  uncaught ClassNotFoundException
	    }    
	                                                      
	    declare soft : ClassNotFoundException : call(* Class.*(..));
	  }


#### 通知优先级 ####


一个切面可以使用`declare precedence`形式来声明一个在具体切面间的优先级:

- declare precedence : TypePatternList 

这表示如果任何连接点具有来自TypePatternList中某些模式匹配的两个具体切面的通知，则通知的优先级将是列表中的顺序。

在`TypePatternList`中，通配符“`*`”最多只能出现一次，它表示“任何类型不与列表中的任何其他模式匹配”。

例如，(1) 具有作为其名称一部分的安全性的切面的约束应优先于所有其他切面，并且 (2) 日志切面（以及继承它的任何切面）应优先于所有非安全切面 ，可以表示为：

	  declare precedence: *..*Security*, Logging+, *;

再举一个例子，`CountEntry`切面可能想要将当前包中接受`Type`对象的方法的条目计数为第一个参数。 但是，它本应该计算所有条目，即使`DisallowNulls`切面导致抛出异常的条目也是如此。 这可以通过声明`CountEntry`优先于`DisallowNulls`来完成。 该声明可以在任何切面或另一切面排序：


	  aspect Ordering {
	      declare precedence: CountEntry, DisallowNulls;
	  }
	  aspect DisallowNulls {
	      pointcut allTypeMethods(Type obj): call(* *(..)) && args(obj, ..);
	      before(Type obj):  allTypeMethods(obj) {
		  	if (obj == null) throw new RuntimeException();
	      }
	  }
	  aspect CountEntry {
	      pointcut allTypeMethods(Type obj): call(* *(..)) && args(obj, ..);
	      static int count = 0;
	      before():  allTypeMethods(Type) {
		  	count++;
	      }
	  }


#### 各种循环 ####

对于任何切面来说，在一个`decare precedence`中有多个TypePattern匹配是错误的，因此：

	  declare precedence:  A, B, A ;  // error

但是，多次`decare precedence`形式可能在这种循环性是合法的。 例如，这些声明的优先顺序都是完全合法的：

	  declare precedence: B, A;
	  declare precedence: A, B;


而且，只要A和B的通知不共享连接点，符合约束的处于活动状态的系统也可能是合法的。 所以这是一个可以用来强制A和B严格独立的语义。

#### 应用于具体切面 ####

考虑以下库切面：

	  abstract aspect Logging {
	      abstract pointcut logged();
	
	      before(): logged() {
	          System.err.println("thisJoinPoint: " + thisJoinPoint);
	      }
	  }
	
	  abstract aspect MyProfiling {
	      abstract pointcut profiled();
	
	      Object around(): profiled() {
	          long beforeTime = System.currentTimeMillis();
	          try {
	              return proceed();
	          } finally {
	              long afterTime = System.currentTimeMillis();
	              addToProfile(thisJoinPointStaticPart,
	                           afterTime - beforeTime);
	          }
	      }
	      abstract void addToProfile(
	          org.aspectj.JoinPoint.StaticPart jp,
	          long elapsed);
	  }

为了使用任意一个切面，他们必须使用具体切面进行继承，比如MyLogging和MyProfiling。 因为通知只适用于具体切面，所以`declare precedence`形式仅在声明具体切面的优先权时才有效。 所以


	  declare precedence: Logging, Profiling;

没有效果，但两者都有

	  declare precedence: MyLogging, MyProfiling;
  	  
	  declare precedence: Logging+, Profiling+;

是有意义的。

#### 静态可确定的切点 ####

出现在`declare`形式的切点具有一定的限制。 像其他切点一样，这些选择连接点，但它们以静态可确定的方式进行。

因此，这些切点可能不会直接或间接（通过用户定义的切点声明）包含基于动态（运行时）上下文进行区分的切点。 因此，这些切点可能不会被定义为

- cflow
- cflowbelow
- this
- target
- args
- if

所有这些都可以区分运行时信息

---

### Aspects ###

一个切面是由aspect声明定义的横切类型。

#### Aspect 声明 ####

aspect声明与class声明类似，都定义了该类型的类型和实现。它也有一些不同：

**切面实现可以跨越其他类型**

除了常规的Java声明（例如方法和字段）之外，切面声明还可以包含AspectJ声明，例如通知，切点和类型间声明。因此，切面包含可以跨越其他类型（包括由其他切面声明定义的类型）的实现声明。

**切面不直接实例化**

切面不直接用new表达式,cloning或serialization来实例化。切面可能有一个构造函数定义，但如果是这样的话，它必须是无参构造函数，并且不会引发检查异常。

**嵌套切面必须是静态的**

切面可以在包级别定义，也可以定义为嵌套静态切面 - 也就是类，接口或切面的静态成员。如果它不在包级别，则必须使用static关键字定义该切面。本地和匿名切面是不允许的。

#### Aspect 继承 ####

为了支持横切关注的抽象和组合，切面可以和类一样继承。 但，Aspect的继承添加了一些新规则。

**切面可以继承类并实现接口**

一个切面，`abstract`或`concrete`，可以继承一个类，并可以实现一组接口。 继承类不提供用new表达式实例化切面的能力：切面可能仍然只定义空构造函数。

**类可能无法继承切面**

类继承或实现一个切面是错误的。

**切面继承切面**

切面可以继承其他切面，在这种情况下，不仅字段和方法被继承，而且切点也是如此。 但是，各切面只能继承抽象切面。 具体切面继承另一个具体切面是错误的。

#### Aspect 实例 ####


与类表达式不同，切面没有用`new`表达式实例化。 相反，切面实例会在程序中自动创建。 程序可以使用静态方法`aspectOf(..)`来获取对某个切面实例的引用。

由于通知仅在切面实例的上下文中运行，因此切面实例化间接控制了通知运行。

用于确定某个切面实例化的标准是从其父切面继承的。 如果该切面没有父切面，那么默认情况下该切面是单例。 切面如何实例化控制者具体切面类上定义的`aspectOf（..）`方法的形式。


**Singleton Aspects**

- aspect Id { ... }
- aspect Id issingleton() { ... }

默认情况下(或者通过使用修饰符`issingleton()`)，切面只有跨越整个程序一个实例。 该实例在程序执行期间的任何时间都可以从所有具体切面自动定义的静态方法`aspectOf()`中获得 - 因此，在上面的示例中，`A.aspectOf()`将返回A的实例。 此切面实例是在加载切面的类文件时创建的。

因为切面的一个实例存在于程序运行的所有连接点(一旦它的类被加载）,它的通知将有机会在所有这样的连接点上运行。

(实际上，切面A的一个实例是针对切面A的每个版本进行的，因此每个A由不同的类加载器加载时会有一个实例)

**Per-object aspects**

- aspect Id perthis(Pointcut) { ... }
- aspect Id pertarget(Pointcut) { ... }

如果切面A被定义为`perthis（Pointcut）`，则在由Pointcut挑选的连接点处会为为`executing object`（即，`this`）对应的对象创建类型A的一个对象。在A中定义的通知只会在当前正在`executing object`与A实例关联的连接点上运行。

同样，如果切面A被定义为`pertarget(Pointcut)`，则在由Pointcut挑选的连接点处会为`target object`对应的对象创建一个类型为A的对象。 A中定义的通知只会在`target object`与A实例关联的连接点上运行。

在任何一种情况下，都可以使用静态方法调用`A.aspectOf(Object)`来获取（类型A）的对象注册的切面实例。每个切面实例尽可能早地创建，但在达到Pointcut挑选的连接点之前没有类型A的关联切面。

正如实现实现附录中所讨论的，`perthis`和`pertarget`切面可能受到AspectJ编译器代码的影响。

**Per-control-flow aspects**

- aspect Id percflow(Pointcut) { ... }
- aspect Id percflowbelow(Pointcut) { ... }

如果切面A被定义为` percflow(Pointcut)或者percflowbelow(Pointcut)`，那么为`Pointcut`选取的每个连接点控制流创建一个类型为A的对象，或者是输入控制流时，或者是在控制流量。 A中定义的溶质可以在该控制流中或之下的任何连接点运行。 在每个这样的控制流程中，静态方法`A.aspectOf()`将返回一个类型为A的对象。在进入每个这样的控制流程时创建该切面的一个实例。


**Aspect instantiation and advice**

所有通知都在一个切面实例的上下文中运行，但是可以创建在切点上一个通知，该切点挑选一个必须在asopect实例化之前发生的连接点。 例如：
	
	 public class Client
	  {
	      public static void main(String[] args) {
	          Client c = new Client();
	      }
	  }
	
	  aspect Watchcall {
	      pointcut myConstructor(): execution(new(..));
	
	      before(): myConstructor() {
	          System.err.println("Entering Constructor");
	      }
	  }

before通知应该在执行系统中的所有构造函数之前运行。 它必须在Watchcall切面的实例的上下文中运行。 获得这种实例的唯一方法是让Watchcall的默认构造函数执行。 但在执行之前，我们需要运行之前的通知......

在编译时没有检测这种循环的一般方法。 如果通知在它的切面被实例化之前运行，AspectJ将抛出一个`org.aspectj.lang.NoAspectBoundException`。


**Aspect 特权**

- privileged aspect Id { ... }

写入切面的代码在引用类或切面的成员时受制于与Java代码相同的访问控制规则。 因此，例如，在一个切面编写的代码可能不会引用具有默认（程序包保护）可见性的成员，除非该切面是在同一个程序包中定义的。

虽然这些限制适用于很多切面，但可能存在通知或类型间成员需要访问其他类型的私有或受保护资源一些切面。 为了做到这一点，可能会声明`privileged`。 特权切面的代码可以访问所有成员，甚至私人成员。


	  class C {
	      private int i = 0;
	      void incI(int x) { i = i+x; }
	  }
	  privileged aspect A {
	      static final int MAX = 1000;
	      before(int x, C c): call(void C.incI(int)) && target(c) && args(x) {
		  if (c.i+x > MAX) throw new RuntimeException();
	      }
	  }


在这种情况下，如果A没有被声明为`privileged`，则字段引用c.i会导致编译器发出错误信号。

如果一个`privileged`切面可以访问某个特定成员的多个版本，那么那些可以看到它是否没有特权的成员优先。 例如，在代码中

	
	  class C {
	      private int i = 0;
	      void foo() { }
	  }
	  privileged aspect A {
	      private int C.i = 999;
	      before(C c): call(void C.foo()) target(c) {
		  System.out.println(c.i);
	      }
	  }


A的私人类型间字段C.i（初始为999）将在通知主体中被引用，优先于C的私有声明字段，因为即使它没有特权，A也可以访问它自己的类型间字段。

请注意，`privileged`切面可以访问由其他切面所做的私有类型间声明，因为它们被简单地视为这些切面的私有成员。


