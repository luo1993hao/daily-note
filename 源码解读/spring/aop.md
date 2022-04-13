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
我们以xml的方式来说明流程
#### 解析标签，注册代理者
##### 标签解析
**1. 注册parser (AopNamespaceHandler)**


该流程发生在ioc容器启动的obtainFreshBeanFactory方法中，不熟悉ioc流程的同学可以理解为spring启动时候，在解析xml中的元素，并注册为Bean。（实际上是注册为BeanDefinition）
这是在注册标签解析器，用于后续我们xml中标签的解析
```
		// In 2.0 XSD as well as in 2.1 XSD.
		registerBeanDefinitionParser("config", new ConfigBeanDefinitionParser());
		registerBeanDefinitionParser("aspectj-autoproxy", new AspectJAutoProxyBeanDefinitionParser());
		registerBeanDefinitionDecorator("scoped-proxy", new ScopedProxyBeanDefinitionDecorator());
		// Only in 2.0 XSD: moved to context namespace as of 2.1
		registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
```

说明：
- < aop:scoped-proxy/>是作用域代理标签，用于装饰bean，改变其生命周期，将暴露出来的bean的生命周期控制在正确的范围类，用的比较少。
- < aop:config/>用于基于XML配置AOP
- < aop:aspectj-autoproxy/>用于基于XML开启AOP注解自动配置的支持，也就是支持@Aspect切面类及其内部的AOP注解

下面流程我们以常用的<aop:config>的方式来说明,xml中类似于
```
   <aop:config>
        <!--下面这两个 Pointcut 是全局的，可以被所有的 Aspect 使用-->
        <!--这里示意了两种 Pointcut 配置-->
        <aop:pointcut id="logArgsPointcut" expression="execution(* com.javadoop.springaoplearning.service.*.*(..))" />
        <aop:pointcut id="logResultPointcut" expression="com.javadoop.springaoplearning.aop_spring_2_schema_based.SystemArchitecture.businessService()" />

        <aop:aspect ref="logArgsAspect">
            <!--在这里也可以定义 Pointcut，不过这是局部的，不能被其他的 Aspect 使用-->
            <aop:pointcut id="internalPointcut"
                          expression="com.javadoop.springaoplearning.aop_spring_2_schema_based.SystemArchitecture.businessService()" />
            <aop:before method="logArgs" pointcut-ref="internalPointcut" />
        </aop:aspect>

        <aop:aspect ref="logArgsAspect">
            <aop:before method="logArgs" pointcut-ref="logArgsPointcut" />
        </aop:aspect>

        <aop:aspect ref="logResultAspect">
            <aop:after-returning method="logResult" returning="result" pointcut-ref="logResultPointcut" />
        </aop:aspect>


    </aop:config>
```

**2. 配置自动代理对象以及标签解析（ConfigBeanDefinitionParser.parse）**


当注册完标签解析器过后，会根据上述三种使用方式选择不同的解析器进行解析，（我们使用的是aop:config类型，所以将使用ConfigBeanDefinitionParser的解析方式）
该方法做2类事情。
1. 配置自动代理对象，该对象就是专门用于后续创建AOP代理对象
2. 解析标签，并将标签封装成为一个bean定义并且注册到IoC容器缓存中
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

- 解析pointCut标签 : parsePointcut(elt, parserContext); 作用：parsePointcut方法用于解析< aop:pointcut/>切入点标签，并将一个< aop:
pointcut/>标签封装成为一个bean定义并且注册到IoC容器缓存中 关键字：RootBeanDefinition，beanClass类型为AspectJExpressionPointcut。随后以id作为beanName 2.2.2
- 解析advisor标签:parseAdvisor(elt, parserContext); 作用：parseAdvisor方法用于解析< aop:advisor/>通知器标签，并将一个aop:
advisor/标签封装成为一个bean定义并且注册到IoC容器缓存中：
关键字：RootBeanDefinition，beanClass类型为DefaultBeanFactoryPointcutAdvisor。以id作为beanName或者自动生成beanName，最后注册到容器中。
-解析Aspect标签 ： parseAspect(elt, parserContext); 作用：parseAspect用于解析< aop:aspect/>内部标签。< aop:aspect/>标签本身并不会被注册成为一个bean定义 内部标签说明：
  - < aop:declare-parents/>
  - advice通知子标签，包括< aop:before/>、< aop:after/>、< aop:after-returning/>、< aop:after-throwing/>、< aop:around/>
  - 解析所有< aop:pointcut/>切入点子标签

**当前全部标签解析完毕，仅仅是向容器中注册了一些bean定义**

#### 3.创建代理对象

- 筛选出所有适合当前Bean的通知器，也就是所有的Advisor、Advise、Interceptor。
- 选择使用JDK还是CGLIB来进行创建代理。
- 使用具体的代理实现来创建代理。

第二步配置的AbstractAdvisorAutoProxyCreator

- < tx:annotation-driven />
  标签或者@EnableTransactionManagement事物注解，第二步配置的AbstractAdvisorAutoProxyCreator=InfrastructureAdvisorAutoProxyCreator.
- < aop:config />标签，AbstractAdvisorAutoProxyCreator=AspectJAwareAdvisorAutoProxyCreator
- < aop:aspectj-autoproxy />标签或者@EnableAspectJAutoProxy注解，AbstractAdvisorAutoProxyCreator=
  AnnotationAwareAspectJAutoProxyCreator.class 负责代理对象的创建。

**3.1 注册AbstractAdvisorAutoProxyCreator**


  上文说到它继承于BeanPostProcessor,所以他的注册发生ioc容器启动的AbstractApplicationContext refresh流程中的registerBeanPostProcessors


**3.2 创建代理对象**


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

resolveBeforeInstantiation
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

 applyBeanPostProcessorsBeforeInstantiation 点进去会走到AbstractAutoProxyCreator.postProcessBeforeInstantiation

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

 创建bean 点进doCreateBean,核心方法是initializeBean

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
- < aop:config/>->AspectJAwareAdvisorAutoProxyCreator
- < aop:aspectj-autoproxy/>以及@EnableAspectJAutoProxy->AnnotationAwareAspectJAutoProxyCreator
- < tx:annotation-driven/>以及@EnableTransactionManagement-> InfrastructureAdvisorAutoProxyCreator
其父类都是AbstractAutoProxyCreator 所以代码将会进入到AbstractAutoProxyCreator.postProcessAfterInitialization 其中wrapIfNecessary
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

getAdvicesAndAdvisorsForBean 这个方法就是返回该bean的advisors集合，前面说过advisors=pointCut+advice.简单的理解就是返回这个bean的哪个方法需要做哪些aop增强操作的集合。
大概流程如下：

1. 查找现在所有为advisor的bean
2. 过滤适合该bean的advisor
  - 比如我们常用的execution切入点表达式是否满足
3. 添加一个特殊的Advisor到Advisors链头部
4. 对advisors进行排序
  - @order 越小的排在前面
  - 对于我们具体的通知，按照如下的排序方式（这个很重要，后续在具体调用的时候会使用）

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

createProxy 创建代理 将advisors中各种通知以指定的时机织入到相应业务方法中，最终调用就会体现通知时机和通知方法。最终产生的AOP动态代理对象。核心代码就在该方法的最后一行。

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

最终产生的AOP动态代理对象，就会返回到最上级的方法上： AbstractBeanFactory.doGetBean

```

sharedInstance = getSingleton(beanName, () -> {
						try {
					
							return createBean(beanName, mbd, args);
						}
						

getSingleton里面的
singletonObject = singletonFactory.getObject();		（这个是一个function会触发createBean方法）				

```

最后放进singletonObjects单例池中供getBean()获取。 到此，整个AOP的生命周期过程结束。

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

总结： 创建代理对象流程内嵌在bean的创建过程中，只因为我们在第一步解析标签生成的
AutoProxyCreator是一个BeanPostProcessor,所以会做一个后置增强，去找到适配该bean的advisor并将具体属性（在什么方法上进行怎样的增强） 给赋在最后的代理对象上，根据配置选择jdk
或者cglib增强创建代理对象，最后返回增强对象到单例池中

#### 代理的调用

上述流程我们已经创建好了增强对象，在进行对象方法调用时，会根据jdk或者cglib方式进行增强方法的执行。无论是哪种方式，流程都是大同小异
- 获取当前调用方法的拦截器链，包含了所有将要执行的advice。
- 如果没有任何拦截器，直接执行目标方法。
- 如果有拦截器存在，则将拦截器和目标方法封装成一个MethodInvocation，递归调用proceed方法进行调用。
##### jdk动态代理的调用
核心方法在JdkDynamicAopProxy.invoke方法

```
//省去部分代码
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		MethodInvocation invocation;
		Object oldProxy = null;
		boolean setProxyContext = false;
//原始对象
		TargetSource targetSource = this.advised.targetSource;
		Object target = null;
			// Get the interception chain for this method.
			//核心方法之一，上述流程所说的获取当前调用方法的拦截器链
			List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);

			// Check whether we have any advice. If we don't, we can fallback on direct
			// reflective invocation of the target, and avoid creating a MethodInvocation.
			if (chain.isEmpty()) {
				// We can skip creating a MethodInvocation: just invoke the target directly
				// Note that the final invoker must be an InvokerInterceptor so we know it does
				// nothing but a reflective operation on the target, and no hot swapping or fancy proxying.
				Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
				retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
			}
			else {
				// We need to create a method invocation...
				//将所有参数封装成MethodInvocation
				invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
				// Proceed to the joinpoint through the interceptor chain.
				//最核心方法，增调对象的方法调用
				retVal = invocation.proceed();
			}

```

**getInterceptorsAndDynamicInterceptionAdvice 获取当前调用方法的拦截器链**
  在上述流程中我们虽然在代理对象中设置了所有的advisors，但是invoke方法是在方法维度的，要筛选出适合当前方法的advisor并且封装成interceptors。
  比如userService有两个方法，update,create。你设置了在update方法的before aop,create方法的update aop，在userService bean创建阶段就有2个advisor.
  但在具体的update方法执行就需要去找到属于自己的updateAdvisor（名字我随便起的，主要是表达这个意思），并且封装成spring的interceptors供后面proceed流程使用
  getInterceptorsAndDynamicInterceptionAdvice点进去会进入到DefaultAdvisorChainFactory.getInterceptorsAndDynamicInterceptionAdvice方法

```
public List<Object> getInterceptorsAndDynamicInterceptionAdvice(
			Advised config, Method method, @Nullable Class<?> targetClass) {

		// This is somewhat tricky... We have to process introductions first,
		// but we need to preserve order in the ultimate list.
		List<Object> interceptorList = new ArrayList<>(config.getAdvisors().length);
		Class<?> actualClass = (targetClass != null ? targetClass : method.getDeclaringClass());
		boolean hasIntroductions = hasMatchingIntroductions(config, actualClass);
		AdvisorAdapterRegistry registry = GlobalAdvisorAdapterRegistry.getInstance();
   //config.getAdvisors就是上述所说的所有advisors。在这个方法进行筛选
		for (Advisor advisor : config.getAdvisors()) {
			if (advisor instanceof PointcutAdvisor) {
				// Add it conditionally.
				PointcutAdvisor pointcutAdvisor = (PointcutAdvisor) advisor;
				if (config.isPreFiltered() || pointcutAdvisor.getPointcut().getClassFilter().matches(actualClass)) {
				//这里就是上述的将advisor转换为MethodInterceptor
					MethodInterceptor[] interceptors = registry.getInterceptors(advisor);
					MethodMatcher mm = pointcutAdvisor.getPointcut().getMethodMatcher();
					if (MethodMatchers.matches(mm, method, actualClass, hasIntroductions)) {
						if (mm.isRuntime()) {
							// Creating a new object instance in the getInterceptors() method
							// isn't a problem as we normally cache created chains.
							for (MethodInterceptor interceptor : interceptors) {
								interceptorList.add(new InterceptorAndDynamicMethodMatcher(interceptor, mm));
							}
						}
						else {
							interceptorList.addAll(Arrays.asList(interceptors));
						}
					}
				}
			}
			else if (advisor instanceof IntroductionAdvisor) {
				IntroductionAdvisor ia = (IntroductionAdvisor) advisor;
				if (config.isPreFiltered() || ia.getClassFilter().matches(actualClass)) {
					Interceptor[] interceptors = registry.getInterceptors(advisor);
					interceptorList.addAll(Arrays.asList(interceptors));
				}
			}
			else {
				Interceptor[] interceptors = registry.getInterceptors(advisor);
				interceptorList.addAll(Arrays.asList(interceptors));
			}
		}

		return interceptorList;
	}
```

**invocation.proceed()**


  这里进行执行增强方法。基于责任链模式，按顺序递归的执行拦截器链中拦截器的invoke方法以及目标方法。
  所谓顺序，就是上述在创建代理对象时，获取advisors后的sort方法（ReflectiveAspectJAdvisorFactory的static快）。 按照 Around.class, Before.class,
  After.class, AfterReturning.class, AfterThrowing.class的方法去递归执行

```
	public Object proceed() throws Throwable {
		//	We start with an index of -1 and increment early.
		//最后执行，原始方法的调用。从-1开始每次+1。
		if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
			return invokeJoinpoint();
		}
   //调用链就每次获取下一个
		Object interceptorOrInterceptionAdvice =
				this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
		if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
			// Evaluate dynamic method matcher here: static part will already have
			// been evaluated and found to match.
			InterceptorAndDynamicMethodMatcher dm =
					(InterceptorAndDynamicMethodMatcher) interceptorOrInterceptionAdvice;
			if (dm.methodMatcher.matches(this.method, this.targetClass, this.arguments)) {
				return dm.interceptor.invoke(this);
			}
			else {
				// Dynamic matching failed.
				// Skip this interceptor and invoke the next in the chain.
				//递归调用
				return proceed();
			}
		}
		else {
			// It's an interceptor, so we just invoke it: The pointcut will have
			// been evaluated statically before this object was constructed.
			//真正的方法调用，这个invoke方法中就有在目标方法前后的后增强的逻辑。
			//这个this指的是当前的ReflectiveMethodInvocation对象
			return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
		}
	}

```

点击invoke方法，可以看到是MethodInterceptor接口的，有许多实现类。我们aop方式有 Around, Before, After, AfterReturning,
AfterThrowing。也就对应5种Interceptor 具体说明

```
1. ExposeInvocationInterceptor
对于AnnotationAwareAspectJAutoProxyCreator和AspectJAwareAdvisorAutoProxyCreator自动代理创建者，
第一个拦截器就是ExposeInvocationInterceptor，它是在此前尝试创建代理对象的wrapIfNecessary方法中通过extendAdvisors扩展方法加入进去的。
因此它也是第一个执行invoke方法的拦截器。
该拦截器不属于通知方法的拦截器，主要目的是将当前MethodInvocation，
也就是ReflectiveMethodInvocation对象设置到线程本地变量属性invocation中（暴露当前MethodInvocation），方便后续拦截器可以快速获取。

public Object invoke(MethodInvocation mi) throws Throwable {
		MethodInvocation oldInvocation = invocation.get();
		invocation.set(mi);
		try {
			return mi.proceed();
		}
		finally {
			invocation.set(oldInvocation);
		}
	}

2.Around->AspectJAroundAdvice
invoke方法内部并没有直接调用MethodInvocation的proceed()方法，而是将MethodInvocation包装成为一个ProceedingJoinPoint，作为环绕通知的参数给使用者。然后使用者在通知方法中可以调用ProceedingJoinPoint的proceed()方法，
其内部还是调用被包装的MethodInvocation的proceed()方法，这样就将递归调用正常延续了下去。
	@Override
	public Object invoke(MethodInvocation mi) throws Throwable {
		if (!(mi instanceof ProxyMethodInvocation)) {
			throw new IllegalStateException("MethodInvocation is not a Spring ProxyMethodInvocation: " + mi);
		}
		ProxyMethodInvocation pmi = (ProxyMethodInvocation) mi;
		ProceedingJoinPoint pjp = lazyGetProceedingJoinPoint(pmi);
		JoinPointMatch jpm = getJoinPointMatch(pmi);
		//我们自己写around的的时候会手动调用process方法
		return invokeAdviceMethod(pjp, jpm, null, null);
	}
	
3.before->MethodBeforeAdviceInterceptor
public Object invoke(MethodInvocation mi) throws Throwable {
    //通过当前通知实例调用前置通知方法，此时目标方法未被执行
    this.advice.before(mi.getMethod(), mi.getArguments(), mi.getThis());
     //继续调用mi.proceed()
    return mi.proceed();
}
4.after->AspectJAfterAdvice
public Object invoke(MethodInvocation mi) throws Throwable {
		try {
			return mi.proceed();
		}
		finally {
		//finally保证了@after逻辑无论如何都会执行
			invokeAdviceMethod(getJoinPointMatch(), null, null);
		}
	}
5.afterReturning->AfterReturningAdviceInterceptor
@Override
public Object invoke(MethodInvocation mi) throws Throwable {
    Object retVal = mi.proceed();
     // 当递归方法返回时，如果上面的方法抛出了异常，下面这行代码就不会执行
     //说明目标方法已被执行，这是开始执行后置方法
    this.advice.afterReturning(retVal, mi.getMethod(), mi.getArguments(), mi.getThis());
    //返回默认就是目标方法的返回值
    return retVal;
}
6. afterThrowing->AspectJAfterThrowingAdvice
   @Override
	public Object invoke(MethodInvocation mi) throws Throwable {
		try {
			return mi.proceed();
		}
		catch (Throwable ex) {
		//如果抛出的异常类型和当前通知方法参数需要的异常类型匹配，那么可以调用当前通知方法，否则不会调用
			if (shouldInvokeOnThrowing(ex)) {
				invokeAdviceMethod(getJoinPointMatch(), null, ex);
			}
			throw ex;
		}
	}
```

##### cglib动态代理的调用
流程几乎一致
1. 获取适合该方法的调用链
2. 封装成CglibMethodInvocation，这是ReflectiveMethodInvocation的子类 
3. 调用CglibMethodInvocation的proceed方法
```
	@Nullable
		public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
			Object oldProxy = null;
			boolean setProxyContext = false;
			Object target = null;
			TargetSource targetSource = this.advised.getTargetSource();
			try {
				if (this.advised.exposeProxy) {
					// Make invocation available if necessary.
					oldProxy = AopContext.setCurrentProxy(proxy);
					setProxyContext = true;
				}
				// Get as late as possible to minimize the time we "own" the target, in case it comes from a pool...
				target = targetSource.getTarget();
				Class<?> targetClass = (target != null ? target.getClass() : null);
				List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
				Object retVal;
				// Check whether we only have one InvokerInterceptor: that is,
				// no real advice, but just reflective invocation of the target.
				if (chain.isEmpty() && Modifier.isPublic(method.getModifiers())) {
					// We can skip creating a MethodInvocation: just invoke the target directly.
					// Note that the final invoker must be an InvokerInterceptor, so we know
					// it does nothing but a reflective operation on the target, and no hot
					// swapping or fancy proxying.
					Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
					retVal = methodProxy.invoke(target, argsToUse);
				}
				else {
					// We need to create a method invocation...
					retVal = new CglibMethodInvocation(proxy, target, method, args, targetClass, chain, methodProxy).proceed();
				}
				retVal = processReturnType(proxy, target, method, retVal);
				return retVal;
			}
		
```