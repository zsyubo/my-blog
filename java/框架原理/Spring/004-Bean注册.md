终于走到了注册Bean了

```java
	protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
		// BeanDefinitionHolder 是对 BeanDefinition的封装
		// BeanDefinitionHolder 主要是为了解析后生成 BeanDefinition
		BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
		if (bdHolder != null) {
      // todo
			bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
			try {
        // 注册Bean
				BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
			}
.......
			// Send registration event.
			// 发送注册事件
			getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
		}
	}
```

深入`registerBeanDefinition`

```java
	public static void registerBeanDefinition(
			BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
			throws BeanDefinitionStoreException {
		String beanName = definitionHolder.getBeanName();
		// 注册Bean
		registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());
		// 别名处理
		String[] aliases = definitionHolder.getAliases();
		if (aliases != null) {
			for (String alias : aliases) {
				registry.registerAlias(beanName, alias);
			}
		}
	}
```

继续看`registerBeanDefinition`，这里的`registry`是`DefaultListableBeanFactory`

<img src="http://q0ta4ww1m.bkt.clouddn.com/DefaultListableBeanFactory继承结构图.jpg" style="zoom:50%;" />

```java
// 装BeanDefinition 的容器。key为BeanName
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);

// bean定义名称列表，按注册顺序排列。
	private volatile List<String> beanDefinitionNames = new ArrayList<>(256);

//	至少已经创建过一次的bean的名称。
	private final Set<String> alreadyCreated = Collections.newSetFromMap(new ConcurrentHashMap<>(256));

public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {

		Assert.hasText(beanName, "Bean name must not be empty");
		Assert.notNull(beanDefinition, "BeanDefinition must not be null");
		// xml的是GenericBeanDefinition
		if (beanDefinition instanceof AbstractBeanDefinition) {
			try {
				// 校验 beanDefinition
				((AbstractBeanDefinition) beanDefinition).validate();
			}
			catch (BeanDefinitionValidationException ex) {
				throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
						"Validation of bean definition failed", ex);
			}
		}

		BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
		// map中存在，也就是此Bean已经被加载了。
		if (existingDefinition != null) {
			// 是否允许覆盖，isAllowBeanDefinitionOverriding默认为true
			if (!isAllowBeanDefinitionOverriding()) {
				throw new BeanDefinitionOverrideException(beanName, beanDefinition, existingDefinition);
			}
			else if (existingDefinition.getRole() < beanDefinition.getRole()) {
				// e.g. was ROLE_APPLICATION, now overriding with ROLE_SUPPORT or ROLE_INFRASTRUCTURE
				if (logger.isInfoEnabled()) {
					logger.info("Overriding user-defined bean definition for bean '" + beanName +
							"' with a framework-generated bean definition: replacing [" +
							existingDefinition + "] with [" + beanDefinition + "]");
				}
			}
			// 这里直接比较两个BeanDefinition是否相等。只是做记录。
			else if (!beanDefinition.equals(existingDefinition)) {
				if (logger.isDebugEnabled()) {
					logger.debug("Overriding bean definition for bean '" + beanName +
							"' with a different definition: replacing [" + existingDefinition +
							"] with [" + beanDefinition + "]");
				}
			}
			else {
				if (logger.isTraceEnabled()) {
					logger.trace("Overriding bean definition for bean '" + beanName +
							"' with an equivalent definition: replacing [" + existingDefinition +
							"] with [" + beanDefinition + "]");
				}
			}
			// 替换beanDefinition
			this.beanDefinitionMap.put(beanName, beanDefinition);
		}
		else {
		//不存在重复时。
			// 检查该工厂的bean创建阶段是否已经开始，即是否有任何bean被标记为同时创建。
			//  也就是检查是否已经开始创建Bean了。
			if (hasBeanCreationStarted()) {
				// Cannot modify startup-time collection elements anymore (for stable iteration)
				// 如果已经开始创建Bean了就要注意线程安全了。
				synchronized (this.beanDefinitionMap) {
					this.beanDefinitionMap.put(beanName, beanDefinition);
					List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
					updatedDefinitions.addAll(this.beanDefinitionNames);
					updatedDefinitions.add(beanName);
					this.beanDefinitionNames = updatedDefinitions;
					removeManualSingletonName(beanName);
				}
			}
			else {
				// Still in startup registration phase
				this.beanDefinitionMap.put(beanName, beanDefinition);
				this.beanDefinitionNames.add(beanName);
				removeManualSingletonName(beanName);
			}
			this.frozenBeanDefinitionNames = null;
		}

		// 如果此bean以存在，同时又是单例Bean
		if (existingDefinition != null || containsSingleton(beanName)) {
			// 重置Bean  todo
			resetBeanDefinition(beanName);
		}
	}
```

至此Bean的注册ok了。