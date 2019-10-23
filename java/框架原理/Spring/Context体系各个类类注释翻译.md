# # 继承结构图
![](https://s2.ax1x.com/2019/10/23/KGbiqA.png)

# # 类注释翻译
## ## ClassPathXmlApplicationContext
继承于AbstractXmlApplicationContext，类名很好解释了其意义，从ClassPath中加载Xml的ApplicationContext。
```
/**

独立的XML应用程序上下文，从类路径中获取上下文定义文件，将普通路径解释为包含包路径的类路径资源名(例如，“mypackage/myresource.txt”)。

适用于测试工具以及嵌入在jar中的应用程序上下文。

配置位置的默认值可以通过getConfigLocations重写，配置位置可以表示具体的文件，比如“/myfiles/context.xml“或ant样式的模式，比如”/myfiles/*-context.xml。参见org.springframework.util。模式细节的AntPathMatcher javadoc)。

`注意:对于多个配置位置，后面的bean定义将覆盖前面加载的文件中定义的配置位置。可以利用这一点，通过额外的XML文件故意覆盖某些bean定义`

这是一个简单的一站式应用程序上下文。考虑将GenericApplicationContext类与org.springframework.bean.factory.xml.XmlBeanDefinitionReader结合使用。用于更灵活的上下文设置

**/
```
## ## AbstractXmlApplicationContext
从XML中创建ApplicationContext，这里面只是定义了解析XML，加载XML资源有子类去实现。
```
/**
 * 方便的ApplicationContext实现基类,通过XmlBeanDefinitionReader解析XML文档中包含bean定义提取出配置
 * 子类只需要实现getConfigResources和/或getConfigLocations方法。此外，它们可能覆盖getResourceByPath钩子，以特定于环境的方式解释相对路径，以及/或用于扩展模式解析的getResourcePatternResolver。
 */
```
## ## AbstractRefreshableConfigApplicationContext
主要对配置文件做公共处理，同时也是 基于配置文件Context的基类。
```
/**
 * Abstractrefrembleapplicationcontext子类，用于添加对指定配置位置的公共处理。定义了一些公共方法
 * 作为基于xml的应用程序上下文实现的基类，如ClassPathXmlApplicationContext和FileSystemXmlApplicationContext，以及org.springframework.web.context.context.support.Xmlwebapplicationcontext
 */
```

## ## AbstractRefreshableApplicationContext
从名字来看：抽象可刷新ApplicationContext，此类很重要，因为BeanFactory就是在在此类初始化的，此类的方法`refreshBeanFactory()`初始化BeanFactory，这里就不深究了。
```
 /** Bean factory for this context. */
 @Nullable
 private DefaultListableBeanFactory beanFactory;

 /** Synchronization monitor for the internal BeanFactory. */
//  reflush 的锁对象
 private final Object beanFactoryMonitor = new Object();
```
**类注释**
```
/**
 * ApplicationContext实现的基类，它应该支持多个refresh()调用，每次创建一个新的内部bean工厂实例。
 * 通常(但不一定)，这样的上下文将由一组配置位置驱动，以便从这些配置位置加载bean定义。
 * 子类要实现的惟一方法是loadbeandefinition，它在每次刷新时调用。
 * 具体的实现应该将bean定义加载到给定的DefaultListableBeanFactory中，通常委托给一个或多个特定的bean定义阅读器。
 * 注意，WebApplicationContexts也有一个类似的基类。
 * org.springframework.web.context.support.Abstractrefremblewebapplicationcontext提供了相同的子类化策略，但还为web环境预先实现了所有上下文功能。
 * 还有一种预定义的方法来接收web上下文的配置位置。
 * 这个基类的具体独立子类(以特定的bean定义格式读取)是ClassPathXmlApplicationContext和FileSystemXmlApplicationContext，它们都派生自公共AbstractXmlApplicationContext基类;
 */
```
`loadbeandefinition`是在此类定义的抽象方法

## ## AbstractApplicationContext
抽象ApplicationContext，是ApplicationContext接口的抽象实现。此类也使用模板模式(refresh()放就是在此类定义并实现)，定义了很多抽象方法
**类注释**
```
/**
 * ApplicationContext接口的抽象实现。
 * 没有强制要求配置使用的存储类型;
 * 简单地实现公共上下文功能。
 * 使用模板方法设计模式，需要具体的子类来实现抽象方法。
 * 与普通BeanFactory相反，ApplicationContext应该检测在其内部bean工厂中定义的特殊bean(什么特殊Bean):因此，该类自动注册beanfactorypostprocessor、beanpostprocessor和applicationlistener，它们在上下文中定义为bean。
 * MessageSource也可以在上下文中作为bean提供，其名称为“MessageSource”;否则，将消息解析委托给父上下文。
 * 此外，应用程序事件的多播器可以作为上下文中applicationEventMulticaster类型的“applicationEventMulticaster”bean提供;否则，将使用SimpleApplicationEventMulticaster类型的默认多播程序。
 * 通过扩展DefaultResourceLoader实现资源加载。因此，将非url资源路径视为类路径资源(支持包含包路径的完整类路径资源名，例如。除非在子类中重写getResourceByPath方法。
 */
```
`疑问`：什么特殊Bean,  MessageSource是干嘛的？