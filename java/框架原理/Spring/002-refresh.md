#refresh

接下来就到了正式的加载Bean的过程了。SpringIOC容器对`Bean`定义资源的载入是从`refresh()`开始的，`refresh()`是一个模板方法，主干在``AbstractApplicationContext`中定义。

在创建 IOC 容器前， `如果已经有容器存在， 则需要把已有的容器销毁和关闭，以保证在 refresh 之后使用的是新建立起来的 IOC 容器`。 `refresh 的作用类似于对 IOC 容器的重启`，在新建立好的容器中对容器进行初始化， 对 Bean 定义资源进行载。以下是方法源代码以及简单的注释：

```java
	public void refresh() throws BeansException, IllegalStateException {
    // 线程安全
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			// 调用容器准备刷新的方法，获取容器的当时时间，同时给容器设置同步标识
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			//告诉子类启动 refreshBeanFactory()方法，Bean 定义资源文件的载入从
			//子类的 refreshBeanFactory()方法启动
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			//为 BeanFactory 配置容器特性，例如类加载器、事件处理器等
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				//为容器的某些子类指定特殊的 BeanPost 事件处理器
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				//调用所有注册的 BeanFactoryPostProcessor 的 Bean
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				//为 BeanFactory 注册 BeanPost 事件处理器.
				// BeanPostProcessor 是 Bean 后置处理器，用于监听容器触发的事件
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				//初始化信息源，和国际化相关.
				initMessageSource();

				// Initialize event multicaster for this context.
				//初始化容器事件传播器.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				//调用子类的某些特殊 Bean 初始化方法
				onRefresh();

				// Check for listener beans and register them.
				//为事件传播器注册事件监听器.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				//初始化所有剩余的单例 Bean
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				//初始化容器的生命周期事件处理器，并发布容器的生命周期事件
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
	}
```

#`prepareRefresh` 容器刷新前的准备

准备此上下文用于刷新、设置其启动日期和活动标志以及执行任何属性源的初始化。

```java
protected void prepareRefresh() {
		// Switch to active.  启动开始时间
		this.startupDate = System.currentTimeMillis();
		// close属性为是否关闭
		this.closed.set(false);
		this.active.set(true);
		// 是否是 debug级别的日志
		if (logger.isDebugEnabled()) {
			if (logger.isTraceEnabled()) {
				logger.trace("Refreshing " + this);
			}
			else {
				logger.debug("Refreshing " + getDisplayName());
			}
		}

		// Initialize any placeholder property sources in the context environment.
  	// 在上下文中初始化占位符资源。
		initPropertySources();

		// Validate that all properties marked as required are resolvable:
		// see ConfigurablePropertyResolver#setRequiredProperties
  // 验证所有标记为必需的属性都是可解析的
		getEnvironment().validateRequiredProperties();

		// Store pre-refresh ApplicationListeners...  初始化 earlyApplicationListeners
		if (this.earlyApplicationListeners == null) {
			this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
		}
		else {
			// Reset local application listeners to pre-refresh state.
			this.applicationListeners.clear();
			this.applicationListeners.addAll(this.earlyApplicationListeners);
		}

		// Allow for the collection of early ApplicationEvents,
		// to be published once the multicaster is available...
		this.earlyApplicationEvents = new LinkedHashSet<>();
	}
```

这此方法也没撒特别的，就一些准备工作。



# `ConfigurableListableBeanFactory beanFactory=obtainFreshBeanFactory();`

这个方法就很重要了，光看着一行，就能看出是初始化`BeanFactory`的。这句话以后的代码都是注册容器的信息源和生命周期时间。

```java
	/**
	 * 告诉子类刷新内部bean工厂。
	 */
	protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
		// 创建一个BeanFactory，创建过程交由子类实现
		refreshBeanFactory();
		return getBeanFactory();
	}

```

这里的`refreshBeanFactory`方法是交由子类去实现，去创建和刷新BeanFactory。最终是调用的：`AbstractRefreshableApplicationContext#refreshBeanFactory`

```java
/**
	 * 创建BeanFactory，如果已有BeanFactory则先销毁并清理已有BeanFactory。
	 */
	@Override
	protected final void refreshBeanFactory() throws BeansException {
		// 判断是否已存在 BeanFactory
		if (hasBeanFactory()) {
			// 销毁
			destroyBeans();
			// 清理
			closeBeanFactory();
		}
		try {
			// 创建IOC容器
			DefaultListableBeanFactory beanFactory = createBeanFactory();
			// 指定一个用于序列化目的的ID，以便根据需要将此BeanFactory从该ID反序列化回BeanFactory对象中。
			beanFactory.setSerializationId(getId());
			// 对 IOC 容器进行定制化， 如设置启动参数， 开启注解的自动装配等
			customizeBeanFactory(beanFactory);
      // 调用载入Bean定义的方法。交由子类实现。核心方法
			loadBeanDefinitions(beanFactory);
			synchronized (this.beanFactoryMonitor) {
				this.beanFactory = beanFactory;
			}
		}
		catch (IOException ex) {
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
	}
```

销毁清理就不细说了，这里看一下`createBeanFactory();`方法。

```java
/**
	* 创建一个BeanFactory
*/
protected DefaultListableBeanFactory createBeanFactory() {
		return new DefaultListableBeanFactory(getInternalParentBeanFactory());
}
```

`getInternalParentBeanFactory()`主要是用于有`parent`的情况，这里主要研究主干部分，就不做详解了。  

`DefaultListableBeanFactory`是实现与`BeanFactory`接口

```java
	/**
	 * Create a new DefaultListableBeanFactory with the given parent.
	 */
	public DefaultListableBeanFactory(@Nullable BeanFactory parentBeanFactory) {
		super(parentBeanFactory);
	}
	
  // super
	public AbstractAutowireCapableBeanFactory(@Nullable BeanFactory parentBeanFactory) {
		this();
		setParentBeanFactory(parentBeanFactory);
	}

// this
	public AbstractAutowireCapableBeanFactory() {
		super();
    // 忽略下面3个接口，也就是在自动装配的时候，忽略下面3个接口的类。
		ignoreDependencyInterface(BeanNameAware.class);
		ignoreDependencyInterface(BeanFactoryAware.class);
		ignoreDependencyInterface(BeanClassLoaderAware.class);
	}
	/**
	 * spuer
	 */
	public AbstractBeanFactory() {
	}

```

上面的方法就是创建一个`DefaultListableBeanFactory`了。这个`DefaultListableBeanFactory`非常重要，后面注册Bean也是会用到。

```java
/**
*  设置两个属性值，不过一般都是为true。
*/
protected void customizeBeanFactory(DefaultListableBeanFactory beanFactory) {
		// 也就是 如果有相同名称不同定义的Bean，则会用后者替换前者，默认true。
		if (this.allowBeanDefinitionOverriding != null) {
			beanFactory.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
		}
		// 是否处理循环引用。默认：true ，如果设置为flase，那么遇到循环应用会抛出异常。
		if (this.allowCircularReferences != null) {
			beanFactory.setAllowCircularReferences(this.allowCircularReferences);
		}
}
```



# loadBeanDefinitions的前置准备工作

ok，开始正式准备解析XML了。是一个抽象方法，具体实现是`AbstractXmlApplicationContext#loadBeanDefinitions(org.springframework.beans.factory.support.DefaultListableBeanFactory)`。

```java
	protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
		// Create a new XmlBeanDefinitionReader for the given BeanFactory.
		// 创建xml   Bean读取器，把创建的BeanFactory注册进去，因为BeanFacoty实现了`BeanDefinitionRegistry`接口
		XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

		// Configure the bean definition reader with this context's
		// resource loading environment.
		beanDefinitionReader.setEnvironment(this.getEnvironment());
    // 因为当前Context实现了ResourceLoader接口，所以可以加载资源。
		beanDefinitionReader.setResourceLoader(this);
		//为 Bean 读取器设置 SAX xml 解析器
		beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

		// Allow a subclass to provide custom initialization of the reader,
		// then proceed with actually loading the bean definitions.
		//当 Bean 读取器读取 Bean 定义的 Xml 资源文件时， 启用 Xml 的校验机制
		initBeanDefinitionReader(beanDefinitionReader);
		//Bean 读取器真正实现加载的方法
		loadBeanDefinitions(beanDefinitionReader);
	}
```

其中的`new XmlBeanDefinitionReader`比较重要，`XmlBeanDefinitionReader`是读取xml配置文件的。

```java
	protected AbstractBeanDefinitionReader(BeanDefinitionRegistry registry) {
		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
		this.registry = registry;

		// Determine ResourceLoader to use.
		// 在XML文件启动时，这儿传入的DefaultListableBeanFactory，所以不是ResourceLoader
		if (this.registry instanceof ResourceLoader) {
			this.resourceLoader = (ResourceLoader) this.registry;
		}
		else {
			// 最终走的这行
			this.resourceLoader = new PathMatchingResourcePatternResolver();
		}
		// Inherit Environment if possible
		if (this.registry instanceof EnvironmentCapable) {
			this.environment = ((EnvironmentCapable) this.registry).getEnvironment();
		}
		else {
			this.environment = new StandardEnvironment();
		}
	}
```

这里需要注意的是：`resourceLoader`引用的是`PathMatchingResourcePatternResolver`。这个在后面`load`时会用到。

`AbstractBeanDefinitionReader#resourceLoader  = new PathMatchingResourcePatternResolver();`



构建好了`Reader`后来看核心的加载Bean的方法

```java
	protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {

		Resource[] configResources = getConfigResources();
		if (configResources != null) {
			reader.loadBeanDefinitions(configResources);
		}
    //  启动传入的xml资源路径
		String[] configLocations = getConfigLocations();
		if (configLocations != null) {
			reader.loadBeanDefinitions(configLocations);
		}
	}
```

如果直接传路径会走下面的String方法

```java
@Override
	public int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException {
		Assert.notNull(locations, "Location array must not be null");
		int count = 0;
		// 多个配置文件循环多次
		for (String location : locations) {
			count += loadBeanDefinitions(location);
		}
		return count;
	}
```

核心方法, 从指定的资源位置加载`BeanDefinitions`

```java
	public int loadBeanDefinitions(String location, @Nullable Set<Resource> actualResources) throws BeanDefinitionStoreException {
		// 获取当前ResourceLoader
		ResourceLoader resourceLoader = getResourceLoader();
		if (resourceLoader == null) {
			throw new BeanDefinitionStoreException(
					"Cannot load bean definitions from location [" + location + "]: no ResourceLoader available");
		}

		// resourceLoader 为 PathMatchingResourcePatternResolver
		if (resourceLoader instanceof ResourcePatternResolver) {
			// Resource pattern matching available.
			try {
				// 通过ResourcePatternResolver，将xml 路径转化Resource资源
				Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
				// loadBeanDefinitions方法具体到子类实现，所以由XmlBeanDefinitionReader实现加载功能。
				int count = loadBeanDefinitions(resources);
				if (actualResources != null) {
					Collections.addAll(actualResources, resources);
				}
				if (logger.isTraceEnabled()) {
					logger.trace("Loaded " + count + " bean definitions from location pattern [" + location + "]");
				}
				return count;
			}
			catch (IOException ex) {
				throw new BeanDefinitionStoreException(
						"Could not resolve bean definition resource pattern [" + location + "]", ex);
			}
		}
		else {
			// Can only load single resources by absolute URL.
			Resource resource = resourceLoader.getResource(location);
			int count = loadBeanDefinitions(resource);
			if (actualResources != null) {
				actualResources.add(resource);
			}
			if (logger.isTraceEnabled()) {
				logger.trace("Loaded " + count + " bean definitions from location [" + location + "]");
			}
			return count;
		}
	}
```

接着看下`AbstractBeanDefinitionReader#loadBeanDefinitions`

```java
@Override
	public int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException {
		Assert.notNull(resources, "Resource array must not be null");
		int count = 0;
		// 当有多个xml文件时循环架子啊，最终返回加载的Bean数量
		for (Resource resource : resources) {
			count += loadBeanDefinitions(resource);
		}
		return count;
	}
```

`loadBeanDefinitions`的具体实现：`XmlBeanDefinitionReader#loadBeanDefinitions`

```java
@Override
	public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
		//将读入的XML进行编码处理
		return loadBeanDefinitions(new EncodedResource(resource));
	}
	
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
		Assert.notNull(encodedResource, "EncodedResource must not be null");
		if (logger.isTraceEnabled()) {
			logger.trace("Loading XML bean definitions from " + encodedResource);
		}
		// 当前记载资源集合，如果为null则初始化，特意使用了ThreadLocal
		Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
		if (currentResources == null) {
			currentResources = new HashSet<>(4);
			this.resourcesCurrentlyBeingLoaded.set(currentResources);
		}
		// 如果添加失败。。。emmm。。。什么情况会出现添加失败？毕竟都初始化了。encodedResource也做了判空处理，同时TreadLocal在执行完此文件读取后也会remove。
		if (!currentResources.add(encodedResource)) {
			throw new BeanDefinitionStoreException(
					"Detected cyclic loading of " + encodedResource + " - check your import definitions!");
		}
		try {
			// 根据resource获取到输入流。
			InputStream inputStream = encodedResource.getResource().getInputStream();
			try {
				// 一个输入源，或者说解析源
				InputSource inputSource = new InputSource(inputStream);
				if (encodedResource.getEncoding() != null) {
					inputSource.setEncoding(encodedResource.getEncoding());
				}
				// 具体的加载方法
				return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
			}
			finally {
				inputStream.close();
			}
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(
					"IOException parsing XML document from " + encodedResource.getResource(), ex);
		}
		finally {
			currentResources.remove(encodedResource);
			if (currentResources.isEmpty()) {
				this.resourcesCurrentlyBeingLoaded.remove();
			}
		}
	}

protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
			throws BeanDefinitionStoreException {

		try {
			// 获取文档的dom对象
			Document doc = doLoadDocument(inputSource, resource);
			// 启动对 Bean定义的解析过程，
			int count = registerBeanDefinitions(doc, resource);
			if (logger.isDebugEnabled()) {
				logger.debug("Loaded " + count + " bean definitions from " + resource);
			}
			return count;
		}
  .....
}

	public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
		// DefaultBeanDefinitionDocumentReader
		BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
		// 获得容器中注册的Bean数量。
		int countBefore = getRegistry().getBeanDefinitionCount();
		// 解析的人口,实际调用的是：DefaultBeanDefinitionDocumentReader.registerBeanDefinitions
		documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
		// 统计解析的Bean的数量
		return getRegistry().getBeanDefinitionCount() - countBefore;
}
```

进行一系列的准备工作后，终于开始解析XML了。

# createReaderContext(resource)

这儿需要注意下此方法

```java
	public XmlReaderContext createReaderContext(Resource resource) {
		return new XmlReaderContext(resource, this.problemReporter, this.eventListener,
				this.sourceExtractor, this, getNamespaceHandlerResolver());
	}
```

























