### ajc翻译 ###

#### AspectJ Compiler 1.8.9 ####

	用法: <options> <source file | @argfile>..

AspectJ-指定 选项:

		-inpath <list>      使用目录中的类(当前目录)和<list>中的jars/zips作为源
							（<list>使用平台特定的路径分隔符）
							use classes in dirs and jars/zips in <list> as source
		                    (<list> uses platform-specific path delimiter)

		-injars <jarList>   使用<jarList>下的zip文件中的类作为源
							（<jarList>使用类路径分隔符）
							不建议使用 - 使用inpath代替。

		-aspectpath <list>  将<list> dirs和jars/zip中的.class文件中的切面编织成源文件
							（<list>使用classpath分隔符）

		-outjar <file>      将输出类放入zip文件<file>中

		-outxml             生成 META-INF/aop.xml 文件

		-outxmlfile <file>  指定 -outxml的备用目标输出文件

		-argfile <file>     指定源文件的以行分隔的列表

		-showWeaveInfo      显示织入信息

		-incremental        连续运行的编译器，需要-sourceroots
							（读取stdin：进入重新编译，'q'退出）

		-sourceroots <dirs> 编译<dirs>中的所有.aj和.java文件
						   （<dirs>使用classpath分隔符）

		-crossrefs          生成.ajsym文件到输出目录中

		-emacssym           为emacs支持生成.ajesym符号文件

		-Xlint              等价于 '-Xlint:warning'

		-Xlint:<level>      为横切消息设置默认级别
							（<level>是ignore, warning, or error）

		-Xlintfile <file>   
							指定属性文件来设置每个消息级别
						   （cf org/aspectj/weaver/XlintDefault.properties）

		-X                  打印非标准选项的帮助信息