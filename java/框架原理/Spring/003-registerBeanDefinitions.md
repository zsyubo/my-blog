# 开始从文件中读取BeanDefinitions

正式解析XML配置文件了。`DefaultBeanDefinitionDocumentReader`

```java
	@Override
	public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
		this.readerContext = readerContext;
		doRegisterBeanDefinitions(doc.getDocumentElement());
	}
```

-

```java
	@SuppressWarnings("deprecation")  // for Environment.acceptsProfiles(String...)
	protected void doRegisterBeanDefinitions(Element root) {
		// this behavior emulates a stack of delegates without actually necessitating one.
		BeanDefinitionParserDelegate parent = this.delegate;
		this.delegate = createDelegate(getReaderContext(), root, parent);
		// XML Schema  是用来描述 XML 文档的结构。  判断是否符合SPring所要求的的文档结构
		if (this.delegate.isDefaultNamespace(root)) {
			String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
			if (StringUtils.hasText(profileSpec)) {
				String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
						profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
				if (!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {
					if (logger.isDebugEnabled()) {
						logger.debug("Skipped XML bean definition file due to specified profiles [" + profileSpec +
								"] not matching: " + getReaderContext().getResource());
					}
					return;
				}
			}
		}
		// 在解析Bean定义前，进行自定义的解析，增强解析过程的可拓展性。
		preProcessXml(root);
		// 从根元素开始解析
		parseBeanDefinitions(root, this.delegate);
		// 在解析Bean定义之后进行自定义解析。增加解析过程的可拓展性。
		postProcessXml(root);

		this.delegate = parent;
	}
```

接下来是循环迭代每一个标签

```java
/**
	 * Parse the elements at the root level in the document:
	 * "import", "alias", "bean".
	 * @param root the DOM root element of the document
	 */
	protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
		// 判断是否使用过Spring默认的xml命名空间
		if (delegate.isDefaultNamespace(root)) {
			// 获得Bean标签下的所有子标签
			NodeList nl = root.getChildNodes();

			// 遍历迭代
			for (int i = 0; i < nl.getLength(); i++) {
				// node节点
				Node node = nl.item(i);
				// 如果node节点时xml元素节点(因为有可能取出并不长xml节点)。
				if (node instanceof Element) {
					Element ele = (Element) node;
					// 判断是否是http://www.springframework.org/schema/beans 中的标签，其中只有4个标签：
					// import标签、alias标签、普通bean标签、beans标签
					// 或者是最初的标签吧，component-scan 的都是后来加入的，属于spring-context.xsd的。
					if (delegate.isDefaultNamespace(ele)) {
						// 根据Spring的Bean解析规则解析xml
						parseDefaultElement(ele, delegate);
					}
					else {
						// 说明此标签需要自定义解析。
						delegate.parseCustomElement(ele);
					}
				}
			}
		}
		else {
			// 没有使用spring 命名空间的情况。todo
			delegate.parseCustomElement(root);
		}
	}
```

这里主要探索的是ioc bean的加载，其他复杂标签暂时先放一边。

**Spring 默认有4种主要标签**

```java
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
	  // 如果是import标签
		// 多个 Spring 配置文件可以通过 import 方式整合
		if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
			importBeanDefinitionResource(ele);
		}
		// 如果是alias 标签  别名  , 为Bean注册别名。
		else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
			processAliasRegistration(ele);
		}
		// 如果是普通 bean标签
		else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
			processBeanDefinition(ele, delegate);
		}
		// 如果是beans标签
		else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
			// recurse
			doRegisterBeanDefinitions(ele);
		}
	}
```

# import标签

```java
protected void importBeanDefinitionResource(Element ele) {
		String location = ele.getAttribute(RESOURCE_ATTRIBUTE);
		// 如果路径为空。
		if (!StringUtils.hasText(location)) {
			getReaderContext().error("Resource location must not be empty", ele);
			return;
		}

		// Resolve system properties: e.g. "${user.dir}"
		location = getReaderContext().getEnvironment().resolveRequiredPlaceholders(location);

		Set<Resource> actualResources = new LinkedHashSet<>(4);

		// 判断是否是绝对路径。
		// Discover whether the location is an absolute or relative URI
		boolean absoluteLocation = false;
		try {
			absoluteLocation = ResourcePatternUtils.isUrl(location) || ResourceUtils.toURI(location).isAbsolute();
		}
........

		// Absolute or relative?
		if (absoluteLocation) {
			// 如果是绝对路径的情况。
			try {
				int importCount = getReaderContext().getReader().loadBeanDefinitions(location, actualResources);
				if (logger.isTraceEnabled()) {
					logger.trace("Imported " + importCount + " bean definitions from URL location [" + location + "]");
				}
			}
			......
		}
		else {
			// 如果是相对路径的情况
			// No URL -> considering resource location as relative to the current file.
			try {
				int importCount;
				Resource relativeResource = getReaderContext().getResource().createRelative(location);
				if (relativeResource.exists()) {
					importCount = getReaderContext().getReader().loadBeanDefinitions(relativeResource);
					actualResources.add(relativeResource);
				}
				else {
					String baseLocation = getReaderContext().getResource().getURL().toString();
					importCount = getReaderContext().getReader().loadBeanDefinitions(
							StringUtils.applyRelativePath(baseLocation, location), actualResources);
				}
				.......
		Resource[] actResArray = actualResources.toArray(new Resource[0]);
		getReaderContext().fireImportProcessed(location, actResArray, extractSource(ele));
	}
```

这段逻辑比较简单，就是判断文件时绝对路径还是别的，然后包装成 `Resouce`,最后调用`loadBeanDefinitions`来加载xml。

## alias node

也就是注册别名, 下面是一个案例，为`resourceHolder`Bean注册别名。

```xml
   <alias name="resourceHolder" alias="resourceHolder1"/>
```

具体解析代码

```java
// 处理给定的别名元素，向注册中心注册别名。
	protected void processAliasRegistration(Element ele) {
		// 获取标签中的值。
		String name = ele.getAttribute(NAME_ATTRIBUTE);
		String alias = ele.getAttribute(ALIAS_ATTRIBUTE);
		boolean valid = true;
		if (!StringUtils.hasText(name)) {
			getReaderContext().error("Name must not be empty", ele);
			valid = false;
		}
		if (!StringUtils.hasText(alias)) {
			getReaderContext().error("Alias must not be empty", ele);
			valid = false;
		}
		// 默认true
		if (valid) {
			try {
				// ReaderContext 为DefaultListableBeanFactory
				// 所以此Registry为：SimpleAliasRegistry，具体看继承结构
				getReaderContext().getRegistry().registerAlias(name, alias);
			}
			catch (Exception ex) {
				getReaderContext().error("Failed to register alias '" + alias +
						"' for bean with name '" + name + "'", ele, ex);
			}
			// 在解析完<alias>之后，发送容器别名处理完成事件
			getReaderContext().fireAliasRegistered(name, alias, extractSource(ele));
		}
	}
```

`registerAlias`中的代码也非常简单

```c++
public void registerAlias(String name, String alias) {
		Assert.hasText(name, "'name' must not be empty");
		Assert.hasText(alias, "'alias' must not be empty");
		// 同步
		synchronized (this.aliasMap) {
			// 如果 bean名字和别名一样，则移除别名map中相同别名的。？
			if (alias.equals(name)) {
				// 感觉加不加无所谓吧
				this.aliasMap.remove(alias);
				if (logger.isDebugEnabled()) {
					logger.debug("Alias definition '" + alias + "' ignored since it points to same name");
				}
			}
			else {
				String registeredName = this.aliasMap.get(alias);
				// 如果别名以关联了bean了。
				if (registeredName != null) {
					if (registeredName.equals(name)) {
						// An existing alias - no need to re-register
						return;
					}
					if (!allowAliasOverriding()) {
						throw new IllegalStateException("Cannot define alias '" + alias + "' for name '" +
								name + "': It is already registered for name '" + registeredName + "'.");
					}
					if (logger.isDebugEnabled()) {
						logger.debug("Overriding alias '" + alias + "' definition for registered name '" +
								registeredName + "' with new target name '" + name + "'");
					}
				}
				checkForAliasCircle(name, alias);
				// 就是注册bean
				this.aliasMap.put(alias, name);
				if (logger.isTraceEnabled()) {
					logger.trace("Alias definition '" + alias + "' registered for name '" + name + "'");
				}
			}
		}
	}
```

从代码中可以看出

```java
	/** Map from alias to canonical name. */
	private final Map<String, String> aliasMap = new ConcurrentHashMap<>(16);
```

`aliasMap`就是存储别名的，且别名为key。