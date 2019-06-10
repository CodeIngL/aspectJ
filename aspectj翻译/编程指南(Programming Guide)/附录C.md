## 附录C ##

### 编译器注意事项 ###


AspectJ的初始实现都是基于编译器的实现。 AspectJ语义的某些元素很难在不修改虚拟机的情况下实现，这是基于编译器的实现所不能做到的。处理这个问题的一种方法是只指定最容易实现的行为。我们选择了一种稍微不同的方法，即指定理想的语言语义，以及允许实现与该语义偏离的明确定义的方式。这使得今天开发符合AspectJ的实现成为可能，同时还明确了明天实施应该做什么以及可能更好的实现。

根据AspectJ语言语义，该声明

	  before(): get(int Point.x) { System.out.println("got x"); }


应该通知所有类型Point(及其子类型)实例中的int字段x。它应该这样做，无论执行访问的所有源代码在包含此通知的方面切面时是否可用，是否稍后进行更改等等。

但是允许AspectJ实现以一种明确定义的方式偏离它 - 它们被允许通知只在实现的代码控制中访问。每个实现在一定的范围内是自由的，以提供它自己定义的控制代码的含义。

在当前的AspectJ编译器ajc中，对代码的控制意味着在编译期间为任何方面以及它们应该影响的所有代码提供字节码。这意味着如果某些类的客户端包含带有新的new Point().x（它在运行时会产生一个`field get`连接点）的代码，那么当前的AspectJ编译器将无法通知该访问，除非Client.java或Client.class为编译也是如此。这也意味着不通知与本地方法中的代码（包括它们的`execution`连接点）相关的连接点。

不同的连接点有不同的要求。只有当ajc控制`caller`的字节码时，Method 和 constructor的`call`连接点才能被通知。只有当ajc控制"`caller`"的字节码时,`Field reference`或者 `assignment`连接点才能被通知，在实际代码上引用或赋值。只有当ajc控制被初始化的类型的字节码时才能建议`Initialization`连接点，并且只有在ajc控制正在讨论的方法或构造函数体的字节码时才能建议`execution `连接点。异常处理程序的末尾在字节码中是未定的，因此ajc不会实现`handler`连接点的`after`或`around`通知。同样，ajc无法实现关于`initialization`或`preinitialization`连接点的`around`通知。在ajc无法实现建议的情况下，它会发出编译时错误，并将其视为编译器限制。

定义了`perthis`或者`pertarget`的切面也受代控制的限制。特别是，在当前正在执行的对象的字节码不可用的连接点处，该连接点的一个方面定义的`perthis`关联将不会关联。因此定义的方面`perthis(Object)`不会为每个对象创建方面实例，除非`Object`是编译的一部分。类似的限制适用于`pertarget`方面。

诸如`declare parents`之类的类型间声明也有基于代码控制的限制。如果类型间声明的目标字节码不可用，则不在该目标上进行类型间声明。因此，`declare parents : String implements MyInterface`不会为`java.lang.String`工作，除非`java.lang.String`是编译的一部分。

在接口上声明成员时，实现必须控制该接口和顶层实现者（实现接口的类，而不是父类实现接口）。你可以单独编织它们，但要注意，如果你运行受影响的顶层类而没有由相同的ajc实现产生的接口，你将得到运行时异常。任何接口上的抽象方法的类型间声明都必须指定为public，如果不指定public，则会得到编译时错误消息，指出这是编译器限制。在接口上声明的非抽象方法可以使用除protected之外的任何访问修饰符。请注意，这与普通Java规则不同，其中在接口中声明的所有成员都是隐式公开的。最后，请注意，无法在接口上定义静态字段或方法。

在目标类型上声明方法时，只有public方法可以在字节码中识别，所以方法必须声明为public，以便在任何子类型中重写，或者在稍后使用目标类型作为库进行编译时从代码中调用。

其他AspectJ实现，实际上，ajc的未来版本，可以更自由地或限制地定义代码实现控制，只要它们符合Java语言。例如，`call`切点不会选择对`java.lang.reflect.Method.invoke（Object，Object []）`中实现的方法的反射调用。一些人认为调用"`happens`"，`call`切点应该挑选出来，但AspectJ语言不应该预测实现控制之外的代码会发生什么，即使它是Java标准库中定义明确的API 。

要记住的重要一点是，AspectJ的核心概念（如连接点）不会改变，无论使用哪种实现。在您的开发过程中，您必须了解您使用的ajc编译器的局限性，但这些限制不应推动您的切面的设计。

---

### 字节码注意事项 ###


#### .class 表达式 和 String + ####

Java语言形式`Foo.class`在字节码中实现，调用`Class.forName`的方式由捕获`ClassNotFoundException`的异常处理程序保护。

Java语言`+`运算符应用于`String`参数时，通过调用`StringBuffer.append`以字节码形式实现。

在这两种情况下，当前的AspectJ编译器都会对这些语言特性的字节码实现进行操作; 简而言之，它运行在真正发生的事情上，而不是源代码中编写的内容。 这意味着可能有来自程序的`Class.forName`或`StringBuffer.append`的`call`连接点，乍看之下，它们似乎包含这样的调用：

	  class Test {
	      void main(String[] args) {
	          System.out.println(Test.class);        // calls Class.forName
	          System.out.println(args[0] + args[1]); // calls StringBuffer.append
	      }
	  }

简而言之，当前AspectJ编译器的连接点模型将它们视为有效的连接点。

#### Handler连接点 ####

在Java字节码中不能可靠地找到`exception handlers`的结尾。 当前的AspectJ编译器并没有完全删除`handler`连接点，而是限制了`handler`连接点可以做什么：

- `After`和`around`建议不能应用于`Handler`连接点。
- 无法检测控制流中的`Handler`连接点。

其中第一个比较简单。 如果任何一条`after`通知（`returning`, `throwing`或者"`finally`"）通常应用于`handler`连接点，它将不会由当前的AspectJ编译器输出代码。 如果检测到这种情况，则会生成编译器警告。 在允许建议之前。

第二个是在控制流中没有选出`handler`连接点。 例如，下面的切入点

	  cflow(call(void foo()) || handler(java.io.IOException))

将捕获`call(void foo()`连接点中控制流中所有连接点，但它不会捕获控制流中的那些`IOException` `handler`连接点。 它相当于`cflow(call(void foo()))`。 一般来说，`cflow(handler(Type))`不会挑选出任何连接点，对此的一个例外是在`handler`的任何before建议执行期间发生的连接点。

这并不限制程序使用`before`通知处理其他控制流程中的`handlers`。 例如，这个建议非常好：

	  before(): handler(java.io.IOException) && cflow(void parse()) {
	      System.out.println("about to handle an exception while parsing");
	  }

AspectJ的源代码实现（如AspectJ 1.0.6）能够检测`handler`连接点的端点，因此可能会有更少的这种限制。


#### 初始化器和类型间构造器 ####


Java初始化程序的代码，例如赋值给d中的字段

	  class C {
	      double d = Math.sqrt(2);
	  }


在AspectJ获取字节码的时候被认为是构造函数的一部分。 也就是说，d的平方根分配在C的默认构造函数内部。

因此，类型间构造函数不一定会运行目标类型的初始化代码。 特别是，如果类型间构造函数调用超级构造函数（与`this`构造函数相对），则在调用类型间构造函数时，不会运行目标类型的初始化代码。

	  aspect A {

	      C.new(Object o) {} // implicitly calls super()
	
	      public static void main(String[] args) {
	         System.out.println((new C()    ).d);    // prints 1.414...
	         System.out.println((new C(null)).d);    // prints 0.0
	  }

执行所有必需的初始化或者在必要时委托给`this`构造函数是一个类型间构造函数的工作。

---

#### 注解风格注意事项 ####


编写注解风格的切面在字节码上有同样的限制，采用相同的形式并以相同的方式编织。 然而，实现差异（例如，实现`around`通知的机制）在运行时可能很明显。 有关更多信息，请参阅关于注解风格的文档。


