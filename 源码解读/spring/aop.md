### 前言

想了解spring aop的整个流程是有一定难度的，需要spring ioc和如何使用aop的基础知识。如何没有这2个基础知识，
就算你看懂了整个aop流程，知识也是凌乱的。而且了解源码整个过程是枯燥的，是漫长的，这篇文章也知识我自己结合源码与网上各位大牛梳理的整个流程，
而且也只是梳理了主流程，对于细节代码（比如解析标签这块）没有去详细说明，这也是我自己阅读spring源码的一个习惯吧，如果太抠细节，容易越绕越晕。 可能会比较乱，建议结合源码阅读。

### 相关概念

- JoinPoint 连接点 增加逻辑的位置，程序中可以被调用的方法都可以叫做JoinPoint
    - before
    - after
    - afterReturning
    - throws
    - around
- Advice 需要增强的逻辑
    - before
    - after
    - afterReturning
    - throws
    - around
- PointCut 定义哪些方法哪些类需要advice
- Advisor 。 组合pointCut+advice
    - 需要增强的目标方法列表，这个通过切入点(Pointcut)来指定
    - 需要在目标方法中增强的逻辑，这个通过(Advice)通知来指定
- Aspect 切面 与Advisor差不多

#### 通知类型

- 前置通知（Before advice）: 在某连接点之前执行的通知，但这个通知不能阻止连接点之前的执行流程（除非抛出异常）。
- 后置通知（After returning advice）：在某连接点正常完成后执行的通知。
- 异常通知（After throw advice）: 在方法抛出异常退出时执行的通知。
- 最终通知(After/finally advice): 当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。
- 环绕通知（Around advice）: 包围一个连接点的通知，如方法调用。这是一种最强大的通知类型，环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或者直接返回它自己的返回这或者抛出异常来结束执行。

### 使用方式

- Spring 1.2 基于接口的配置：最早的 Spring AOP 是完全基于几个接口。
- Spring 2.0 schema-based 配置：Spring 2.0 以后使用 XML 的方式来配置，使用 命名空间 <aop />
- Spring 2.0 @AspectJ 配置：使用注解的方式来配置，这种方式感觉是最方便的，还有，这里虽然叫做 @AspectJ，但是这个和 AspectJ 其实没啥关系。
- @EnableAspectJAutoProxy 彻底摆脱XML文件

### 底层原理

原理很简单，通过动态代理来创建代理对象，通过代理对象来访问目标对象，而代理对象中融入了增强的代码，最终起到对目标对象增强的效果。

### 思考

如果让我们自己设计完成aop的流程，需要解决三个问题。1.目标对象的提取 2.找到目标对象的pointCut，也就是在哪儿进行增强 3.如何增强，也就是将advice的逻辑如何织入 所以我们的大概逻辑应该是

1. 创建实例对象
2. 提取切面信息，(advice+pointCut)
3. 在pointCut上织入advice，创建代理对象

### 整体流程

1. 解析标签，注册代理者
2. 创建代理对象
3. 代理对象方法调用时候，通过JDK或者CGLIB进行增强调用

### 详细流程

#### 解析标签，注册代理者

##### 标签解析

1. 注册parser (AopNamespaceHandler)

```
		// In 2.0 XSD as well as in 2.1 XSD.
		registerBeanDefinitionParser("config", new ConfigBeanDefinitionParser());
		registerBeanDefinitionParser("aspectj-autoproxy", new AspectJAutoProxyBeanDefinitionParser());
		registerBeanDefinitionDecorator("scoped-proxy", new ScopedProxyBeanDefinitionDecorator());
		// Only in 2.0 XSD: moved to context namespace as of 2.1
		registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
```

说明：
< aop:scoped-proxy/>是作用域代理标签，用于装饰bean，改变其生命周期，将暴露出来的bean的生命周期控制在正确的范围类，用的比较少。
< aop:config/>用于基于XML配置AOP，
< aop:aspectj-autoproxy/>用于基于XML开启AOP注解自动配置的支持，也就是支持@Aspect切面类及其内部的AOP注解

2. 配置自动代理对象以及标签解析（ConfigBeanDefinitionParser.parse）

```
...
 //注入或者升级代理创建者，类型为AspectJAwareAdvisorAutoProxyCreator。该bean的id为
 //"org.springframework.aop.config.internalAutoProxyCreator"，专门用于后续创建AOP代理对象。
 //该类核心是继承了BeanPostProcessor
 //不同的使用方式将注册不同的ProxyCreator
 // < aop:config/>->AspectJAwareAdvisorAutoProxyCreator
 //< aop:aspectj-autoproxy/>以及@EnableAspectJAutoProxy->AnnotationAwareAspectJAutoProxyCreator
 //< tx:annotation-driven/>以及@EnableTransactionManagement-> InfrastructureAdvisorAutoProxyCreator
 //但是容器最终只会注册一个，采用优先级最高的（InfrastructureAdvisorAutoProxyCreator < AspectJAwareAdvisorAutoProxyCreator < AnnotationAwareAspectJAutoProxyCreator）
	configureAutoProxyCreator(parserContext, element);
 //解析标签
		List<Element> childElts = DomUtils.getChildElements(element);
		for (Element elt: childElts) {
			String localName = parserContext.getDelegate().getLocalName(elt);
			if (POINTCUT.equals(localName)) {
			//解析< aop:pointcut/>,并将标签封装成为一个bean定义并且注册到IoC容器缓存中
				parsePointcut(elt, parserContext);
			}
			else if (ADVISOR.equals(localName)) {
				parseAdvisor(elt, parserContext);
			}
			else if (ASPECT.equals(localName)) {
				parseAspect(elt, parserContext);
			}
		}
		...
```

2.1 配置自动代理对象(上述的configureAutoProxyCreator(parserContext, element))

```
1.
	public static void registerAspectJAutoProxyCreatorIfNecessary(
			ParserContext parserContext, Element sourceElement) {
    
		BeanDefinition beanDefinition = AopConfigUtils.registerAspectJAutoProxyCreatorIfNecessary(
				parserContext.getRegistry(), parserContext.extractSource(sourceElement));
	//解析proxy-target-class与expose-proxy属性
      //proxy-target-class用于设置代理模式，默认是优先JDK动态代理，其次CGLIB代理，可以指定为CGLIB代理
      //expose-proxy用于暴露代理对象，主要用来解决同一个目标类的方法互相调用时代理不生效的问题
		useClassProxyingIfNecessary(parserContext.getRegistry(), sourceElement);
		registerComponentIfNecessary(beanDefinition, parserContext);
	}
2.  AopConfigUtils.registerAspectJAutoProxyCreatorIfNecessary 点进去
     //注册或者修改自动代理创建者的bean定义。
     //不同的标签 cls参数不同。具体对应见上文
     //所以说，如果我们设置了< aop:config />和< aop:aspectj-autoproxy />两个标签，
     //那么最终会注册AnnotationAwareAspectJAutoProxyCreator类型的自动代理创建者。
   	private static BeanDefinition registerOrEscalateApcAsRequired(Class<?> cls, BeanDefinitionRegistry registry,
			@Nullable Object source) {

		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");

		if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
			BeanDefinition apcDefinition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
			if (!cls.getName().equals(apcDefinition.getBeanClassName())) {
			//点进这个方法可以看到优先级。
			//InfrastructureAdvisorAutoProxyCreator>AspectJAwareAdvisorAutoProxyCreator>AnnotationAwareAspectJAutoProxyCreator
				int currentPriority = findPriorityForClass(apcDefinition.getBeanClassName());
				int requiredPriority = findPriorityForClass(cls);
				if (currentPriority < requiredPriority) {
					apcDefinition.setBeanClassName(cls.getName());
				}
			}
			return null;
		}

		RootBeanDefinition beanDefinition = new RootBeanDefinition(cls);
		beanDefinition.setSource(source);
		beanDefinition.getPropertyValues().add("order", Ordered.HIGHEST_PRECEDENCE);
		beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
		registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME, beanDefinition);
		return beanDefinition;
	}
```

2.2 标签解析 常见标签配置

```
<!--AOP配置-->
<aop:config expose-proxy="true">
    <!--切入点-->
    <aop:pointcut id="pointcut" expression="execution(* *(..))"/>
    <!--切面，ref引用切面通知类aspectMethod，方便调用其方法-->
    <aop:aspect id="aspect" ref="aspectMethod">
        <!--通知-->
        <aop:before method="aspect" pointcut-ref="pointcut"/>
        <aop:after method="aspect" pointcut-ref="pointcut"/>
        <aop:after method="aspect" pointcut-ref="pointcut"/>
    </aop:aspect>
</aop:config>
```

2.2.1 解析pointCut标签    :    parsePointcut(elt, parserContext); 作用：parsePointcut方法用于解析< aop:pointcut/>切入点标签，并将一个< aop:
pointcut/>标签封装成为一个bean定义并且注册到IoC容器缓存中 关键字：RootBeanDefinition，beanClass类型为AspectJExpressionPointcut。随后以id作为beanName 2.2.2
解析advisor标签:    parseAdvisor(elt, parserContext); 作用：parseAdvisor方法用于解析< aop:advisor/>通知器标签，并将一个aop:
advisor/标签封装成为一个bean定义并且注册到IoC容器缓存中：
关键字：RootBeanDefinition，beanClass类型为DefaultBeanFactoryPointcutAdvisor。以id作为beanName或者自动生成beanName，最后注册到容器中。 2.2.3
解析Aspect标签 ： parseAspect(elt, parserContext); 作用：parseAspect用于解析< aop:aspect/>内部标签。< aop:aspect/>标签本身并不会被注册成为一个bean定义
内部标签说明：

- < aop:declare-parents/>
- advice通知子标签，包括< aop:before/>、< aop:after/>、< aop:after-returning/>、< aop:after-throwing/>、< aop:around/>
- 解析所有< aop:pointcut/>切入点子标签

当前全部标签解析完毕，仅仅是向容器中注册了一些bean定义


#### 创建代理对象

- 筛选出所有适合当前Bean的通知器，也就是所有的Advisor、Advise、Interceptor。
- 选择使用JDK还是CGLIB来进行创建代理。
- 使用具体的代理实现来创建代理。

 第二步配置的AbstractAdvisorAutoProxyCreator
- < tx:annotation-driven />
  标签或者@EnableTransactionManagement事物注解，第二步配置的AbstractAdvisorAutoProxyCreator=InfrastructureAdvisorAutoProxyCreator.
- < aop:config />标签，AbstractAdvisorAutoProxyCreator=AspectJAwareAdvisorAutoProxyCreator
- < aop:aspectj-autoproxy />标签或者@EnableAspectJAutoProxy注解，AbstractAdvisorAutoProxyCreator=
  AnnotationAwareAspectJAutoProxyCreator.class 负责代理对象的创建。 3.1 注册AbstractAdvisorAutoProxyCreator
  上文说到它继承于BeanPostProcessor,所以他的注册发生ioc容器启动的AbstractApplicationContext refresh流程中的registerBeanPostProcessors 3.2 创建代理对象
  AbstractAdvisorAutoProxyCreator继承BeanPostProcessor，所以可以大胆猜测对代理对象的创建发生在bean初始化填充属性阶段，利于BeanPostProcessor的postProcessBeforeInitialization
  与postProcessAfterInitialization阶段进行代理对象的增强，而bean的创建都在AbstractApplicationContext refresh流程中，代理对象的增强发生在
  beanFactory.preInstantiateSingletons() 最后一步。随后点击getBean(),然后doGetBean()

```
	protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
			//...省略前面代码
				// Create bean instance.
				if (mbd.isSingleton()) {
					sharedInstance = getSingleton(beanName, () -> {
						try {
						//核心方法
						//这一步完成了普通Bean与代理bean的创建
							return createBean(beanName, mbd, args);
						}
						catch (BeansException ex) {
							// Explicitly remove instance from singleton cache: It might have been put there
							// eagerly by the creation process, to allow for circular reference resolution.
							// Also remove any beans that received a temporary reference to the bean.
							destroySingleton(beanName);
							throw ex;
						}
					});
					bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
				}
			}

```

再点进createBean

```
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
			throws BeanCreationException {
			///省略前面代码
				try {
			// Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
			//第一次调用，该方法前bean还没有被实例化出来，这个resolveBeforeInstantiation方法执行后也还没有实例化出bean，
			//这个方法比较有迷惑性，简单来说这个方法做了2个事情。（第一点甚至可以忽略）
			//1.如果自定义的了一个targetSource（这块还不是很清楚，目前所了解的都是没有自定义的），则不走下面doCreateBean，也就是spring创建bean的流程。随后进行代理增强，最后返回
			//2.将一些aop基类作为黑名单加入一个集合，后续这些类都不做aop处理
			Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
			if (bean != null) {
				return bean;
			}
		}
		catch (Throwable ex) {
			throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
					"BeanPostProcessor before instantiation of bean failed", ex);
		}

		try {
		//这才是核心流程
			Object beanInstance = doCreateBean(beanName, mbdToUse, args);
			if (logger.isDebugEnabled()) {
				logger.debug("Finished creating instance of bean '" + beanName + "'");
			}
			return beanInstance;
		}
			}

```

3.2.1 resolveBeforeInstantiation
点进resolveBeforeInstantiation方法，核心方法就是applyBeanPostProcessorsBeforeInstantiation与applyBeanPostProcessorsAfterInitialization，
其中applyBeanPostProcessorsBeforeInstantiation

```
	@Nullable
	protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
		Object bean = null;
		if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
			// Make sure bean class is actually resolved at this point.
			//mbd.isSynthetic()：一个类是否为合并类（如内部类），一般扫描的都不是合并类
			//这个hasInstantiationAwareBeanPostProcessors标志是在refresh方法的registerBeanPostProcessors方法内赋值了，

			if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
				Class<?> targetType = determineTargetType(beanName, mbd);
				if (targetType != null) {
					bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
					if (bean != null) {
						bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
					}
				}
			}
			mbd.beforeInstantiationResolved = (bean != null);
		}
		return bean;
	}

```

3.2.1 applyBeanPostProcessorsBeforeInstantiation 点进去会走到AbstractAutoProxyCreator.postProcessBeforeInstantiation

```
//这个方法基本都会返回null，所以上述流程的applyBeanPostProcessorsAfterInitialization 也不会执行
@Override
public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) {
 
   //第一步：通过beanName返回对应类名，
   //如果beanName对应普通Bean则直接返回beanClass,
   //如果beanName对应FactoryBean的话则名称前加上&表明得到的是FactoryBean本身类名(这个是Bean的知识点)
   Object cacheKey = getCacheKey(beanClass, beanName);
 
   //第二步：对扫描的所有bean，判断哪些bean是不能被代理，获取已经被代理了，
   //如果发现了上述两种bean则放入到advisedBeans中，这个advisedBeans对象的作用是辅助剔除哪些bean不能被AOP代理。
   //我们知道，切面类，类中包含Advice，Pointcut，Advisor，AopInfrastructureBean都是不能被代理的bean，
   //而且启动配置类加了开启AOP注解则是已经被代理的bean，也会被放入这个advisedBeans中
   if (!StringUtils.hasLength(beanName) || !this.targetSourcedBeans.contains(beanName)) {
      if (this.advisedBeans.containsKey(cacheKey)) {
         return null;
      }
      //isInfrastructureClass会判断是否包含Aspect注解，或者xml中配置为了Aspect，
      //如果是就会放入到advisedBeans中
      if (isInfrastructureClass(beanClass) || shouldSkip(beanClass, beanName)) {
         this.advisedBeans.put(cacheKey, Boolean.FALSE);
         return null;
      }
   }
 
   // Create proxy here if we have a custom TargetSource.
   // Suppresses unnecessary default instantiation of the target bean:
   // The TargetSource will handle target instances in a custom fashion.
 
   //第三步：内部通过internalBeanFactory.registerBeanDefinition(beanName, bdCopy)产生一个目标对象targetSource
   //但是这一步都是返回了null。目前还不了解不返回null的场景，下面代码可以忽略
   TargetSource targetSource = getCustomTargetSource(beanClass, beanName);
 
   //第四步：获取通知，创建代理对象。由于targetSource都是null，所以下面代码都不会执行，整个方法都返回null
   if (targetSource != null) {
      if (StringUtils.hasLength(beanName)) {
         this.targetSourcedBeans.add(beanName);
      }
 
      //从Bean中获取所有通知
      Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(beanClass, beanName, targetSource);
 
      //创建代理对象
      Object proxy = createProxy(beanClass, beanName, specificInterceptors, targetSource);
 
      //放入proxyTypes缓存Map中
      this.proxyTypes.put(cacheKey, proxy.getClass());
      return proxy;
   }
 
   return null;

```

3.2.2 创建bean 点进doCreateBean,核心方法是initializeBean

```
	protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {
			//......
				// Initialize the bean instance.
		Object exposedObject = bean;
		try {
			populateBean(beanName, mbd, instanceWrapper);
			exposedObject = initializeBean(beanName, exposedObject, mbd);
		}
			}

```

点进initializeBean

```
	protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
	//.....
		Object wrappedBean = bean;
		if (mbd == null || !mbd.isSynthetic()) {
		//这里其实什么都没做，直接返回的bean
			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
		}

		try {
			invokeInitMethods(beanName, wrappedBean, mbd);
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					(mbd != null ? mbd.getResourceDescription() : null),
					beanName, "Invocation of init method failed", ex);
		}
		if (mbd == null || !mbd.isSynthetic()) {
		//核心方法，进行后置增强
			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
		}
	}

```
点进applyBeanPostProcessorsAfterInitialization，我们上述说了无论使用哪种AOP方式，
 < aop:config/>->AspectJAwareAdvisorAutoProxyCreator
< aop:aspectj-autoproxy/>以及@EnableAspectJAutoProxy->AnnotationAwareAspectJAutoProxyCreator
< tx:annotation-driven/>以及@EnableTransactionManagement-> InfrastructureAdvisorAutoProxyCreator
其父类都是AbstractAutoProxyCreator
所以代码将会进入到AbstractAutoProxyCreator.postProcessAfterInitialization
其中wrapIfNecessary
这个方法非常重要，关于判断是否代理，创建AOP动态代理对象以及往黑名单map中添加如配置启动类等操作都在这里

```
	@Override
	public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) throws BeansException {
		if (bean != null) {
			Object cacheKey = getCacheKey(bean.getClass(), beanName);
			if (!this.earlyProxyReferences.contains(cacheKey)) {
				return wrapIfNecessary(bean, beanName, cacheKey);
			}
		}
		return bean;
	}

```
点进wrapIfNecessary，核心方法为getAdvicesAndAdvisorsForBean与createProxy
```
	protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
	//  如果targetSourcedBeans缓存中包含该beanName，表示已通过TargetSource创建了代理，直接返回原始bean实例
      targetSourcedBeans在postProcessBeforeInstantiation中就见过了
		if (StringUtils.hasLength(beanName) && this.targetSourcedBeans.contains(beanName)) {
			return bean;
		}
		//   如果advisedBeans缓存中包含该cacheKey，并且value为false，表示不需要代理，直接返回原始bean实例
     //advisedBeans在postProcessBeforeInstantiation中就见过了
		if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
			return bean;
		}
		//   如果当前bean是Spring AOP的基础结构类，或者shouldSkip返回true，表示不需要代理，直接返回原始bean实例
     //这个shouldSkip方法，在AspectJAwareAdvisorAutoProxyCreator子类中将会被重写并初始化全部的Advisor通知器实例
     // 这两个方法在postProcessBeforeInstantiation中就见过了

		if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
			this.advisedBeans.put(cacheKey, Boolean.FALSE);
			return bean;
		}

		// Create proxy if we have advice.
		//首先会根据bean查找以这个bean作为连接点的所有通知放到Object[]数组中：
		Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
		if (specificInterceptors != DO_NOT_PROXY) {
			this.advisedBeans.put(cacheKey, Boolean.TRUE);
			//通过JDK或者CGLIB创建代理对象，使用默认的SingletonTargetSource包装目标对象
			Object proxy = createProxy(
					bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
			this.proxyTypes.put(cacheKey, proxy.getClass());
			return proxy;
		}

		this.advisedBeans.put(cacheKey, Boolean.FALSE);
		return bean;
	}

```


getAdvicesAndAdvisorsForBean
这个方法就是返回该bean的advisors集合，前面说过advisors=pointCut+advice.简单的理解就是返回这个bean的哪个方法需要做哪些aop增强操作的集合。
大概流程如下：
1. 查找现在所有为advisor的bean
2. 过滤适合该bean的advisor
  - 比如我们常用的execution切入点表达式是否满足
3. 添加一个特殊的Advisor到Advisors链头部
4. 对advisors进行排序
 - 1.@order 越小的排在前面
 - 2.对于我们具体的通知，按照如下的排序方式（这个很重要，后续在具体调用的时候会使用）

```
**ReflectiveAspectJAdvisorFactory**
	static {
		Comparator<Method> adviceKindComparator = new ConvertingComparator<>(
				new InstanceComparator<>(
						Around.class, Before.class, After.class, AfterReturning.class, AfterThrowing.class),
				(Converter<Method, Annotation>) method -> {
					AspectJAnnotation<?> annotation =
						AbstractAspectJAdvisorFactory.findAspectJAnnotationOnMethod(method);
					return (annotation != null ? annotation.getAnnotation() : null);
				});
		Comparator<Method> methodNameComparator = new ConvertingComparator<>(Method::getName);
		METHOD_COMPARATOR = adviceKindComparator.thenComparing(methodNameComparator);
	}
```

- createProxy 创建代理
 将advisors中各种通知以指定的时机织入到相应业务方法中，最终调用就会体现通知时机和通知方法。最终产生的AOP动态代理对象。核心代码就在该方法的最后一行。
```
return proxyFactory.getProxy(getProxyClassLoader());
 
 点进getProxy，进入DefaultAopProxyFactory的createAopProxy


根据配置类型选择创建JDK或者CGLIB类型的AopProxy并返回。 大概逻辑如下：

如果isOptimize方法返回true，即optimize属性为true，表示CGLIB代理应该主动进行优化，默认false；或者如果isProxyTargetClass方法返回true，即proxyTargetClass属性为true，表示应该使用CGLIB代理，默认false；或者如果hasNoUserSuppliedProxyInterfaces方法返回true，表示没有可使用的合理的代理接口或者只有一个代理接口并且属于SpringProxy接口体系（即校验interfaces集合，这个集合就是在此前evaluateProxyInterfaces方法中加入的接口集合）。以上三个条件满足任意一个，继续判断：
如果要代理的目标类型是接口，或者目标类型就是Proxy类型，这两个条件满足任意一个，那么还是采用JDK代理，返回JdkDynamicAopProxy类型的AopProxy，proxyFactory作为构造器参数。
两个条件都不满足，那么最终采用CGLIB代理，返回ObjenesisCglibAopProxy类型的AopProxy，proxyFactory作为构造器参数。

 	public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
		if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
			Class<?> targetClass = config.getTargetClass();
			if (targetClass == null) {
				throw new AopConfigException("TargetSource cannot determine target class: " +
						"Either an interface or a target is required for proxy creation.");
			}
			if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
				return new JdkDynamicAopProxy(config);
			}
			return new ObjenesisCglibAopProxy(config);
		}
		else {
			return new JdkDynamicAopProxy(config);
		}
	}


```
最终产生的AOP动态代理对象，就会返回到最上级的方法上：
AbstractBeanFactory.doGetBean
```

sharedInstance = getSingleton(beanName, () -> {
						try {
					
							return createBean(beanName, mbd, args);
						}
						

getSingleton里面的
singletonObject = singletonFactory.getObject();		（这个是一个function会触发createBean方法）				

```
最后放进singletonObjects单例池中供getBean()获取。
到此，整个AOP的生命周期过程结束。

```
--getBean
   --doGetBean
    --getSingleton
     --createBean
      --resolveBeforeInstantiation
       --applyBeanPostProcessorsBeforeInstantiation
        --doCreateBean
         --initializeBean
         postProcessBeforeInitialization
          --applyBeanPostProcessorsAfterInitialization
           --postProcessAfterInitialization
            --wrapIfNecessary
             --createProxy
     --addSingleton

```

总结：
创建代理对象流程内嵌在bean的创建过程中，只因为我们在第一步解析标签生成的
AutoProxyCreator是一个BeanPostProcessor,所以会做一个后置增强，去找到适配该bean的advisor并将具体属性（在什么方法上进行怎样的增强）
给赋在最后的代理对象上，根据配置选择jdk 或者cglib增强创建代理对象，最后返回增强对象到单例池中




#### 代理的使用

- 获取当前调用方法的拦截器链，包含了所有将要执行的advice。
- 如果没有任何拦截器，直接执行目标方法。
- 如果有拦截器存在，则将拦截器和目标方法封装成一个MethodInvocation，递归调用proceed方法进行调用。
