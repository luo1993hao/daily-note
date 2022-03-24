### 前言
想了解spring aop的整个流程是有一定难度的，需要spring ioc和如何使用aop的基础知识。如何没有这2个基础知识，
就算你看懂了整个aop流程，知识也是凌乱的。而且了解源码整个过程是枯燥的，是漫长的，这篇文章也知识我自己结合源码与网上各位大牛梳理的整个流程，
可能会比较乱，建议结合源码阅读。
### 相关概念
- JoinPoint
- Advice 需要增强的逻辑
    - before
    - after
    - afterReturning
    - throws
    - around
- PointCut 定义哪些方法哪些类需要advice
- Advisor 切面。 组合pointCut+advice
    - 需要增强的目标方法列表，这个通过切入点(Pointcut)来指定
    - 需要在目标方法中增强的逻辑，这个通过(Advice)通知来指定

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

2. 标签解析（ConfigBeanDefinitionParser.parse）

```
...
 //注入或者升级代理创建者，类型为AspectJAwareAdvisorAutoProxyCreator。该bean的id为
 //"org.springframework.aop.config.internalAutoProxyCreator"，专门用于后续创建AOP代理对象。
 //该类核心是继承了BeanPostProcessor
 //不同的使用方式将注册不同的ProxyCreator
 // < aop:config/>->AspectJAwareAdvisorAutoProxyCreator
 //< aop:aspectj-autoproxy/>以及@EnableAspectJAutoProxy->AnnotationAwareAspectJAutoProxyCreator
 //< tx:annotation-driven/>以及@EnableTransactionManagement-> InfrastructureAdvisorAutoProxyCreator
 //但是容器最终只会注册一个，采用优先级最高的
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
2.1  配置自动代理对象(上述的configureAutoProxyCreator(parserContext, element))
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


#### 创建代理对象

- 筛选出所有适合当前Bean的通知器，也就是所有的Advisor、Advise、Interceptor。
- 选择使用JDK还是CGLIB来进行创建代理。
- 使用具体的代理实现来创建代理。

#### 代理的使用

- 获取当前调用方法的拦截器链，包含了所有将要执行的advice。
- 如果没有任何拦截器，直接执行目标方法。
- 如果有拦截器存在，则将拦截器和目标方法封装成一个MethodInvocation，递归调用proceed方法进行调用。
