### AOP

> AOP 切面， 主要功能解耦，比如全局异常捕获，请求日志记录，DB事务管理 以及 为了给原方法新增新功能，
> 比如给查询文章方法, queryArticle(), getArticle()等方法，增加已读未读的功能，
> 可以用切面给这几个统一使用切面增加已读，未读功能

#### ***环境***
> 
> spring 版本： 5.3.7
> JDK: 1.8
> tomcat: 8.5

#### spring 配置

```xml
<dependencies>
    <!-- spring aop api and support -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aop</artifactId>
      <version>5.3.7</version>
    </dependency>
    <!-- aop annotation aop aspectj注解支持以及aop weaver -->
     <dependency>
          <groupId>org.aspectj</groupId>
          <artifactId>aspectjweaver</artifactId>
          <version>1.9.6</version>
      </dependency>
</dependencies>
    
```



#### 常用注解

AOP注解：
> 1. @EnableAspectJAutoProxy 开启aop 
> 2. @Component 生成bean
> 3. @Aspect 切面注解 
> 4. @Pointcut 切点，用于确定定位应该在什么地方应用通知 
> 5. @Before 目标方法执行之前执行的逻辑
> 6. @After 目标方法执行之后或者发生异常之后执行的逻辑
> 7. @AfterReturning 目标方法执行成功之后执行的逻辑
> 8. @AfterThrowing  目标方法执行发生异常之后执行的逻辑
> 9. @Around 环绕，Before + process + After, 结合目标执行之前以及之后的执行逻辑

附加
> mvc servlet 配置：src\main\webapp\WEB-INF\springmvc-servlet.xml
> <aop:aspectj-autoproxy proxy-target-class="true"/> 与 @EnableAspectJAutoProxy 一样


```java

@EnableAspectJAutoProxy
@Component
@Aspect
public class ControllerAOP {

    @Pointcut("execution(* com.qzz.controller.BaseController.hello())")
    public void pointcut() {

    }

    @Pointcut("execution(* com.qzz.controller.BaseController.hello(String)) && args(name)")
    public void helloName(String name) {

    }

    @Before("pointcut()")
    public void before() {
        System.out.println("before hello ");
    }

    @Around("helloName(name)")
    public Object around(ProceedingJoinPoint pjp, String name) {
       try {
           System.out.println("hello, " + name);
           Object o = pjp.proceed();
           System.out.println("Good Bye, " + name);
           return o;
       } catch (Throwable throwable) {
           System.out.println("Sorry, " + name);
       }
       return null;
    }
}


```
#### AspectJ指示器
> 1. execution 匹配连接点的执行器 execution(* com.qzz.controller.BaseController.hello(..)) 
> > \* : 任意方法返回类型
> > com.qzz.controller.BaseController.hello(..): 匹配任意参数的连接点方法，也可指定参数类型
> > .. : 匹配任意接收参数
> 2. args 限制匹配参数名
> 3. @args 限制连接点匹配 由指定注解标注的参数的执行方法
> 4. this 限制连接点的bean的指定类型
> 5. target 限制连接点的目标对象的指定类型
> 6. @target 限制连接点目标对象的指定类(具有指定类型的注解)
> 7. within 限制连接点的指定类型
> 8. @within 限制连接点匹配的具有指定注解的指定类型（使用Spring AOP时，方法定义在指定注解所标注的类里）
> 9. @annotation 限制匹配具有指定注解的连接点
> 10. bean Spring引入的指示器，限制匹配指定的bean
* execution 方法执行时，实际执行的匹配，其他都是限制匹配的条件
* &&（and）， ||(or)，!(not), execution 与 其他指示器的标识关系
 