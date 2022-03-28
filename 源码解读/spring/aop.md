### 前言
想了解spring aop的整个流程是有一定难度的，需要spring ioc和如何使用aop的基础知识。如何没有这2个基础知识，
就算你看懂了整个aop流程，知识也是凌乱的。而且了解源码整个过程是枯燥的，是漫长的，这篇文章也知识我自己结合源码与网上各位大牛梳理的整个流程，
而且也只是梳理了主流程，对于细节代码（比如解析标签这块）没有去详细说明，这也是我自己阅读spring源码的一个习惯吧，如果太抠细节，容易越绕越晕。
可能会比较乱，建议结合源码阅读。
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

2. 注册自动代理对象以及标签解析（ConfigBeanDefinitionParser.parse）

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
2.2 标签解析
常见标签配置
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
2.2.1 解析pointCut标签 	:	parsePointcut(elt, parserContext);
作用：parsePointcut方法用于解析< aop:pointcut/>切入点标签，并将一个< aop:pointcut/>标签封装成为一个bean定义并且注册到IoC容器缓存中
关键字：RootBeanDefinition，beanClass类型为AspectJExpressionPointcut。随后以id作为beanName
2.2.2 解析advisor标签:	parseAdvisor(elt, parserContext);
作用：parseAdvisor方法用于解析< aop:advisor/>通知器标签，并将一个aop:advisor/标签封装成为一个bean定义并且注册到IoC容器缓存中：
关键字：RootBeanDefinition，beanClass类型为DefaultBeanFactoryPointcutAdvisor。以id作为beanName或者自动生成beanName，最后注册到容器中。
2.2.3 解析Aspect标签 ：	parseAspect(elt, parserContext);
作用：parseAspect用于解析< aop:aspect/>内部标签。< aop:aspect/>标签本身并不会被注册成为一个bean定义
内部标签说明：
- < aop:declare-parents/>
- advice通知子标签，包括< aop:before/>、< aop:after/>、< aop:after-returning/>、< aop:after-throwing/>、< aop:around/>
- 解析所有< aop:pointcut/>切入点子标签

当前全部标签解析完毕，仅仅是向容器中注册了一些bean定义

3. 创建代理对象
第二步配置的AbstractAdvisorAutoProxyCreator
- < tx:annotation-driven />标签或者@EnableTransactionManagement事物注解，第二步配置的AbstractAdvisorAutoProxyCreator=InfrastructureAdvisorAutoProxyCreator.
- < aop:config />标签，AbstractAdvisorAutoProxyCreator=AspectJAwareAdvisorAutoProxyCreator
- < aop:aspectj-autoproxy />标签或者@EnableAspectJAutoProxy注解，，AbstractAdvisorAutoProxyCreator= AnnotationAwareAspectJAutoProxyCreator.class
负责代理对象的创建。上文说到它继承与

#### 创建代理对象

- 筛选出所有适合当前Bean的通知器，也就是所有的Advisor、Advise、Interceptor。
- 选择使用JDK还是CGLIB来进行创建代理。
- 使用具体的代理实现来创建代理。

#### 代理的使用

- 获取当前调用方法的拦截器链，包含了所有将要执行的advice。
- 如果没有任何拦截器，直接执行目标方法。
- 如果有拦截器存在，则将拦截器和目标方法封装成一个MethodInvocation，递归调用proceed方法进行调用。
