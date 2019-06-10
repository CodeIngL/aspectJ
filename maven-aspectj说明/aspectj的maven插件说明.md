### aspectj-maven使用 ###



### 多模块构建说明 ###

在多模块项目中，使用AspectJ的方便模式是创建一个包含Aspect定义的项目，供Maven中其他项目的代码使用（“编入”）。由于某些依赖项（如AspectJ运行时）必须在Maven反应器中的所有项目中都相同，因此本演练将分步指导您轻松使用多模块项目中的各个切面。

为了使本演练不太抽象，我们将在整个循序渐进指南中使用验证作为示例。在这个例子中，我们将创建一个小API用于验证对象的内部状态，并通过AspectJ Maven插件将其正确使用委托给AspectJ。除了AspectJ之外，还有许多方法可以执行此操作，但AspectJ是少数不依赖于特定环境（例如容器或应用程序服务器）的方法之一。事实上，小型验证解决方案可以直接在普通的Java SE环境中运行。所有步骤将在下面的演练中详细说明，但我们将首先看看最终结果：

![](http://www.mojohaus.org/aspectj-maven-plugin/images/plantuml/multimodule_ajc.png)

该项目包含以下部分：

- **rootParentPOM**：包含aspectjrt和aspectjtools库的版本定义，用于多模块中的所有项目。

- **validation-api**：定义一个用于验证对象内部状态的小API。

- **validation-aspect**：使用`validation-api`中的类定义一个Aspect来验证对象的内部状态。

- **aspectParentPOM**：配置AspectJ Maven插件（此插件！）以使用在验证方面项目中定义的方面，并将其应用于所有具有父窗口的`aspectParentPOM`的maven项目。

- **mavenProjectWithAspects**：一个项目，其类可以使用/实现`validation-api`中的类型，以表示在`validation-aspect`项目中定义的方面应该被编入它的字节码表示中。
介绍完之后，让我们继续分步指南。


#### 订阅运行时依赖的版本 ####

在rootParentPom中，为AspectJ运行时和库的持有方面定义依赖关系。 建议引入一个Maven属性，以减少必须更改AspectJ版本以升级依赖项的位置数量。


      <!-- +========================================= -->
      <!-- | Maven property definitions               -->
      <!-- +========================================= -->
      <properties>
        <aspectj.runtime.version>1.8.2</aspectj.runtime.version>

        ...
      </properties>


      <!-- +========================================= -->
      <!-- | Dependency (management) settings         -->
      <!-- +========================================= -->
      <dependencyManagement>
          <dependencies>

            <!-- AOP dependencies. -->
            <dependency>
                <groupId>org.aspectj</groupId>
                <artifactId>aspectjrt</artifactId>
                <version>${aspectj.runtime.version}</version>
            </dependency>
            <dependency>
                <groupId>org.aspectj</groupId>
                <artifactId>aspectjtools</artifactId>
                <version>${aspectj.runtime.version}</version>
            </dependency>

            ...

          </dependencies>
      </dependencyManagement>

rootParentPOM中的定义现已完成。


#### 实现Validation-API ####


为了这个例子的目的，我们希望AspectJ为内部状态可以验证的对象强制实施一个共同的行为/生命周期。 Validation-API包含一个定义接口Validatable，它应该由任何一个类创建的对象应该被验证的类来实现。 Validatable接口如下所示：


	/**
	 * Specification for providing an object with Validation mechanics.
	 * Each Validatable object can perform internal validation to assert
	 * its state before being serialized and after being deserialized.
	 *
	 * Making an object implement Validatable does not imply that all
	 * uses of the object is guaranteed. Validatable objects should
	 * primarily make use of their own data to ascertain its valid state.
	 *
	 * It is the responsibilities of services using the Validatable object
	 * (as opposed to the validation mechanics provided within this Validatable)
	 * to provide extra/semantic validation for object <strong>graphs</strong>
	 * in which this Validatable instance is part.
	 */
	public interface Validatable {
	
	    /**
	     * Performs validation of the internal state of this Validatable.
	     *
	     * @throws InternalStateValidationException
	     *          if the state of this Validatable was
	     *          in an incorrect state (i.e. invalid).
	     */
	    void validateInternalState() throws InternalStateValidationException;
	}


在定义了一个名为InternalStateValidationException的自定义异常类之后，它应该扩展IllegalStateException，小的validation-api项目已经完成：


	/**
	 * Exception indicating problems occurred when validating a Validatable instance.
	 */
	public class InternalStateValidationException extends IllegalStateException {
	
	    /**
	     * Constructs an InternalStateValidationException with the specified detail message.
	     * A detail message is a String that describes this particular exception.
	     *
	     * @param message the String that contains a detailed message
	     */
	    public InternalStateValidationException(final String message) {
	        super(message);
	    }
	}

假设rootParentPOM包含应该应用于validation-api项目的通用构建定义，我们应该记得将rootParentPOM作为其父项。 在这个简短的例子中，我们没有理由将验证API连接到父 - 但在现实生活中，诸如许可，覆盖范围，代码风格等的配置通常仅在一个POM中配置和维护 - 即rootParentPOM。

#### 实施切面进行验证 ####


现在我们的小验证API已经完成，现在是时候实施其相应的方面。 这是在validation-aspect项目中完成的，该项目必须将validation-api作为其POM中的依赖项导入。 验证方面项目将其POM父项分配给rootParentPOM也很重要，因为我们想使用父项的AspectJ定义。 验证方面项目的POM位如下所示：


    <!-- +========================================= -->
    <!-- | Define the Parent POM                    -->
    <!-- +========================================= -->
    <parent>
        <groupId>some.group.id</groupId>
        <artifactId>rootParentPOM</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>

    <!-- +========================================= -->
    <!-- | Dependency (management) settings         -->
    <!-- +========================================= -->
    <dependencies>
        <dependency>
            <groupId>some.group.id</groupId>
            <artifactId>validation-api</artifactId>
            <version>1.0.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjrt</artifactId>
            <scope>compile</scope>
        </dependency>

        ...
    </dependencies>

    <!-- +========================================= -->
    <!-- | Build settings                           -->
    <!-- +========================================= -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>aspectj-maven-plugin</artifactId>
                <version>1.11</version>
                <configuration>
                    <complianceLevel>1.6</complianceLevel>
                    <includes>
                        <include>**/*.java</include>
                        <include>**/*.aj</include>
                    </includes>
                </configuration>
                <executions>
                    <execution>
                        <id>compile_with_aspectj</id>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>test-compile_with_aspectj</id>
                        <goals>
                            <goal>test-compile</goal>
                        </goals>
                    </execution>
                </executions>
                <dependencies>
                    <dependency>
                        <groupId>org.aspectj</groupId>
                        <artifactId>aspectjrt</artifactId>
                        <version>${aspectj.runtime.version}</version>
                    </dependency>
                    <dependency>
                        <groupId>org.aspectj</groupId>
                        <artifactId>aspectjtools</artifactId>
                        <version>${aspectj.runtime.version}</version>
                    </dependency>
                </dependencies>
            </plugin>

            ...

除了包含正确的父POM以及来自aspectjrt和validation-api项目的必需依赖项之外，validation-aspect项目的POM还必须定义完整的aspectj-maven-plugin配置以执行aspectj编译。

请注意，aspectjrt和aspectjtools应作为aspectj-maven插件的依赖项包含在此处，以确保在整个反应器中使用了用于AspectJ编译的相同版本的AspectJ。

尽管只有一种主动方法，但方面本身的实现看起来有些复杂。主要的相关方法调用是validatable.validateInternalState（）;这出现在performValidationAfterCompoundConstructor方法的结尾处​​。这是Aspect从validation-api调用方法的地方，因此让对象验证其内部状态。方面中的另外两种方法是两个Pointcut表达式的占位符，用于定义何时应该调用建议。在这个例子中，我们希望在调用默认构造函数以外的任何构造函数之后调用验证。



    Why would you not want to perform validation after a default constructor is called?
    Consider the lifecycle for frameworks which create an object instance by calling the
    default constructor of a class, followed by poulating its internal state
    (i.e. its private members) using reflection.
    Some such frameworks include JAXB and JPA, implying that validation cannot be performed
    immediately after a default constructor has been run since the state of
    the object is still empty.


该方面作为使用AspectJ注释的标准Java类来实现，如下所示：


	/**
	 * The aspect enforcing validity on a class implementing Validatable (i.e. Entities).
	 * This aspect should be fired immediately after a non-default constructor is invoked,
	 * and is intended to run as a singleton.
	 *
	 * Validation should be run only once, and only after the constructor of the ultimate
	 * created instance is run (default AspectJ behaviour is to run the Aspect after any
	 * constructor within the inheritance hierarchy is executed [i.e. after constructors
	 * in superclasses are run, within the constructor of subtypes]).
	 */
	@Aspect
	public class ValidationAspect {
	
	    // Our log
	    private static final Logger log = LoggerFactory.getLogger(ValidationAspect.class);
	
	    /**
	     * Pointcut defining a default constructor within any class.
	     */
	    @Pointcut("initialization(*.new())")
	    void anyDefaultConstructor() {
	    }
	
	    /**
	     * Defines a Pointcut for any constructor in a class implementing Validatable -
	     * except default constructors (i.e. those having no arguments).
	     *
	     * @param joinPoint    The currently executing joinPoint.
	     * @param aValidatable The Validatable instance just created.
	     */
	    @Pointcut(value = "initialization(se.jguru.nazgul.tools.validation.api.Validatable+.new(..)) "
	            + "&& this(aValidatable) "
	            + "&& !anyDefaultConstructor()", argNames = "joinPoint, aValidatable")
	    void anyNonDefaultConstructor(final JoinPoint joinPoint, final Validatable aValidatable) {
	    }
	
	    /**
	     * Validation aspect, performing its job after calling any constructor except
	     * non-private default ones (having no arguments).
	     *
	     * @param joinPoint   The currently executing joinPoint.
	     * @param validatable The validatable instance just created.
	     * @throws InternalStateValidationException
	     *          if the validation of the validatable failed.
	     */
	    @AfterReturning(value = "anyNonDefaultConstructor(joinPoint, validatable)", argNames = "joinPoint, validatable")
	    public void performValidationAfterCompoundConstructor(final JoinPoint joinPoint, final Validatable validatable)
	            throws InternalStateValidationException {
	
	        if (log.isDebugEnabled()) {
	            log.debug("Validating instance of type [" + validatable.getClass().getName() + "]");
	        }
	
	        if (joinPoint.getStaticPart() == null) {
	            log.warn("Static part of join point was null for validatable of type: "
	                    + validatable.getClass().getName(), new IllegalStateException());
	            return;
	        }
	
	        // Ignore calling validateInternalState when we execute constructors in
	        // any class but the concrete Validatable class.
	        final ConstructorSignature sig = (ConstructorSignature) joinPoint.getSignature();
	        final Class<?> constructorDefinitionClass = sig.getConstructor().getDeclaringClass();
	        if (validatable.getClass() == constructorDefinitionClass) {
	
	            // Now fire the validateInternalState method.
	            validatable.validateInternalState();
	
	        } else {
	
	            log.debug("Ignored firing validatable for constructor defined in ["
	                    + constructorDefinitionClass.getName() + "] and Validatable of type ["
	                    + validatable.getClass().getName() + "]");
	        }
	    }
	}


#### 将aspect纳入aspectParentPOM ####

AspectJ插件之谜的最后一部分是将我们的验证方面包含到标准构建周期中，这是在aspectParentPOM项目中执行的。 因此，aspectParentPOM必须包含对验证方面项目的依赖关系，并使用rootParentPOM作为父项。


    <!-- +========================================= -->
    <!-- | Define the Parent POM                    -->
    <!-- +========================================= -->
    <parent>
        <groupId>some.group.id</groupId>
        <artifactId>rootParentPOM</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>

    <!-- +========================================= -->
    <!-- | Dependency (management) settings         -->
    <!-- +========================================= -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>some.group.id</groupId>
                <artifactId>validation-aspect</artifactId>
                <version>1.0.0-SNAPSHOT</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <!-- Include AOP dependencies -->
        <dependency>
            <groupId>some.group.id</groupId>
            <artifactId>validation-aspect</artifactId>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjrt</artifactId>
        </dependency>

        ...
    </dependencies>

    <!-- +========================================= -->
    <!-- | Build settings                           -->
    <!-- +========================================= -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>aspectj-maven-plugin</artifactId>
                <version>1.11</version>
                <configuration>
                    <complianceLevel>1.6</complianceLevel>
                    <includes>
                        <include>**/*.java</include>
                        <include>**/*.aj</include>
                    </includes>
                    <aspectDirectory>src/main/aspect</aspectDirectory>
                    <testAspectDirectory>src/test/aspect</testAspectDirectory>
                    <XaddSerialVersionUID>true</XaddSerialVersionUID>
                    <showWeaveInfo>true</showWeaveInfo>
                    <aspectLibraries>
                        <aspectLibrary>
                            <groupId>some.group.id</groupId>
                            <artifactId>validation-aspect</artifactId>
                        </aspectLibrary>
                    </aspectLibraries>
                </configuration>
                <executions>
                    <execution>
                        <id>compile_with_aspectj</id>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>test-compile_with_aspectj</id>
                        <goals>
                            <goal>test-compile</goal>
                        </goals>
                    </execution>
                </executions>
                <dependencies>
                    <dependency>
                        <groupId>org.aspectj</groupId>
                        <artifactId>aspectjtools</artifactId>
                        <version>${aspectj.runtime.version}</version>
                    </dependency>
                    <dependency>
                        <groupId>some.group.id</groupId>
                        <artifactId>validation-aspect</artifactId>
                        <version>1.0.0-SNAPSHOT</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>


对于现在使用aspectParentPOM作为其父项目的所有项目，validationAspect将被编入实现Validatable接口的所有类中。 这意味着您可以在任何容器内部或外部的对象中执行自动AspectJ驱动的验证。