### AspectJ语言语义 ###


### 连接点 ###

虽然方面定义了横切的类型，但AspectJ系统不允许完全任意的横切。相反，方面定义了在程序执行过程中切入原则性点的类型。这些原则性的点被称为连接点。

连接点是程序执行过程中明确的一点。 AspectJ定义的连接点是：

- Method call
	- 当一个方法被调用时，不包括非静态方法的超级调用。
- Method execution
	- 当实际方法的代码体执行时。
- Constructor call
	- 构建一个对象并调用该对象的初始构造函数（即，不用于“超级”或“this”构造函数调用）。被构造的对象在构造函数调用连接点返回，所以它的返回类型被认为是对象的类型，并且可以在返回通知后访问对象本身。
- Constructor execution
	- 当实际构造函数的代码体执行时，在其超级构造函数调用之后执行。正在构造的对象是当前正在执行的对象，因此可以使用此切入点来访问。调用超级构造函数的构造函数的构造函数执行连接点还包含封装类的任何非静态初始化方法。没有值从构造函数执行连接点返回，所以它的返回类型被认为是无效的。
- Static initializer execution
	- 当一个类的静态初始化器执行时。没有值从静态初始化器执行连接点返回，所以它的返回类型被认为是无效的。
- Object pre-initialization
	- 在特定类的对象初始化代码运行之前。这包含从其第一个被调用的构造函数开始到其父构造函数开始之间的时间。因此，这些连接点的执行包含评估this（）和super（）构造函数调用的参数的连接点。没有值从对象预初始化连接点返回，所以它的返回类型被认为是无效的。
- Object initialization
	- 当一个特定类的对象初始化代码运行时。这包含了返回其父构造函数和返回其第一个构造函数之间的时间。它包含所有用于创建对象的动态初始化器和构造函数。正在构造的对象是当前正在执行的对象，因此可以使用此切入点来访问。没有值从构造函数执行连接点返回，所以它的返回类型被认为是无效的。
- Field reference
	- 引用非常量字段时。 [请注意，对常量字段（绑定到常量字符串对象或原始值的静态最终字段）的引用不是连接点，因为Java要求将它们内联。]
- Field set
	- 当一个字段被分配给。字段集连接点被认为有一个参数，该字段被设置为的值。没有值从字段集合连接点返回，所以它的返回类型被认为是无效的。 [请注意，初始化常量字段（初始化程序为常量字符串对象或原始值的静态最终字段）不是连接点，因为Java要求将其引用内联。]
- Handler execution
	- 当一个异常处理程序执行。处理程序执行连接点被认为有一个参数，正在处理异常。没有值从字段集合连接点返回，所以它的返回类型被认为是无效的。
- Advice execution
	- 当一条建议的代码体执行时。

每个连接点可能有三个与之关联的状态：当前正在执行的对象，目标对象和参数对象数组。它们分别由三个状态暴露的切入点，this，target和args暴露。

非正式地，当前正在执行的对象是该表达式将在连接点挑选出的对象。目标对象是控制或注意力由连接点转移到的位置。这些论据是为了控制或关注转移而通过的那些值。


	| Join Point        | Current Object	    |  Target Object  | Arguments |
    | --------   | -----:   | ----: | :----: |
    | Method Call	 | executing object*	 | target object**	 | method arguments |
    | Method Execution	 | executing object*	 | executing object*	 | method arguments |
    | Constructor Call	 | executing object*	 | None | constructor arguments |
    | Constructor Execution	 | executing object*	 | executing object	 | constructor arguments |
    | Static initializer execution	 | None | None | None |
    | Object pre-initialization	 | None | None | constructor arguments |
    | Object initialization	 | executing object*	 | executing object	 | constructor arguments |
    | Field reference	 | executing object*	 | target object**	 | None |
    | Field assignment	 | executing object*	 | target object**	 | assigned value |
    | Handler execution	 | executing object*	 | executing object	 | caught exception |
    | Advice execution	 | executing object*	 | executing aspect	 | advice arguments |



>*静态上下文中没有执行对象，例如静态方法体或静态初始化程序。

>**没有与静态方法或字段关联的连接点的目标对象。


### 切点 ###


### 切入点 ###

切入点是一个程序元素，用于挑选连接点并从这些连接点的执行上下文中公开数据。切入点主要由建议使用。他们可以用布尔运算符来组成其他切入点。该语言提供的原始切入点和组合器是：

- call(MethodPattern)
	- 挑出签名与MethodPattern匹配的每个方法调用连接点。
- execution(MethodPattern)
	- 挑出签名与MethodPattern匹配的每个方法执行连接点。
- get（FieldPattern）
	- 挑出签名与FieldPattern匹配的每个字段引用连接点。 [请注意，对常量字段（绑定到常量字符串对象或原始值的静态最终字段）的引用不是连接点，因为Java要求将它们内联。]
- set（FieldPattern）
	- 挑出签名与FieldPattern匹配的每个字段集连接点。 [请注意，初始化常量字段（初始化程序为常量字符串对象或原始值的静态最终字段）不是连接点，因为Java要求将其引用内联。]
- call（ConstructorPattern）
	- 挑出签名与ConstructorPattern匹配的每个构造函数调用连接点。
- execution（ConstructorPattern）
	- 挑出签名与ConstructorPattern匹配的每个构造函数执行连接点。
- initialization（ConstructorPattern）
	- 挑出签名与ConstructorPattern匹配的每个对象初始化连接点。
- preinitialization（ConstructorPattern）
	- 挑出签名与ConstructorPattern匹配的每个对象预初始化连接点。
- staticinitialization（TypePattern）
	- 挑出签名与TypePattern相匹配的每个静态初始化程序执行连接点。
- handler（TypePattern）
	- 挑出签名与TypePattern匹配的每个异常处理程序连接点。
- adviceexecution（）
	- 选出所有建议执行连接点。
- within（TypePattern）
	- 找出每个连接点，其中执行的代码是在TypePattern匹配的类型中定义的。
- withincode（MethodPattern）
	- 挑出每个连接点，其执行代码在签名匹配MethodPattern的方法中定义。
- withincode（ConstructorPattern）
	- 在签名与ConstructorPattern相匹配的构造函数中找出执行代码所在的每个连接点。
- cflow（Pointcut）
	- 挑出Pointcut挑选的任何连接点P的控制流中的每个连接点，包括P本身。
- cflowbelow（Pointcut）
	- 指出Pointcut选取的任何连接点P的控制流中的每个连接点，但不包含P本身。
- this（Type或Id）
	- 找出当前正在执行的对象（绑定到此的对象）是Type的实例或标识符Id的类型（必须在封闭通知或切入点定义中绑定的）的每个连接点。不会匹配来自静态上下文的任何连接点。
- target（Type或Id）
	- 找出目标对象（应用了调用或字段操作的对象）是Type的实例或标识符Id的类型（必须在封闭通知或切入点定义中绑定）的每个连接点， 。不匹配任何调用，获取或静态成员集。
- args（Type或Id，...）
	- 挑出参数是适当类型的实例（或使用该表单的标识符的类型）的每个连接点。如果参数的静态类型（声明的参数类型或字段类型）与指定的参数类型相同或为其子类型，则会匹配null参数。
- PointcutId（TypePattern或Id，...）
	- 挑出由PointcutId命名的用户定义的切入点指示符挑选的每个连接点。
- if（BooleanExpression）
	- 挑出布尔表达式计算结果为true的每个连接点。所使用的布尔表达式只能访问静态成员，包含切入点或通知的参数以及thisJoinPoint形式。特别是，它不能调用方面的非静态方法，或者使用- - - after advice通知的返回值或异常。
- ！Pointcut
	- 选出Pointcut未选取的每个连接点。
- Pointcut0 && Pointcut1
	- 挑出由Pointcut0和Pointcut1挑选的每个连接点。
- Pointcut0 || Pointcut1
	- 挑出每个切入点挑选的每个连接点。 Pointcut0或Pointcut1。
- （Pointcut）
	- 挑出Pointcut挑选的每个连接点。

切入点定义

切入点由程序员用切入点声明来定义和命名。

  pointcut publicIntCall(int i):
      call(public * *(int)) && args(i);

一个已命名的切入点可以在一个类或方面中定义，并被视为发现它的类或方面的成员。 作为成员，它可能具有访问修饰符，如公共或私有访问修饰符。


	class C {
	      pointcut publicCall(int i):
		  call(public * *(int)) && args(i);
	  }

	  class D {
	      pointcut myPublicCall(int i):
		  C.publicCall(i) && within(SomeType);
	  }

不是最终的切入点可以被声明为抽象的，并且没有正文定义。抽象切入点只能在抽象方面内声明。

	  abstract aspect A {
	      abstract pointcut publicCall(int i);
	  }

在这种情况下，扩展方面可能会覆盖抽象切入点。

	  aspect B extends A {
	      pointcut publicCall(int i): call(public Foo.m(int)) && args(i);
	  }

为了完整性，带有声明的切入点可以声明为final。

虽然命名切入点声明看起来有点像方法声明，并且可以在子方面覆盖，但它们不能被重载。两个切入点在相同的类或方面声明中使用相同的名称命名是错误的。

命名切入点的范围是封闭类声明。这与其他成员的范围不同;其他成员的范围是封闭的类体。这意味着下面的代码是合法的：

	aspect B percflow(publicCall()) {
	      pointcut publicCall(): call(public Foo.m(int));
	  }

上下文暴露

切入点有一个接口; 他们暴露了他们挑选的连接点的执行上下文的某些部分。 例如，上面的PublicIntCall暴露了所有公开的一元整数方法接收的第一个参数。 通过为命名切入点和建议提供类型化的形式参数（如Java方法的形式参数）来暴露此上下文。 这些形式参数受名称匹配的约束。

在建议或切入点声明的右侧，在某些切入点指示符中，允许使用Java标识符来代替类型或类型集合。 允许这样做的切入点指示符是这个，目标和参数。 在所有这些情况下，使用标识符而不是类型会做两件事。 首先，它根据形式参数的类型选择连接点。 所以切入点


 	pointcut intArg(int i): args(i);


挑选一个int（或一个byte，short或char;任何可分配给int的东西）作为参数传递的连接点。 其次，它使得该参数的值可用于封闭的建议或切入点。

值也可以从命名切入点公开，所以


	  pointcut publicCall(int x): call(public *.*(int)) && intArg(x);
	  pointcut intArg(int i): args(i);


是一种合法的方式来挑选所有对接受int参数的公共方法的调用，并暴露该参数。

这种曝光有一个特例。 暴露一个Object类型的参数也会匹配原始类型的参数，并且暴露该原语的“盒装”版本。 所以，

  	pointcut publicCall(): call(public *.*(..)) && args(Object);

将挑出所有以其子类型为唯一参数的一元方法（即，不是像int这样的基本类型），但

	pointcut publicCall(Object o): call(public *.*(..)) && args(o);

将挑出所有采用任何参数的一元方法：如果参数为int，则传递给advice的值将是java.lang.Integer类型。

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

切入点将匹配并公开整数参数，但它会将其作为整数而不是长整数公开。


### 原始切入点 ###

方法相关的切入点

AspectJ提供了两个原始切入点指示符，用于捕获方法调用和执行连接点。

- call(MethodPattern)
- execution(MethodPattern)

与场相关的切入点

AspectJ提供了两个原始切入点指示符，用于捕获字段参考和设置连接点：

- get(FieldPattern)
- set(FieldPattern)

所有设置的连接点都被视为具有一个参数，即该字段设置的值，因此在设置的连接点处，可以使用参数切入点来访问该值。 因此，守护在类型T中声明的静态整数变量x的一个方面可能被写为

	 aspect GuardedX {
	      static final int MAX_CHANGE = 100;
	      before(int newval): set(static int T.x) && args(newval) {
		  if (Math.abs(newval - T.x) > MAX_CHANGE)
		      throw new RuntimeException();
	      }
	  }

与对象创建相关的切入点

AspectJ提供原始切入点指示符，用于捕获对象的初始化程序执行连接点。

- call(ConstructorPattern)
- execution(ConstructorPattern)
- initialization(ConstructorPattern)
- preinitialization(ConstructorPattern)


类初始化相关的切入点

AspectJ提供了一个基本的切入点指示符来挑选静态初始化程序执行连接点。

- staticinitialization(TypePattern)


与异常处理程序执行相关的切入点


AspectJ提供了一个基本的切入点指示符来捕获异常处理程序的执行：

- handler(TypePattern)


	  aspect NormalizeFooException {
	      before(FooException e): handler(FooException) && args(e) {
		  e.normalize();
	      }
	  }


建议执行相关的切入点

AspectJ提供了一个原始切入点指示符来捕获建议的执行

	adviceexecution（）

例如，这可以用来过滤来自特定方面的建议控制流中的任何连接点。


	  aspect TraceStuff {
	      pointcut myAdvice(): adviceexecution() && within(TraceStuff);
	
	      before(): call(* *(..)) && !cflow(myAdvice) {
		  // do something
	      }
	  }


基于状态的切入点

当一个特定类型的对象正在执行，正在运行或正在被传递时，许多问题都会在动态时间中出现。 AspectJ提供了在这些时间捕获连接点的原始切入点。这些切入点使用其对象的动态类型来挑选连接点。它们也可能用于暴露用于区分的物体。

- this(Type or Id)
- target(Type or Id)

这个切入点挑出了当前正在执行的对象（绑定到此的对象）是特定类型的实例的每个连接点。目标切入点挑选目标对象（方法被调用的对象或访问的字段）是特定类型的实例的每个连接点。请注意，目标应该被理解为当前连接点将控制转移到的对象。这意味着目标对象与方法执行连接点处的当前对象相同，但可能在方法调用连接点处不同。

- args(Type or Id or "..", ...)

args切入点挑出参数是某些类型的实例的每个连接点。逗号分隔列表中的每个元素都是四件事情之一。如果它是一个类型名称，那么该位置的参数必须是该类型的一个实例。如果它是一个标识符，那么该标识符必须被绑定在封闭的通知或切入点声明中，因此该位置的参数必须是标识符类型的实例（或者如果标识符被键入到对象，则为任何类型的实例） 。如果它是“*”通配符，那么任何参数都将匹配，如果它是特殊通配符“..”，那么任何数量的参数都将匹配，就像签名模式中一样。所以切入点

	args（int，..，String）


将挑出所有第一个参数为int且最后一个为String的连接点。

控制基于流的切入点

一些问题涉及该计划的控制流程。 cflow和cflowbelow原始切入点指示符基于控制流来捕获连接点。

- cflow(Pointcut)
- cflowbelow(Pointcut)


cflow pointcut会挑选出由Pointcut挑选的每个连接点P（包括P本身）的入口和出口之间发生的所有连接点。因此，它挑选了由Pointcut挑选的连接点的控制流中的连接点。

cflowbelow pointcut会挑选Pointcut所挑选的每个连接点P的入口和出口之间发生的所有连接点，但不包括P本身。因此，它会在Pointcut挑选的连接点的控制流之下挑选出连接点。

来自控制流的上下文暴露

cflow和cflowbelow切入点可以通过包含this，target和args切入点来暴露上下文状态。

任何时候访问此类状态时，都会通过匹配的最新控制流进行访问。因此，即使在许多控制流程中，由以下程序打印的“当前参数”也为零。


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


通过取消控制流切入点来暴露此类状态是错误的，例如！ cflowbelow（P）。

编程基于文本的切入点

尽管许多问题涉及程序的运行时结构，但有些必须处理词法结构。 AspectJ允许方面根据定义相关代码的位置挑选出连接点。

- within(TypePattern)
- withincode(MethodPattern)
- withincode(ConstructorPattern)

内部切入点在TypePattern中的某个类型的声明中挑选每个连接点，其中的代码执行是定义的。这包括该类的类初始化，对象初始化，方法和构造函数执行连接点以及与该类型的语句和表达式关联的任何连接点。它还包括与类型的嵌套类型中的代码相关联的任何连接点，以及该类型的默认构造函数（如果有的话）。

内部代码切入点挑选每个连接点，其中代码执行是在特定方法或构造函数的声明中定义的。这包括方法或构造函数执行连接点以及与方法或构造函数的语句和表达式关联的任何连接点。它还包括与方法或构造函数的本地或匿名类型中的代码关联的所有连接点。

基于表达式的切入点

- if(BooleanExpression)

if pointcut根据动态属性挑选连接点。它的语法需要一个表达式，该表达式必须计算为布尔值true或false。在这个表达式中，thisJoinPoint对象是可用的。因此挑选所有呼叫连接点的一种（非常低效的）方法是使用切入点

	if(thisJoinPoint.getKind().equals("call"))

请注意，连接点处的切入点表达式组件的评估顺序未定义。如果有副作用的切入点被认为是不好的样式，并且可能导致潜在的混淆甚至改变测试代码运行时间或行为的行为。


签名

连接点的一个非常重要的属性是它的签名，许多AspectJ的切入点指示符用它来选择特定的连接点。

方法

与方法关联的连接点通常具有方法签名，包括方法名称，参数类型，返回类型，声明（检查）异常的类型以及该方法可以调用的某种类型（以下称为“限定类型” ）。

在方法调用连接点上，签名是一种方法签名，其限定类型是用于访问该方法的静态类型。这意味着从调用（（Integer）i）.toString（）创建的连接点的签名与调用（（Object）i）.toString（）的签名不同，即使我是同一个变量也是如此。

在方法执行连接点处，签名是方法签名，其限定类型是方法的声明类型。

字段

与字段相关联的加入点通常具有字段签名，由字段名称和字段类型组成。一个字段引用连接点有这样的签名，并且没有参数。字段集连接点具有这样的签名，但具有单个参数，其类型与字段类型相同。

构造函数

与构造函数关联的连接点通常具有构造函数签名，包括参数类型，声明（检查）的异常类型和声明类型。

在构造函数调用连接点处，签名是被调用构造函数的构造函数签名。在构造函数执行连接点处，签名是当前正在执行的构造函数的构造函数签名。

在对象初始化和预初始化连接点上，签名是开始此初始化的构造函数的构造函数签名：在此类型初始化此对象期间输入的第一个构造函数。

其他

在处理程序执行连接点处，签名由处理程序处理的异常类型组成。

在通知执行连接点处，签名由方面类型，通知的参数类型，返回类型（除了围绕所有通知以外的所有其他类型）以及声明（检查）异常的类型组成。


匹配

withincode，call，execution，get和set原始切入点指示符都使用签名模式来确定它们描述的连接点。 签名模式是一个或多个连接点签名的抽象描述。 签名模式旨在与声明个别成员和构造函数时所写的相同类型的事物非常接近。

Java中的方法声明包括方法名称，方法参数，返回类型，类似于静态或私有的修饰符以及throws子句，而构造函数声明省略了返回类型并将方法名称替换为类名称。 例如，类Test中的特定方法声明的开始可能是

	
	  class C {
	      public final void foo() throws ArrayOutOfBoundsException { ... }
	  }


在AspectJ中，方法签名模式具有所有这些，但大多数元素都可以用通配符来替换。所以

	  call(public final void C.foo() throws ArrayOutOfBoundsException)

选择该方法的呼叫加入点和切入点

	call(public final void *.*() throws ArrayOutOfBoundsException)


只要它们没有参数，不返回任何值，都是公共和最终的，并且声明抛出ArrayOutOfBounds异常，无论它们的名称是名称还是定义了哪个类，都会挑选所有的方法调用连接点。

定义类型名称（如果不存在）默认为*，所以写入该切入点的另一种方式是

	  call(public final void *() throws ArrayOutOfBoundsException)


通配符..表示零个或多个参数，所以

	  execution(void m(..))


为任意数量的参数的名为m的void方法挑选执行连接点，while

	  execution(void m(.., int))


为最后一个参数为int类型的名为m的void方法挑选执行连接点。

修饰符也构成签名模式的一部分。如果一个AspectJ签名模式应该匹配没有特定修饰符的方法，比如所有的非公开方法，那么适当的修饰符应该用！运营商。所以，

	  withincode(!public void foo())


选择与名为foo的null非公共无效方法中的代码相关的所有连接点

	withincode(void foo())

无论访问修饰符如何，都会挑选与名为foo的null void方法中的代码关联的所有连接点。

无论访问修饰符如何，都会挑选与名为foo的null void方法中的代码关联的所有连接点。

	  call(int *())

挑选出所有的调用连接点到int方法而不管名字，但是

	 call(int *())

将所有调用连接点挑选到方法名以字符“get”开头的int方法。

AspectJ将新关键字用于构造函数签名模式，而不是使用特定的类名。 因此，定义为抛出ArithmeticException的类C的私有空构造函数的执行连接点可以被挑选出来

	  execution(private C.new() throws ArithmeticException)

基于声明类型进行匹配

签名匹配切入点都指定了一个声明类型，但是对于每个连接点签名，含义略有不同，符合Java语义。

当匹配代码中的切入点，获取和设置时，声明类型是包含声明的类。

匹配方法调用连接点时，声明类型是用于访问方法的静态类型。 一个常见的错误是为作为最初声明类型的子类型的调用pointcut指定一个声明类型。 例如，给定班级

	 class Service implements Runnable {
	    public void run() { ... }
	  } 


以下切入点

	  call(void Service.run())


将无法挑出代码的连接点

	  ((Runnable) new Service()).run();


指定最初声明的类型是正确的，但会挑选出任何此类调用（这里是对任何Runnable的run（）方法的调用）。在这种情况下，请考虑选择目标类型：


	call(void run()) && target(Service)

匹配方法执行连接点时，如果执行pointcut方法签名指定了一个声明类型，则切入点只会匹配在该类型中声明的方法，或者覆盖在该类型中声明或继承的方法的方法。所以切入点

	  execution(public void Middle.*())

为返回void的公共方法挑选所有方法执行，并且没有在Middle中声明或继承的参数，即使这些方法在Middle的子类中被重写。因此，切入点将在此代码中为Sub.m（）选择方法执行连接点：

	  class Super {
	    protected void m() { ... }
	  }
	  class Middle extends Super {
	  }
	  class Sub extends Middle {
	    public void m() { ... }
	  }

基于throws子句进行匹配

类型模式可以用来根据它们的throws子句挑选出方法和构造函数。这允许以下两种非常通配的切入点：

	
	  pointcut throwsMathlike():
	      // each call to a method with a throws clause containing at least
	      // one exception exception with "Math" in its name.
	      call(* *(..) throws *..*Math*);
	
	  pointcut doesNotThrowMathlike():
	      // each call to a method with a throws clause containing no
	      // exceptions with "Math" in its name.
	      call(* *(..) throws !*..*Math*);


A ThrowsClausePattern is a comma-separated list of ThrowsClausePatternItems, where

ThrowsClausePatternItem :
[ ! ] TypeNamePattern
A ThrowsClausePattern matches the throws clause of any code member signature. To match, each ThrowsClausePatternItem must match the throws clause of the member in question. If any item doesn't match, then the whole pattern doesn't match.

If a ThrowsClausePatternItem begins with "!", then it matches a particular throws clause if and only if none of the types named in the throws clause is matched by the TypeNamePattern.

If a ThrowsClausePatternItem does not begin with "!", then it matches a throws clause if and only if any of the types named in the throws clause is matched by the TypeNamePattern.

The rule for "!" matching has one potentially surprising property, in that these two pointcuts

call(* *(..) throws !IOException)
call(* *(..) throws (!IOException))
will match differently on calls to
void m() throws RuntimeException, IOException {}
[1] will NOT match the method m(), because method m's throws clause declares that it throws IOException. [2] WILL match the method m(), because method m's throws clause declares the it throws some exception which does not match IOException, i.e. RuntimeException.


[1]将不匹配方法m（），因为方法m的throws子句声明它抛出IOException。 [2]将匹配方法m（），因为方法m的throws子句声明它会抛出一个与IOException不匹配的异常，即RuntimeException。

### 键入模式 ###

类型模式是一种挑选出类型集合的方法，并在只使用一种类型的地方使用它们。使用类型模式的规则很简单。

精确的类型模式

首先，所有类型名称也是类型模式。所以Object，java.util.HashMap，Map.Entry，int都是类型模式。

如果一个类型模式是一个确切的类型 - 如果它不包含通配符 - 那么匹配就像Java中的普通类型查找一样工作：

- 与基本类型（如int）具有相同名称的模式与这些基本类型匹配。
- 通过包名称（如java.util.HashMap）进行限定的模式与其他包中的类型匹配。
- 不合格的模式（如HashMap）匹配Java正常范围规则解决的类型。因此，例如，HashMap可能会匹配相同包中的包级别类型或者使用java导入形式导入的类型。但它不匹配java.util.HashMap，除非该方面在java.util中，或者类型已经被导入。

所以确切的类型模式基于通常的Java范围规则进行匹配。

输入名称模式

有一个特殊的类型名称*，它也是一个类型模式。 *挑选出所有类型，包括原始类型。所以

	 call(void foo(*))

挑选出所有的调用连接点来取消名为foo的方法，并取一个任意类型的参数。

包含两个通配符“*”和“..”的类型名称也是类型模式。 *通配符匹配除“。”以外的零个或多个字符，因此可以在类型具有特定命名约定时使用。所以

	  handler(java.util.*Map)

挑选出java.util.Map和java.util.java.util.HashMap等类型和

	  handler(java.util.*)

挑选以“java.util”开头的所有类型。 并且不再有“。”s，即java.util包中的类型，但不包含内部类型（如java.util.Map.Entry）。


“..”通配符匹配以“。”开头和结尾的任何字符序列，因此可用于挑选任何子包或所有内部类型中的所有类型。 所以

	  within(com.xerox..*)

挑选代码位于任何名称以“com.xerox”开头的类型的声明中的所有连接点。

带有通配符的类型模式不依赖于Java通常的作用域规则 - 它们与织机中可用的所有类型匹配，而不仅仅是那些被导入到Aspect的声明文件中的类型。

子类型模式

使用“+”通配符可以挑选出某个类型的所有子类型（或一组类型）。 “+”通配符紧跟在类型名称模式之后。 所以，同时

	  call(Foo.new())

挑出所有构造函数调用连接点，其中构建了完全类型为Foo的实例，

	  call(Foo+.new())

挑选Foo的任何子类型的实例（包括Foo本身）构建的所有构造函数调用连接点，并且不太可能

	 call(*Handler+.new())

挑选所有构造函数调用连接点，其中构造名称以“Handler”结尾的任何类型的任何子类型的实例。

**数组类型模式**

类型名称模式或子类型模式后面可以跟着一组或多组方括号来创建数组类型模式。 所以Object []是一个数组类型模式，com.xerox .. * [] []也是如此，Object + []也是如此。

**键入模式**

类型模式由类型名称模式，子类型模式和数组类型模式构建，并由布尔运算符&&，||和！构造。 所以

  	staticinitialization(Foo || Bar)

挑选出Foo或Bar以及的静态初始化程序执行连接点

	     call((Foo+ && ! Foo).new(..))

当Foo的子类型（而不是Foo本身）被构建时挑选构造函数调用连接点。

### 模式摘要 ###

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


### 通知 ###

每条建议都是这种形式

	[ strictfp ] AdviceSpec [ throws TypeList ] : Pointcut { Body }

AdviceSpec是其中之一


- before( Formals )
- after( Formals ) returning [ ( Formal ) ]
- after( Formals ) throwing [ ( Formal ) ]
- after( Formals )
- Type around( Formals )


Formal指的是像变量名称那样的变量绑定，类似于用于方法参数的变量绑定，而Formals指的是Formal的逗号分隔列表。

建议定义横切行为。 它是根据切入点定义的。 一条建议的代码运行在其切入点所选择的每个连接点上。 代码的运行方式取决于建议的类型。

AspectJ支持三种建议。 这种建议决定了它如何与它所定义的连接点进行交互。 因此，AspectJ将建议分为在其连接点之前运行的建议，在连接点之后运行的建议，以及代替（或“围绕”）连接点运行的建议。

虽然在建议相对没有问题之前，可以对after建议有三种解释：连接点的执行正常完成后，在抛出异常之后或者在执行任何一个之后。 AspectJ允许在任何这些情况下提供建议。


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

返回建议后可能不关心其返回的对象，在这种情况下，它可能会被写入


	after() returning: call(public Object *(..)) {
	      System.out.println("Returned normally");
	  }


如果在返回之后确实暴露了它的返回对象，那么该参数的类型被认为是对通知的类似于instanceof的约束：只有当返回值是适当的类型时它才会运行。

如果可以将该值指定给该类型的变量，那么值是适当的类型，在Java中。也就是说，一个字节值可以分配给一个短参数，反之则不然，int可以分配给一个浮点参数，布尔值只能分配给布尔参数，而引用类型通过instanceof工作。

有两种特殊情况：如果暴露的值被输入到Object，那么通知不受该类型的约束：实际返回值被转换为通知主体的对象类型：int值表示为java.lang .Integer对象等，并且没有值（例如来自void方法）被表示为null。

其次，如果连接点可以返回类型T的东西，则可以将空值赋给参数T.

周围的建议代替其运行的连接点，而不是在它之前或之后运行。因为around允许返回一个值，所以它必须用返回类型声明，就像方法一样。

因此，围绕建议的简单使用是使特定方法保持不变：


	  aspect A {
	      int around(): call(int C.foo()) {
		  return 3;
	      }
	  }


然而，在周围的建议中，原始连接点的计算可以用特殊的语法来执行


	 proceed( ... )

进行表单将周围的切入点所暴露的上下文作为参数，并返回无论周围是否声明要返回的内容。所以，下面的建议会在foo被调用时将第二个参数加倍，然后减半结果：


	  aspect A {
	      int around(int i): call(int C.foo(Object, int)) && args(i) {
		  int newi = proceed(i*2)
		  return newi/2;
	      }
	  }

如果around通知的返回值被输入到Object，则继续的结果被转换为对象表示，即使它最初是一个原始值。当通知返回一个Object值时，该值将被转换回原来的任何表示形式。因此，另一种编写加倍和减半建议的方法是：


	  aspect A {
	      Object around(int i): call(int C.foo(Object, int)) && args(i) {
		  Integer newi = (Integer) proceed(i*2)
		  return new Integer(newi.intValue() / 2);
	      }
	  }

除非指定方向实例以外的其他目标作为调用的接收方，否则任何在around建议内部发生的进程（..）都被视为特殊的进行方式（即使方面定义名为proceed的方法）。例如，在以下程序中，第一个要继续的调用将被视为对ICanProceed实例的方法调用，而第二个要继续的调用被视为特殊继续的形式。


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

在各种建议中，建议的参数的行为与方法参数完全相同。特别是，分配给任何参数只会影响参数的值，而不会影响参数的值。这意味着


	 aspect A {
	      after() returning (int i): call(int C.foo()) {
		  i = i * 2;
	      }
	  }


不会使建议的返回值增加一倍。相反，它会使本地参数加倍。可以通过使用around建议来更改连接点的参数值或返回值。

通过继续（..），可以通过为变量提供不同的值来更改较先例的建议和基础连接点所使用的值。例如，此方面将替换名为pointcut privateData中的s的字符串：

	aspect A {
	    Object around(String s): MyPointcuts.privateData(s) {
	      return proceed("private data");
	    }
	  }

如果替换参数以继续（..），则当参数引用实际类型的超类型并且不提供实际类型的引用时，您可以在运行时导致ClassCastException。在以下方面，around建议将声明的目标List替换为ArrayList。这是类型匹配后编译时的有效代码。

	  import java.util.*;
	
	  aspect A {
	    Object around(List list): call(* List+.*()) && target(list) {
	      return proceed(new ArrayList());
	    }
	  }

但想象一下一个简单的程序，其中实际的目标是LinkedList。在这种情况下，建议会在运行时导致ClassCastException，并且未在ArrayList中声明peek（）。

	  public class Test {
	    public static void main(String[] args) {
	      new LinkedList().peek();
	    }
	  }

即使在看起来不需要的情况下，例如，如果程序更改为在List中声明的调用size（），ClassCastException也会发生：

	 public class Test {
	    public static void main(String[] args) {
	      new LinkedList().size();
	    }
	  }

仍然会有一个ClassCastException，因为无法证明LinkedList层次结构中的运行时二进制兼容更改或者需要LinkedList的连接点上的某些其他建议是不可能的。


### 建议修饰符 ###

strictfp修饰符是通知中允许的唯一修饰符，它的作用是使通知中的所有浮点表达式都是FP-strict。

### 建议和检查异常 ###

建议声明必须包含一个throws子句，列出正文可能抛出的已检查的异常。 此检查的异常列表必须与建议的每个目标连接点兼容，否则编译器会发出错误信号。

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


这两条建议都是非法的。 第一个是因为主体抛出未声明的检查异常，第二个是因为字段获取连接点不能抛出FileNotFoundExceptions。

AspectJ中每种连接点可能抛出的异常是：

- method call and execution
	- 由目标方法的throws子句声明的检查异常。
- constructor call and execution
	- 由目标构造函数的throws子句声明的checked异常。
- field get and set
	- 没有检查过的异常可以从这些连接点中抛出。
- exception handler execution
	- 目标异常处理程序可能抛出的异常。
- static initializer execution
	- 没有检查过的异常可以从这些连接点中抛出。
- pre-initialization and initialization
	- 初始化类的所有构造函数的throws子句中的任何异常。
- advice execution
	- 任何在通知的throws子句中的异常。



### 建议优先 ###

多条建议可能适用于同一个连接点。在这种情况下，建议的解决顺序基于建议优先顺序。

确定优先顺序

有许多规则决定了一条建议在建议相同连接点时是否优先于另一条建议。

如果这两条建议在不同方面有所界定，则有三种情况：

- 如果方面A在某些声明的优先级形式中比方面B匹配得早，那么当具体方面A中的所有通知在同一个连接点上时，其优先于具体方面B中的所有通知。
- 否则，如果方面A是方面B的子方面，那么在A中定义的所有建议优先于在B中定义的所有建议。因此，除非声明优先级另外指定，否则子方面的建议优先于超方面的建议。
- 否则，如果在两个不同方面定义了两条建议，则没有定义哪一条优先。

如果这两条建议是在同一方面定义的，则有两种情况：

- 如果要么是建议之后，则在该方面稍后出现的那个优先于先前出现的那个。
- 否则，该方面中较早出现的那个优先于稍后出现的那个。
- 
这些规则可以导致循环，例如

	  aspect A {
	      before(): execution(void main(String[] args)) {}
	      after():  execution(void main(String[] args)) {}
	      before(): execution(void main(String[] args)) {}
	  }


**优先效应**

在特定的连接点，建议按优先顺序排列。

一条around建议控制着通过调用proceed来运行较低优先级的通知。继续调用将运行下一个优先级的建议，或者如果没有进一步的建议，则在连接点下执行计算。

之前的一条建议可以通过抛出异常来防止运行较低优先级的建议。但是，如果它正常返回，那么将运行下一个优先级的建议，或者在没有进一步建议的情况下在连接点下的计算。

如果没有进一步的建议，在返回建议后运行将运行下一个优先级的建议，或运行连接点下的计算。然后，如果该计算正常返回，建议的主体将运行。

抛出建议后运行将运行下一个优先级的建议，或者在没有进一步建议的情况下运行连接点下的计算。然后，如果该计算抛出了适当类型的异常，则建议的主体将运行。

在建议之后运行将运行下一个优先级的建议，或者在没有进一步建议的情况下运行连接点下的计算。然后建议的主体将运行。

### 反射访问连接点 ###

在建议体内和if（）切入点表达式中可见三个特殊变量：thisJoinPoint，thisJoinPointStaticPart和thisEnclosingJoinPointStaticPart。每个绑定到一个对象，该对象封装了建议的当前或封闭连接点的一些上下文。存在这些变量是因为某些切入点可能会挑选非常大的连接点集合。例如，切入点


	pointcut publicCall(): call(public * *(..));


挑选出许多方法的调用。然而，这个切入点的建议主体可能希望访问特定连接点的方法名称或参数。

thisJoinPoint绑定到一个完整的连接点对象。

thisJoinPointStaticPart绑定到包含更少信息的连接点对象的一部分，但每次执行建议时都不需要分配内存。它相当于thisJoinPoint.getStaticPart（）。

thisEnclosingJoinPointStaticPart绑定到包含当前连接点的连接点的静态部分。通过这种机制，只有这个封闭连接点的静态部分是可用的。

标准Java反射使用java.lang.reflect层次结构中的对象来构建其反射对象。同样，AspectJ连接点对象在类型层次结构中有类型。绑定到thisJoinPoint的对象的类型是org.aspectj.lang.JoinPoint，而thisStaticJoinPoint绑定到接口类型为org.aspectj.lang.JoinPoint.StaticPart的对象。



### 静态横切 ###

建议声明改变了它们横切的类的行为，但不改变它们的静态类型结构。 对于通过类型层次结构的静态结构进行操作的横切关注点，AspectJ提供了类型间成员声明和其他声明表单。

类型间成员声明

AspectJ允许通过与其他类型相关的方面来声明成员。

类型间方法声明看起来像


- [ Modifiers ] Type OnType . Id(Formals) [ ThrowsClause ] { Body }
- abstract [ Modifiers ] Type OnType . Id(Formals) [ ThrowsClause ] ;

这种声明的效果是使OnType支持新的方法。 即使OnType是一个接口。 即使这种方法既不公开也不抽象。 所以以下是合法的AspectJ代码：

	
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

类型之间的构造函数声明看起来像

- [ Modifiers ] OnType . new ( Formals ) [ ThrowsClause ] { Body }


这种声明的效果是使OnType支持新的构造函数。 OnType成为接口是错误的。

不能使用类型间声明的构造函数为在OnType中声明的最终变量赋值。 此限制显着增加了分别理解和编译OnType类和声明方面的能力。

请注意，在Java语言中，没有定义构造函数的类有一个隐含的无参构造函数，只调用super（）。 这意味着尝试在这样的类上声明一个无参数的类型间构造函数可能会导致冲突，即使看起来没有定义构造函数。

类型间字段声明看起来像是其中之一

- [ Modifiers ] Type OnType . Id = Expression;
- [ Modifiers ] Type OnType . Id;


这种声明的效果是使OnType支持新字段。即使OnType是一个接口。即使该领域既不是公开的，也不是静态的，也不是最终的。

类型间字段声明的初始化程序（如果有）在其目标类中定义的类本地初始化程序之前运行。

在类型间构造函数或方法声明的主体中或在类型间字段声明的初始化程序中，任何出现的标识符都指向OnType对象而不是指向方面类型;从静态类型间成员声明中访问这个位置是错误的。

### 访问修饰符 ###

类型间成员声明可以是公开的或私有的，或者具有默认的（程序包保护的）可见性。 AspectJ不提供受保护的类型间成员。

访问修饰符适用于方面，与目标类型无关。因此，只有在声明方面定义的代码中才能看到私有类型间成员。默认可见性类型间成员只能在声明方面的包中定义的代码中可见。

请注意，声明私有类型间方法（AspectJ支持）与将私有方法声明插入另一个类中非常不同。前者只允许从声明方面访问，而后者只允许从目标类型访问。例如，Java序列化使用私有方法void writeObject（ObjectOutputStream）来实现java.io.Serializable。该方法的私有类型间声明不符合此要求，因为它对于该方面是私有的，而不是对于目标类型是私有的。

抽象类型间方法的访问修饰符有一个约束：在公共接口上声明一个抽象的非公共类型间方法是非法的。这是非法的，因为它会说公共接口的约束条件是只有非公开的实现者必须履行。这不符合Java的类型系统。


### 冲突 ###

类型间声明增加了当地声明的成员和类型间成员之间发生冲突的可能性。例如，假设otherPackage不是包含方面A的包，则为代码


	  aspect A {
	      private Registry otherPackage.onType.r;
	      public void otherPackage.onType.register(Registry r) {
		    r.register(this);
		    this.r = r;
	      }
	  }


声明在otherPackage中的onType有一个字段r。 然而，这个字段只能从A方面的代码中访问。这个方面还声明了onType有一个“register”方法，但是这个方法可以从任何地方访问。

如果onType已经定义了私有或者包保护字段“r”，则不存在冲突：方面不能看到这样的字段，并且otherPackage中的任何代码都不能看到类型间的“r”。

如果onType定义了公共字段“r”，则存在冲突：表达式


	  this.r = r


是一个错误，因为应该使用私有的类型间“r”还是公共的本地定义的“r”是不明确的。

如果onType定义了一个“注册（注册表）”方法，就会产生冲突，因为任何可以看到“注册（注册）”方法适用的定义方法的代码都是不明确的。

根据Java的冲突解决规则，尽可能地解决冲突：

- 一个子类可以从它的超类继承多个字段，全部使用相同的名称和类型。但是，对字段进行模糊引用是错误的。
- 如果只有零个或其中一个是具体的（即，除了一个是抽象的，或者全部是抽象的），则子类只能从其超类中继承具有相同名称和参数类型的多个方法。

考虑到不同类型的成员声明之间存在潜在的冲突，如果一方面优先于另一方面，则其声明将在没有来自编译器的任何冲突通知的情况下生效。当优先级被声明为优先声明以及子方面优先于其超级方面时，情况都是如此。



### 扩展和实施 ###

一个方面可以通过改变一个类型的超类或者将一个超类接口添加到一个类型上来改变系统的继承层次结构，用declare父表单。

- declare parents: TypePattern extends Type;
- declare parents: TypePattern implements TypeList;

例如，如果某个方面希望使某个类可运行，它可能会定义适当的类型间void run（）方法，但它也应该声明该类满足Runnable接口。 为了实现Runnable接口中的方法，类型间run（）方法必须是公共的：


	aspect A {
	      declare parents: SomeClass implements Runnable;
	      public void SomeClass.run() { ... }
	  }


### 与会员接口 ###

通过使用类型间成员，接口现在可以携带（non-public-static-final）字段和类可以继承的（非公共抽象）方法。 由于不明确地继承超类和多个超级接口中的成员，可能会发生冲突。

因为接口可能携带非静态的初始化器，所以每个接口的行为就好像它有一个包含它的初始化器的零参数构造器。 超级接口实例化的顺序是可观察的。 我们使用以下属性修复此顺序：超类型在子类型之前被初始化，初始化的代码只运行一次，而类型超类的初始化器在其超级接口的初始化器之前运行。 考虑下面的层次结构，其中{对象，C，D，E}是类，{M，N，O，P，Q}是接口。


	    Object  M   O
		 \ / \ /
		  C   N   Q
		   \ /   /
		    D   P
		     \ /
		      E

当新E被实例化时，初始化器按此顺序运行：


    Object M C O N D Q P E


### 警告和错误 ###

一个方面可以指定永远不应该达到特定的连接点。

- declare error: Pointcut: String;
- declare warning: Pointcut: String;

如果编译器确定可能会到达Pointcut中的一个连接点，那么它将使用字符串为其消息发出错误或警告，如声明的那样。

### 软化的例外 ###

一个方面可以指定一个特定种类的异常，如果抛出一个连接点，应该绕过Java通常的静态异常检查系统，而是抛出一个org.aspectj.lang.SoftException，它是RuntimeException的子类型，因此不需要 被宣布。

- declare soft: Type: Pointcut;

例如，该方面

	 aspect A {
	      declare soft: Exception: execution(void main(String[] args));
	  }

在执行连接点处，会捕获任何异常并重新抛出包含原始异常的org.aspectj.lang.SoftException。

这与以下建议的作用类似


	  aspect A {
	      void around() execution(void main(String[] args)) {
		  try { proceed(); }
		  catch (Exception e) {
		      throw new org.aspectj.lang.SoftException(e);
		  }
	      }
	  }


除了包装异常外，它还影响Java的静态异常检查机制。

与建议一样，声明软窗体在抽象方面没有影响，但没有通过混凝土方面扩展。 所以下面的代码不会编译，除非它是用扩展的具体方面编译的：


	  abstract aspect A {
	    abstract pointcut softeningPC();
	
	    before() : softeningPC() {     
	      Class.forName("FooClass"); // error:  uncaught ClassNotFoundException
	    }    
	                                                      
	    declare soft : ClassNotFoundException : call(* Class.*(..));
	  }

### 忠告优先 ###

一个方面可以声明具体方面与声明优先表单之间的优先关系：

- declare precedence : TypePatternList ;

这表示如果任何连接点具有来自TypePatternList中某些模式匹配的两个具体方面的建议，则建议的优先级将是列表中的顺序。

在TypePatternList中，通配符“*”最多只能出现一次，它表示“任何类型不与列表中的任何其他模式匹配”。

例如，（1）具有作为其名称一部分的安全性的方面的约束应优先于所有其他方面，并且（2）日志方面（以及扩展它的任何方面）应优先于所有非安全方面 ，可以表示为：

	declare precedence: *..*Security*, Logging+, *;


再举一个例子，CountEntry方面可能想要将当前包中接受Type对象的方法的条目计数为第一个参数。 但是，它应该计算所有条目，即使DisallowNulls方面引发异常的条目也是如此。 这可以通过声明CountEntry优先于DisallowNulls来完成。 该声明可以在任何方面或另一方面排序：


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


### 各种周期 ###

对于任何方面来说，在一个decare优先级中由多个TypePattern匹配是错误的，因此：

	  declare precedence:  A, B, A ;  // error

但是，多个声明优先表单可能在法律上具有这种循环性。 例如，这些声明的优先顺序都是完全合法的：

	  declare precedence: B, A;
	  declare precedence: A, B;

而且，只要A和B的建议不共享连接点，两个约束都处于活动状态的系统也可能是合法的。 所以这是一个可以用来强制A和B强烈独立的成语。

**适用于具体方面**

考虑以下库方面：

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

为了使用任何一个方面，他们必须用具体方面进行扩展，比如MyLogging和MyProfiling。 因为建议只适用于具体方面，所以声明优先权形式仅在声明具体方面的优先权时才重要。 所以

	  declare precedence: Logging, Profiling;

没有效果，但两者都有

	declare precedence: MyLogging, MyProfiling;
	  declare precedence: Logging+, Profiling+;

是有意义的。



### 静态可确定的切入点 ###

出现在声明表单中的切入点具有一定的限制。 像其他切入点一样，这些选择连接点，但它们以静态可确定的方式进行。

因此，这些切入点可能不会直接或间接（通过用户定义的切入点声明）包含基于动态（运行时）上下文进行区分的切入点。 因此，这些切入点可能不会被定义为

- cflow
- cflowbelow
- this
- target
- args
- if

所有这些都可以区分运行时信息。



### Aspects ###


一个方面是由方面声明定义的横切类型。

### 方面声明 ###

方面声明与类声明类似，因为它定义了该类型的类型和实现。它在很多方面有所不同：

**方面实现可以跨越其他类型**

除了常规的Java类声明（例如方法和字段）之外，方面声明还可以包含AspectJ声明，例如通知，切入点和类型间声明。因此，方面包含可以跨越其他类型（包括由其他方面声明定义的类型）的实现声明。

**方面不直接实例化**

方面不直接用新的表达式，克隆或序列化来实例化。方面可能有一个构造函数定义，但如果是这样的话，它必须是一个不带参数的构造函数，并且不会引发检查异常。

**嵌套方面必须是静态的**

方面可以在包级别定义，也可以定义为静态嵌套方面 - 也就是类，接口或方面的静态成员。如果它不在包级别，则必须使用static关键字定义该方面。本地和匿名方面是不允许的。

### Aspect Extension ###

为了支持横切关注的抽象和组合，方面可以以与类相同的方式扩展。但是，Aspect扩展添加了一些新规则。

**方面可以扩展类并实现接口**

一个方面，抽象的或具体的，可以扩展一个类，并可以实现一组接口。扩展类不提供用新表达式实例化方面的能力：方面可能仍然只定义一个空构造函数。


**类可能无法扩展方面**

一个类扩展或实现一个方面是错误的。

**方面扩展方面**

方面可以扩展其他方面，在这种情况下，不仅字段和方法被继承，而且切入点也是如此。但是，各方面只能扩展抽象方面。具体方面扩大另一个具体方面是错误的。

### 方面实例化 ###

与类表达式不同，方面没有用新的表达式实例化。相反，方面实例会自动创建以切割不同的程序。程序可以使用静态方法aspectOf（..）来获取对某个方面实例的引用。

由于建议仅在方面实例的上下文中运行，因此方面实例化间接控制了建议运行的时间。

用于确定某个方面实例化的标准是从其父级方面继承的。如果该方面没有父级方面，那么默认情况下该方面是单身方面。实例化方面如何控制在具体方面类上定义的aspectOf（..）方法的形式。

单身方面

- aspect Id { ... }
- aspect Id issingleton() { ... }

默认情况下（或者通过使用修饰符issingleton（）），一个方面只有一个切割整个程序的实例。该实例在程序执行期间的任何时间都可以从所有具体方面自动定义的静态方法aspectOf（）中获得 - 因此，在上面的示例中，A.aspectOf（）将返回A的实例。此方面实例是在加载方面的类文件时创建的。

因为方面的一个实例存在于程序运行的所有连接点（一旦它的类被加载），它的建议将有机会在所有这样的连接点上运行。

（实际上，方面A的一个实例是针对方面A的每个版本进行的，因此每个A由不同的类加载程序加载时会有一个实例。）

**每个对象方面**

- aspect Id perthis(Pointcut) { ... }
- aspect Id pertarget(Pointcut) { ... }


如果方面A被定义为perthis（Pointcut），则在由Pointcut挑选的任何连接点处为每个执行对象（即，“this”）的每个对象创建类型A的一个对象。在A中定义的通知只会在当前正在执行的对象与A实例关联的连接点上运行。

同样，如果方面A被定义为pertarget（Pointcut），那么为每个对象创建一个类型为A的对象，该对象是由Pointcut挑选的连接点的目标对象。 A中定义的通知只会在目标对象与A实例关联的连接点上运行。

无论哪种情况，都可以使用静态方法调用A.aspectOf（Object）来获取（对象类型A）的aspect实例。每个方面实例尽可能早地创建，但在达到Pointcut挑选的连接点之前没有类型A的关联方面。

正如实现注释附录中所讨论的，Perthis和pertarget方面可能受到AspectJ编译器控件代码的影响。

**每个控制流程方面**

- aspect Id percflow(Pointcut) { ... }
- aspect Id percflowbelow(Pointcut) { ... }

如果方面A被定义为percflow（Pointcut）或percflowbelow（Pointcut），那么为Pointcut选取的每个连接点控制流创建一个类型为A的对象，或者是输入控制流，或者是在 控制流量。 A中定义的建议可以在该控制流中或之下的任何连接点运行。 在每个这样的控制流程中，静态方法A.aspectOf（）将返回一个类型为A的对象。在进入每个这样的控制流程时创建方面的一个实例。

**方面实例化和建议**

所有通知都在一个方面实例的上下文中运行，但是可以用一个切入点来写一条建议，该切入点挑选一个必须在asopect实例化之前发生的连接点。 例如：


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


之前的建议应该在执行系统中的所有构造函数之前运行。它必须在Watchcall方面的实例的上下文中运行。获得这种实例的唯一方法是让Watchcall的默认构造函数执行。但在执行之前，我们需要运行之前的建议......

在编译时没有检测这种循环的一般方法。如果通知在其方面被实例化之前运行，AspectJ将抛出一个org.aspectj.lang.NoAspectBoundException。

### 方面特权 ###
- privileged aspect Id { ... }

写入方面的代码在引用类或方面的成员时受制于与Java代码相同的访问控制规则。因此，例如，在一个方面编写的代码可能不会引用具有默认（程序包保护）可见性的成员，除非该方面是在同一个程序包中定义的。

虽然这些限制适用于很多方面，但建议或类型间成员需要访问其他类型的私有或受保护资源可能存在一些方面。为了做到这一点，可能会声明特权。特权方面的代码可以访问所有成员，甚至私人成员。

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


在这种情况下，如果A没有被声明为特权，则字段引用c.i会导致编译器发出错误信号。

如果一个特权方面可以访问某个特定成员的多个版本，那么那些可以看到它是否没有特权的成员优先。 例如，在代码中


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


A的私人类型间字段C.i（最初绑定到999）将在建议主体中被引用，优先于C的私有声明字段，因为即使它没有特权，A也可以访问它自己的类型间字段。

请注意，特权方面可以访问由其他方面所做的私有类型间声明，因为它们被简单地视为该另一方面的私有成员。
