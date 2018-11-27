### spring mvc
 - dispatcherServlet工作流程
 - 配置dispatcherServlet方法
 - 2个上下文区别
   - dispatcherServlet加载web bean
   - contextLoaderListener加载其他bean,（中间层和数据层）
 - @EnableWebMvc mvc启动注解
 - 注解校验表单
 

---

 ### 视图
- 自带13个视图解析器

----

### mvc高级技术
- springmvc的替代方案
  - 自定义DispatcherServlet配置
     - 重写customizeRegistration()
  - 自定义Servlet
    自定义filter 继承webApplicationInitializer接口 
- 文件图片上传的content-type不同
- 配置mulitipart解析器
  - @bean模式
     - 优点：简洁
     - 缺点：没有构造函数，无法设置临时目录、限制文件大小，需要在servlet初始化类，将细节作为DispatcherServler配置的一部分
  - xml模式
    - <servlet>中的<multipart-config>元素
    - 必须配置<location>
- 处理mulitipart请求
  - @RequestPart注解 与@Part注解类似
  - 二者的差异？
    - 一个是spring的，一个是servlet的
    - 后者不用配置解析器
- 处理异常
  - 特定异常自动映射为http状态码
  - @ResponseStatus->异常映射为状态码
    - 注解在自定义异常中，避免500
  - @ExceptionHandler
    - 处理同一个控制器中所有处理方法抛出的异常
 - 控制器通知
   - 带有@ControllerAdvince注解的类，这个类一般会有@ExceptionHandler、@InitBinder注解、@ModelAttribute注解标注的方法，会运用到所有控制器中@RequestMapping注解的方法
 - 重定向
   -控制器返回的String以“redirect:”
   - String连接缺点？
   - 方式
     - 属性 使用{}占位符，会对不安全字符转译
     - 实体 使用flash
----
### spring web flow
- 目前只支持xml配置
- 流程
1. 创建
2. 配置
 - 状态
   - 行为
   - 决策
   - 结束
   - 子流程
   - 视图
 - 转移
 - 流程数据
  - 五种作用域
 -----
### spring security
 - 基于spring apo与servlet规范中Filter实现的安全框架
 - web请求级别和方法调用级别处理身份认证和授权
 - 重载WebSecurityConfigurerAdapter的configure()
   - 配置fiter链
   - 通过拦截器保护请求
   - 配置user-detail服务
  - @EnableWebMvcSecuriy
  ---
### spring与jdbc
- spring访问数据模板化（模板模式）
- spring将数据访问过程固定和可变的分为2个不同的类：模板和回调，模板管理过程中固定的部分，回调处理自定义的数据访问代码
- 配置数据源
  - 通过jdbc驱动程序定义的数据源
  - 通过jndi查找的数据源
  - 连接池的数据源
----
### spring缓存与安全
- 二种方法
  - 注解驱动
  - xml声明
- @EnableCaching 或者<cache:annotation-driven>启动注解驱动
- 5种缓存管理器
- @Cacheable,@CachePut,@CacheEvict,@Caching
- spring提供三种不同的安全注解