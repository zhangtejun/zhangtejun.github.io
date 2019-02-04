---
layout: post
title:  "spring aop"
date:   2017-10-21 08:13:04
author: zhangtejun
categories: Spring
---
##### Introduction
面向切面编程（AOP）提供另外一种思考程序结构的方式，通过这种方式弥补了面向对象编程（OOP）的不足。 

OOP模块化的关键单位是类（class），而在AOP中，模块化单位是方面。方面可以实现诸如跨多个类型和对象之间的事
务管理等问题的模块化。 （这种关注在AOP文献中通常被称为横切关注点。）

Spring的一个关键的组件就是 AOP框架(大多数AOP框架以拦截器做通知模型)。 

尽管如此，Spring IoC容器并不依赖于AOP，这意味着你可以自由选择是否使用AOP，AOP提供强大的中间件解决方案，这使得Spring IoC容器更加完善。

**Spring 2.0 AOP**

Spring 2.0 引入了一种更加简单并且更强大的方式来自定义切面，用户可以选择使用基于模式（schema-based）的方式或者使用@AspectJ注解。 这两种风格都完全支持通知（Advice）类型和AspectJ的切入点语言，虽然实际上仍然使用Spring AOP进行织入（Weaving）。

_Spring中所使用的AOP：_

* 提供声明式企业服务，特别是为了替代EJB声明式服务。 最重要的服务是 声明性事务管理（declarative transaction management） ， 这个服务建立在Spring的抽象事务管理（transaction abstraction）之上。 

* 允许用户实现自定义的切面，用AOP来完善OOP的使用。这样你可以把Spring AOP看作是对Spring的一种增强，它使得Spring可以不需要EJB就能提供声明式事务管理； 或者也可以使用Spring AOP框架的全部功能来实现自定义的切面。 

######  AOP concepts/AOP概念
* 切面（Aspect）： 一个关注点的模块化，这个关注点可能会横切多个对象。事务管理是J2EE应用中一个关于横切关注点的很好的例子。 在Spring AOP中，切面可以使用通用类（基于模式的风格） 或者在普通类中以 @Aspect 注解（@AspectJ风格）来实现。 

* 连接点（Joinpoint）： 在程序执行过程中某个特定的点，比如**某方法调用**的时候或者**处理异常**的时候。 在Spring AOP中，一个连接点 总是 代表一个方法的执行。 

* 通知（Advice）： 在切面的某个特定的连接点（Joinpoint）上执行的动作。通知有各种类型，其中包括“around”、“before”和“after”等通知。 通知的类型将在后面部分进行讨论。许多AOP框架，包括Spring，都是**以拦截器做通知模型**， 并维护一个以连接点为中心的拦截器链。 

* 切入点（Pointcut）： 匹配连接点（Joinpoint）的断言。通知和一个切入点表达式关联，并在满足这个切入点的连接点上运行（例如，当执行某个特定名称的方法时）。 切入点表达式如何和连接点匹配是AOP的核心：Spring缺省使用AspectJ切入点语法。 

* 引入（Introduction）： （也被称为内部类型声明（inter-type declaration））。声明额外的方法或者某个类型的字段。 Spring允许引入新的接口（以及一个对应的实现）到任何被代理的对象。 例如，你可以使用一个引入来使bean实现 IsModified 接口，以便简化缓存机制。 

* 目标对象（Target Object）： 被一个或者多个切面（aspect）所通知（advise）的对象。也有人把它叫做 被通知（advised） 对象。 既然Spring AOP是通过运行时代理实现的，这个对象永远是一个 被代理（proxied） 对象。 

* AOP代理（AOP Proxy）： AOP框架创建的对象，用来实现切面契约（aspect contract）（包括通知方法执行等功能）。 在Spring中，AOP代理可以是JDK动态代理或者CGLIB代理。 注意：Spring 2.0最新引入的基于模式（schema-based）风格和@AspectJ注解风格的切面声明，对于使用这些风格的用户来说，代理的创建是透明的。 

* 织入（Weaving）： 把切面（aspect）连接到其它的应用程序类型或者对象上，并创建一个被通知（advised）的对象。 这些可以在编译时（例如使用AspectJ编译器），类加载时和运行时完成。 Spring和其他纯Java AOP框架一样，在运行时完成织入。 

**通知的类型：**

* 前置通知（Before advice）： 在某连接点（join point）之前执行的通知，但这个通知不能阻止连接点前的执行（除非它抛出一个异常）。 

* 返回后通知（After returning advice）： 在某连接点（join point）正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回。 

* 抛出异常后通知（After throwing advice）： 在方法抛出异常退出时执行的通知。 

* 后通知（After (finally) advice）： 当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。 

* 环绕通知（Around Advice）： 包围一个连接点（join point）的通知，如方法调用。这是最强大的一种通知类型。 环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它们自己的返回值或抛出异常来结束执行。 

环绕通知是最常用的一种通知类型。大部分基于拦截的AOP框架，例如Nanning和JBoss4，都只提供环绕通知。 

跟AspectJ一样，Spring提供所有类型的通知，我们推荐你使用尽量简单的通知类型来实现需要的功能。 例如，如果你只是需要用一个方法的返回值来更新缓存，虽然使用环绕通知也能完成同样的事情， 但是你最好使用After returning通知而不是环绕通知。 用最合适的通知类型可以使得编程模型变得简单，并且能够避免很多潜在的错误。 比如，你不需要调用 JoinPoint（用于Around Advice）的 proceed() 方法，就不会有调用的问题。 

在Spring 2.0中，所有的通知参数都是静态类型，因此你可以使用合适的类型（例如一个方法执行后的返回值类型）作为通知的参数而不是使用一个对象数组。 

切入点（pointcut）和连接点（join point）匹配的概念是AOP的关键，这使得AOP不同于其它仅仅提供拦截功能的旧技术。 切入点使得定位通知（advice）可独立于OO层次。 例如，一个提供声明式事务管理的around通知可以被应用到一组横跨多个对象中的方法上（例如服务层的所有业务操作）。 


##### Spring AOP的功能和目标
Spring AOP用纯Java实现。它不需要专门的编译过程。Spring AOP不需要控制类装载器层次，因此它适用于J2EE web容器或应用服务器。 

Spring目前仅支持使用方法调用作为连接点（join point）（在Spring bean上通知方法的执行）。 虽然可以在不影响到Spring AOP核心API的情况下加入对成员变量拦截器支持，但Spring并没有实现成员变量拦截器。 如果你需要把对成员变量的访问和更新也作为通知的连接点，可以考虑其它语法的Java语言，例如AspectJ。 

Spring实现AOP的方法跟其他的框架不同。Spring并不是要尝试提供最完整的AOP实现（尽管Spring AOP有这个能力）， 相反的，它其实侧重于提供一种AOP实现和Spring IoC容器的整合，用于帮助解决在企业级开发中的常见问题。 

因此，Spring AOP通常都和Spring IoC容器一起使用。 Aspect使用普通的bean定义语法（尽管Spring提供了强大的“自动代理（autoproxying）”功能）： 与其他AOP实现相比这是一个显著的区别。有些事使用Spring AOP是无法轻松或者高效的完成的，比如说通知一个细粒度的对象。 这种时候，使用AspectJ是最好的选择。不过经验告诉我们： 于大多数在J2EE应用中遇到的问题，只要适合AOP来解决的，Spring AOP都没有问题，Spring AOP提供了一个非常好的解决方案。 

Spring AOP从来没有打算通过提供一种全面的AOP解决方案来取代AspectJ。 我们相信无论是基于代理（proxy-based ）的框架比如说Spring亦或是full-blown的框架比如说是AspectJ都是很有价值的，他们之间的关系应该是互补而不是竞争的关系。 Spring 2.0可以无缝的整合Spring AOP，IoC 和AspectJ，使得所有的AOP应用完全融入基于Spring的应用体系。 这样的集成不会影响Spring AOP API或者AOP Alliance API；Spring AOP保留了向下兼容性。接下来的一章会详细讨论Spring AOP API。 


##### Spring的AOP代理
Spring缺省使用J2SE 动态代理（dynamic proxies）来作为AOP的代理。这样任何接口都可以被代理。

Spring也支持使用CGLIB代理. 对于需要代理类而不是代理接口的时候CGLIB代理是很有必要的。 如果一个业务对象并没有实现一个接口，默认就会使用CGLIB。 作为面向接口编程的最佳实践，业务对象通常都会实现一个或多个接口。但也有可能会 强制使用CGLIB， 在这种情况（希望不常有）下，你可能需要通知一个没有在接口中声明的方法，或者需要传入一个代理对象给方法作为具体类型

在Spring 2.0之后，Spring可能会提供多种其他类型的AOP代理，包括了完整的生成类。这不会影响到编程模型。

##### Spring AOP APIs
Spring的切入点模型使得切入点可以独立于通知类型进行重用，这就使得针对不同 advice使用相同的pointcut成为可能。
org.springframework.aop.Pointcut 是最核心的接口，用来将 通知应用于特定的类和方法，完整的接口定义如下：
```java
public interface Pointcut {

    ClassFilter getClassFilter();

    MethodMatcher getMethodMatcher();

}

```
将Pointcut接口分成有利于重用类和方法匹配的两部分，以及进行更细粒度的操作组合（例如与另一个方法匹配实现执行一个“或操作”）。

ClassFilter接口用来将切入点限定在一个给定的类集合中。如果matches()方法总是返回true，所有目标类都将被匹配：
```java
public interface ClassFilter {
	/**
	 * Should the pointcut apply to the given interface or target class?
	 * @param clazz the candidate target class
	 * @return whether the advice should apply to the given target class
	 */
	boolean matches(Class<?> clazz);

	/**
	 * Canonical instance of a ClassFilter that matches all classes.
	 */
	ClassFilter TRUE = TrueClassFilter.INSTANCE;
}
```
MethodMatcher接口通常更重要，完整的接口定义如下：
```
public interface MethodMatcher {

    boolean matches(Method m, Class targetClass);

    boolean isRuntime();

    boolean matches(Method m, Class targetClass, Object[] args);
}
```
matches(Method, Class)方法用来测试这个切入点是否匹配目标类的指定方法。这将在AOP代理被创建的时候执行，这样可以避免在每次方法调用的时候都执行。如果matches(Method, Class )对于一个给定的方法返回true，并且isRuntime() 也返回true，那么matches(Method, Class , Object[])将在每个方法调用的时候被调用。这使得切入点在通知将被执行前可以查看传入到方法的参数。

大多数MethodMatcher是静态的，这意味着isRuntime()方法返回 false。在这种情况下，matches(Method, Class , Object[])(3个参数的方法)永远不会被调用。

**切入点运算**
Spring在切入点上支持以下运算： 或和与。

*或运算*表示只需有一个切入点被匹配就执行方法。

*与运算*表示所有的切入点都匹配的情况下才执行。

通常或运算要更有用一些。

切入点可以使用org.springframework.aop.support.Pointcuts类中的静态方法来编写，或者使用同一个包内的ComposablePointcut类。不过使用AspectJ切入点表达式通常会更简单一些。

##### 切入点的实现
Spring提供了一些很方便的切入点实现。一些是开箱即用的，其它的是切入点应用规范的子集。

`静态切入点：`静态切入点基于方法和目标类进行切入点判断而不考虑方法的参数。在多数情况下，静态切入点是高效的、最好的选择。 Spring只在第一次调用方法时执行静态切入点：以后每次调用这个方法时就不需要再执行。

* 正则表达式切入点：一种显而易见的描述静态切入点的方式是使用正则表达式。包含Spring在内的一些AOP框架都支持这种方式。
org.springframework.aop.support.JdkRegexpMethodPointcut是通用正则表达式切入点，使用JDK 1.4+中的正则表达式支持。
使用JdkRegexpMethodPointcut类，可以提供一组模式字符串列表。 如果其中任意一个模式匹配，切入点将被解析为true。 （实际上就是这些切入点的或集。）

The usage is shown below:
```java
<bean id="settersAndAbsquatulatePointcut"
        class="org.springframework.aop.support.JdkRegexpMethodPointcut">
    <property name="patterns">
        <list>
            <value>.*set.*</value>
            <value>.*absquatulate</value>
        </list>
    </property>
</bean>
```
Spring提供了一个方便的类 RegexpMethodPointcutAdvisor， 它也允许我们引用一个通知（记住这里一个通知可以是拦截器，前置通知（before advice），异常通知（throws advice）等类型中的一个）。 在背后，如果使用J2SE 1.4或者以上版本，Spring将使用JdkRegexpMethodPointcut，在之前版本的虚拟机上，Spring将退回去使用Perl5RegexpMethodPointcut。 可以通过设置perl5属性为true来强制使用Perl5RegexpMethodPointcut。使用RegexpMethodPointcutAdvisor可以简化织入，因为`一个bean可以同时作为切入点和advisor`，如下所示
```java
<bean id="settersAndAbsquatulateAdvisor"
        class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
    <property name="advice">
        <ref bean="beanNameOfAopAllianceInterceptor"/>
    </property>
    <property name="patterns">
        <list>
            <value>.*set.*</value>
            <value>.*absquatulate</value>
        </list>
    </property>
</bean>
```
**RegexpMethodPointcutAdvisor可以和任何通知类型一起使用**

* 属性驱动的切入点：一个重要的静态切入点是元数据驱动(metadata-driven)切入点。这使用元数据参数：特别是源代码级别的元数据。 

`动态切入点：`动态切入点比起静态切入点在执行时要消耗更多的资源。它们同时计算方法参数和静态信息。 这意味着它们必须在每次方法调用时都被执行；由于参数的不同，评估结果不能被缓存。
* 控制流切入点：动态切入点的常见例子是控制流切入点。Spring控制流切入点在概念上和AspectJ的cflow 切入点很相似， 虽然它的功能不如后者那么强大。（目前还不能让一个切入点在另外一个切入点所评估的连接点处执行）。 一个控制流切入点匹配当前的调用栈。例如，一个连接点被com.mycompany.web包内的一个 方法或者SomeCaller类调用，切入点就可能被激活。 控制流切入点是由org.springframework.aop.support.ControlFlowPointcut 类声明的。

**注意：**在执行时控制流切入点的开销是非常昂贵的，甚至与其它动态切入点比起来也是如此。在Java 1.4里，它的开销差不多是其它动态切入点的5倍；在Java 1.3中，这个比例甚至达到10倍。

##### 切入点的基类/超类
Spring提供了很多有用的切入点基础类来帮助你实现你自己的切入点。

因为静态切入点是最常用的，你可能会像下面那样继承StaticMethodMatcherPointcut。这只需要实现一个抽象方法 （虽然也可以覆盖其它方法来定制行为）：
```java
class TestStaticPointcut extends StaticMethodMatcherPointcut {

    public boolean matches(Method m, Class targetClass) {
        // return true if custom criteria match
    }
}
```
动态切入点也有很多基类。

你可以用Spring 1.0 RC2和更高版本里的自定义切入点及通知类型。

##### 自定义切入点
因为在Spring AOP中的切入点是Java类而不是语言的特性（后者像AspectJ中那样），所以可以声明自定义的切入点，不论是静态还是动态的。自定义切入点在Spring里可能很强大。即使这样我们仍推荐尽可能使用AspectJ切入点表达式语言。

##### Spring的通知(现在让我们看一下SPringAOP是怎样处理通知的。)

**通知的生命周期/Advice lifecycles**

每个通知都是一个Spring`bean`。一个通知实例既可以被所有被通知的对象共享，也可以被每个被通知对象独占。 这根据设置类共享（per-class）或基于实例（per-instance）的参数来决定。
 
类共享通知经常会被用到。它很适合用作通用的通知例如事务advisor。这些advisor不依赖于代理对象的状态也不会向代理对象添加新的状态；它们仅仅在方法和参数上起作用。 

基于实例的通知很适合用作导入器来支持混合类型。在这种情况下，通知向代理对象添加状态。

在同一个AOP代理里混合使用类共享和基于实例的通知是可能的。

**Spring的通知类型**

Spring提供了多种开箱即用的通知类型，而且它们也可以被扩展来支持任何通知类型。让我们先看看基本概念和标准的通知类型。

* Interception around advice/拦截around通知

  在Spring中最基础的通知类型是拦截around通知。
  
  Spring里使用方法拦截的around通知兼容AOP联盟接口。实现around通知的MethodInterceptor应当实现下面的接口：
  ```
  public interface MethodInterceptor extends Interceptor {
  
      Object invoke(MethodInvocation invocation) throws Throwable;
  }
  ```
  invoke()方法的MethodInvocation参数暴露了被调用的方法,目标连接点,AOP代理以及传递给方法的参数。invoke()方法应该返回调用的结果：即连接点的返回值。
              
  一个简单的MethodInterceptor实现看起来像下面这样：
  ```
  public class DebugInterceptor implements MethodInterceptor {
  
      public Object invoke(MethodInvocation invocation) throws Throwable {
          System.out.println("Before: invocation=[" + invocation + "]");
          Object rval = invocation.proceed();
          System.out.println("Invocation returned");
          return rval;
      }
  }
  ```
  注意对MethodInvocation中proceed()方法的调用。这个方法 继续运行指向连接点的拦截器链并返回proceed()的结果。然而，一个类似任何环绕通知的MethodInterceptor， 可以返回一个不同的值或者抛出一个异常而不是调用proceed方法。但除非你有很好的理由，否则不要考虑这样做！

* Before advice/前置通知

    一个更简单的通知类型是before通知。它不需要MethodInvocation对象，因为它只是在进入方法之前被调用。
      
    前置通知/Before advice的一个主要优点是它不需要调用proceed() 方法，因此就不会发生 无意间运行拦截器链失败的情况。
    
    MethodBeforeAdvice接口被显示在下面。（Spring的API设计能够为类中的成员变量提供前置通知，虽然这可以把通用对象应用到成员变量拦截上，但看起来Spring并不打算实现这个功能。）
  ```java
  public interface MethodBeforeAdvice extends BeforeAdvice {
  
      void before(Method m, Object[] args, Object target) throws Throwable;
  }
  ```
  注意返回值的类型是void。前置通知可以在连接点执行之前插入自定义行为，但是不能修改连接点的返回值。如果一个前置通知抛出异常，这将中止拦截器链的进一步执行。 异常将沿着拦截器链向回传播。如果异常是非强制检查的（unchecked）或者已经被包含在被调用方法的签名中（译者：即出现在方法声明的throws子句中），它将被直接返回给客户端； 否则它将由AOP代理包装在一个非强制检查异常中返回。
  
  An example of a before advice in Spring, which counts all method invocations:
  ```java
  
  public class CountingBeforeAdvice implements MethodBeforeAdvice {
  
      private int count;
  
      public void before(Method m, Object[] args, Object target) throws Throwable {
          ++count;
      }
  
      public int getCount() {
          return count;
      }
  }
  
  ```
  提示：前置通知可以和任何切入点一起使用。

* 异常通知/Throws advice

  如果连接点抛出异常，异常通知（throws advice）将在连接点返回后被调用。 Spring提供类型检查的异常通知，这意味着org.springframework.aop.ThrowsAdvice接口不包含任何方法：它只是一个标记接口用来标识 所给对象实现了一个或者多个针对特定类型的异常通知方法。这些方法应当满足下面的格式 
  `afterThrowing([Method, args, target], subclassOfThrowable)`
  
  只有最后一个参数是必须的。因此异常通知方法对方法及参数的需求，方法的签名将从一到四个参数之间变化。 下面是一些throws通知的例子。
  
  当一个RemoteException（包括它的子类）被抛出时，下面的通知会被调用：
  
  ```java
  public class RemoteThrowsAdvice implements ThrowsAdvice {
  
      public void afterThrowing(RemoteException ex) throws Throwable {
          // Do something with remote exception
      }
  }
  ```
  当一个ServletException被抛出，下面的通知将被调用。 和上面的通知不同，它声明了4个参数，因此它可以访问被调用的方法，方法的参数以及目标对象：
  ```java
  public class ServletThrowsAdviceWithArguments implements ThrowsAdvice {
  
      public void afterThrowing(Method m, Object[] args, Object target, ServletException ex) {
          // Do something with all arguments
      }
  }
  ```
  最后一个例子说明怎样在同一个类里使用两个方法来处理 RemoteException和ServletException。可以在一个类里组合任意数量的异常通知方法。
  ```java
  public static class CombinedThrowsAdvice implements ThrowsAdvice {
  
      public void afterThrowing(RemoteException ex) throws Throwable {
          // Do something with remote exception
      }
  
      public void afterThrowing(Method m, Object[] args, Object target, ServletException ex) {
          // Do something with all arguments
      }
  }
  ```
  提示：异常通知可以和任何切入点一起使用

* 后置通知/After Returning advice

  Spring中的一个后置通知（After Returning advice）必须实现 org.springframework.aop.AfterReturningAdvice 接口，像下面显示的那样：
  ```java
  public interface AfterReturningAdvice extends Advice {
  
      void afterReturning(Object returnValue, Method m, Object[] args, Object target)
              throws Throwable;
  }
  ```
  一个后置通知可以访问返回值（但不能进行修改），被调用方法，方法参数以及目标对象。
  
  下面的后置通知计算所有运行成功（没有抛出异常）的方法调用：
  ```java
  public class CountingAfterReturningAdvice implements AfterReturningAdvice {
  
      private int count;
  
      public void afterReturning(Object returnValue, Method m, Object[] args, Object target)
              throws Throwable {
          ++count;
      }
  
      public int getCount() {
          return count;
      }
  }
  ```
  这个通知不改变执行路线。如果通知抛出异常，异常将沿着拦截器链返回（抛出）而不是返回被调用方法的执行结果。
  
  提示：后置通知可以和任何切入点一起使用。

* Introduction advice/引入通知

  Spring 把引入通知（introduction advice）作为一种特殊的拦截通知进行处理。
  
  引入通知需要一个IntroductionAdvisor, 和一个IntroductionInterceptor, 后者实现下面的接口：
  ```java
  public interface IntroductionInterceptor extends MethodInterceptor {
  
      boolean implementsInterface(Class intf);
  }
  ```
  invoke() 方法，继承了AOP联盟MethodInterceptor 接口，必须确保实现引入： 这里的意思是说，如果被调用的方法位于一个已经被引入接口里，这个引入拦截器将负责完成对这个方法的调用--因为后者不能调用proceed()方法。
  
  引入通知不能和任何切入点一起使用，因为它是应用在类级别而不是方法级别。 你可以通过IntroductionAdvisor来使用引入通知，这个接口包括下面的方法：
  ```java
  public interface IntroductionAdvisor extends Advisor, IntroductionInfo {
  
      ClassFilter getClassFilter();
  
      void validateInterfaces() throws IllegalArgumentException;
  }
  
  public interface IntroductionInfo {
  
      Class[] getInterfaces();
  }
  ```
  这里没有MethodMatcher接口,因此也就没有 Pointcut接口与引入通知相关联。这里只进行类过滤。
  
  getInterfaces()方法返回这个advisor所引入的接口。
  
  validateInterfaces()方法将被内部使用来查看被引入的接口是否能够由配置的IntroductionInterceptor来实现。
  
  让我们看看从Spring测试集里拿来的一个简单例子。让我们假设我们希望把下面的接口引入给一个或者多个对象：
  
  ```java
  public interface Lockable {
      void lock();
      void unlock();
      boolean locked();
  }
  ```
  这里描述了一个混合类型。我们希望不论原本对象是什么类型，都把这个被通知对象转换为Lockable接口并可以调用lock 和unlock 方法。 如果我们调用lock() 方法，我们希望所有的setter 方法抛出一个LockedException异常。这样我们就可以加入一个方面来确保对象在得到通知之前是不可修改的：一个关于AOP的好例子。
  
  首先，我们需要一个IntroductionInterceptor来做粗活。这里，我们扩展了 org.springframework.aop.support.DelegatingIntroductionInterceptor这个方便的类。我们能够直接实现 IntroductionInterceptor接口，但在这个例子里使用DelegatingIntroductionInterceptor是最好的选择。 
  
  DelegatingIntroductionInterceptor设计为把一个引入托管给一个实现这个接口的类， 这通过隐藏拦截的使用来实现。托管可以被设置到任何具有构造器方法的类；这里使用缺省托管(即使用无参构造器)。 因此在下面这个例子里，托管者将是DelegatingIntroductionInterceptor的子类 LockMixin。 当一个托管实现被提供，DelegatingIntroductionInterceptor实例将查找托管所实现的所有接口 （除了IntroductionInterceptor之外），并为这些接口的介绍提供支持。子类例如LockMixin 可以调用suppressInterface(Class intf) 方法来禁止那些不应该被暴露的接口。 然而，不论IntroductionInterceptor支持多少接口， IntroductionAdvisor的使用将控制哪些接口真正被暴露。 一个被引入的接口将覆盖目标对象实现的相同接口. 
  
  这样LockMixin就继承了DelegatingIntroductionInterceptor并实现了Lockable 接口本身。 这里父类会自动选择Lockable接口并提供引入支持，因此我们不需要配置它。用这种方法我们能够介绍任意数量的接口。
  
  注意locked实例变量的用法。这有效地向目标对象增加了额外状态。
  
  ```java
  public class LockMixin extends DelegatingIntroductionInterceptor implements Lockable {
  
      private boolean locked;
  
      public void lock() {
          this.locked = true;
      }
  
      public void unlock() {
          this.locked = false;
      }
  
      public boolean locked() {
          return this.locked;
      }
  
      public Object invoke(MethodInvocation invocation) throws Throwable {
          if (locked() && invocation.getMethod().getName().indexOf("set") == 0) {
              throw new LockedException();
          }
          return super.invoke(invocation);
      }
  
  }
  ```
  覆盖invoke()方法通常是不必要的：DelegatingIntroductionInterceptor里面已经包含了一个实现--如果一个方法被引入，这个实现将调用实际的托管方法，否则它将直接处理连接点--通常这已经足够了。在当前这个例子里，我们需要增加一个检查：如果处于加锁（locked）状态，没有setter方法可以被调用。 
     
  引入处理器的要求是很简单的。它的全部要求只是保持一个特定的LockMixin实例， 并说明被通知的接口--在这个例子里，只有一个Lockable接口。 一个更复杂的例子也许会获取一个介绍拦截器的引用（后者可以被定义为一个prototype）： 在这种情况下，不需要对LockMixin进行相关配置，因此我们可以简单的用new关键字来创建它。 
  ```java
  public class LockMixinAdvisor extends DefaultIntroductionAdvisor {
  
      public LockMixinAdvisor() {
          super(new LockMixin(), Lockable.class);
      }
  }
  ```
  我们可以很容易应用这个advisor：它不需要配置。（然而，下面是必须记住的：不可以在没有IntroductionAdvisor的情况下使用IntroductionInterceptor。） 对于通常的引入advisor必须是基于实例的，因为它是有状态的。因此，对于每个被通知对象我们需要一个不同 实例的LockMixinAdvisor和LockMixin。这种情况下advisor保存了被通知对象的部分状态。 
  
  我们能够通过使用Advised.addAdvisor() 的编程方式来应用advisor，或者像其它advisor那样（也是推荐的方式）在XML里进行配置。全部的代理创建选择（包括“自动代理创建器”）将在下面进行讨论， 看看如何正确地处理introduction和有状态混合类型。


##### 使用ProxyFactoryBean创建AOP代理
如果你正在使用Spring IoC容器（即ApplicationContext或BeanFactory）来管理你的业务对象--这正是你应该做的--你也许会想要使用Spring中关于AOP的FactoryBean。（记住使用工厂bean引入一个间接层之后，我们就可以创建不同类型的对象了）。

在Spring里创建一个AOP代理的基本方法是使用org.springframework.aop.framework.ProxyFactoryBean。这个类对应用的切入点和通知提供了完整的控制能力（包括它们的应用顺序）。然而如果你不需要这种控制，你会喜欢更简单的方式。

**基础**

像其它的FactoryBean实现一样，ProxyFactoryBean引入了一个间接层。如果你定义一个名为foo的ProxyFactoryBean， 引用foo的对象看到的将不是ProxyFactoryBean实例本身，而是一个ProxyFactoryBean实现里getObject() 方法所创建的对象。 这个方法将创建一个AOP代理，它包装了一个目标对象。

使用ProxyFactoryBean或者其它IoC相关类带来的最重要的好处之一就是创建AOP代理，这意味着通知和切入点也可以由IoC来管理。这是一个强大的功能并使得某些特定的解决方案成为可能， 而这些用其它AOP框架很难做到。例如，一个通知也许本身也要引用应用程序对象（不仅仅是其它AOP框架中也可以访问的目标对象），这令你可以从依赖注射的可拔插特性中获益。

通常情况下Spring提供了大多数的FactoryBean实现，ProxyFactoryBean类本身也是一个JavaBean。它的属性被用来： 

* 指定你希望代理的目标对象

* 指定是否使用CGLIB

一些主要属性从org.springframework.aop.framework.ProxyConfig里继承下来（这个类是Spring里所有AOP代理工厂的父类）。这些主要属性包括： 

* proxyTargetClass：这个属性为true时，目标类本身被代理而不是目标类的接口。如果这个属性值被设为true，CGLIB代理将被创建（可以参看下面名为第 7.5.3 节 “基于JDK和CGLIB的代理”的章节）。 

* optimize：用来控制通过CGLIB创建的代理是否使用激进的优化策略。除非完全了解AOP代理如何处理优化，否则不推荐用户使用这个设置。目前这个属性仅用于CGLIB代理；对于JDK动态代理（缺省代理）无效。 

* frozen：用来控制代理工厂被配置之后，是否还允许修改通知。缺省值为false（即在代理被配置之后，不允许修改代理的配置）。 

* exposeProxy：决定当前代理是否被保存在一个ThreadLocal中以便被目标对象访问。（目标对象本身可以通过MethodInvocation来访问，因此不需要ThreadLocal。） 如果个目标对象需要获取代理而且exposeProxy属性被设为true，目标对象可以使用AopContext.currentProxy()方法。 

ProxyFactoryBean中需要说明的其它属性包括： 

* proxyInterfaces：需要代理的接口名的字符串数组。如果没有提供，将为目标类使用一个CGLIB代理（也可以查看下面名为第 7.5.3 节 “基于JDK和CGLIB的代理”的章节）。 

* interceptorNames：Advisor的字符串数组，可以包括拦截器或其它通知的名字。顺序是很重要的，排在前面的将被优先服务。就是说列表里的第一个拦截器将能够第一个拦截调用。 

这里的名字是当前工厂中bean的名字，包括父工厂中bean的名字。这里你不能使用bean的引用因为这会导致ProxyFactoryBean忽略通知的单例设置。 

你可以把一个拦截器的名字加上一个星号作为后缀（*）。这将导致这个应用程序里所有名字以星号之前部分开头的advisor都被应用。你可以在第 7.5.6 节 “使用“全局”advisor” 发现一个使用这个特性的例子。 

* 单例：工厂是否应该返回同一个对象，不论方法getObject()被调用的多频繁。多个FactoryBean实现都提供了这个方法。缺省值是true。如果你希望使用有状态的通知--例如，有状态的mixin--可以把单例属性的值设置为false来使用原型通知。 

**基于JDK和CGLIB的代理**

这个小节作为说明性文档，解释了对于一个目标对象（需要被代理的对象），ProxyFactryBean是如何决定究竟创建一个基于JDK还是CGLIB的代理的。 

注意：ProxyFactoryBean需要创建基于JDK还是CGLIB代理的具体行为在版本1.2.x和2.0中有所不同。现在ProxyFactoryBean在关于自动检测接口方面使用了与TransactionProxyFactoryBean相似的语义。 

如果一个需要被代理的目标对象的类（后面将简单地称它为目标类）没有实现任何接口，那么一个基于CGLIB的代理将被创建。这是最简单的场景，因为JDK代理是基于接口的，没有接口意味着没有使用JDK进行代理的可能。 在目标bean里将被插入探测代码，通过interceptorNames属性给出了拦截器的列表。注意一个基于CGLIB的代理将被创建即使ProxyFactoryBean的proxyTargetClass属性被设置为false。 （很明显这种情况下对这个属性进行设置是没有意义的，最好把它从bean的定义中移除，因为虽然这只是个多余的属性，但在许多情况下会引起混淆。） 

如果目标类实现了一个（或者更多）接口，那么创建代理的类型将根据ProxyFactoryBean的配置来决定。 

如果ProxyFactoryBean的proxyTargetClass属性被设为true，那么一个基于CGLIB的代理将创建。这样的规定是有意义的，遵循了最小惊讶法则（保证了设定的一致性）。 甚至当ProxyFactoryBean的proxyInterfaces属性被设置为一个或者多个全限定接口名，而proxyTargetClass属性被设置为true仍然将实际使用基于CGLIB的代理。 

如果ProxyFactoryBean的proxyInterfaces属性被设置为一个或者多个全限定接口名，一个基于JDK的代理将被创建。被创建的代理将实现所有在proxyInterfaces属性里被说明的接口；如果目标类实现了全部在proxyInterfaces属性里说明的接口以及一些额外接口，返回的代理将只实现说明的接口而不会实现那些额外接口。 

如果ProxyFactoryBean的proxyInterfaces属性没有被设置，但是目标类实现了一个（或者更多）接口，那么ProxyFactoryBean将自动检测到这个目标类已经实现了至少一个接口， 一个基于JDK的代理将被创建。被实际代理的接口将是目标类所实现的全部接口；实际上，这和在proxyInterfaces属性中列出目标类实现的每个接口的情况是一样的。然而，这将显著地减少工作量以及输入错误的可能性。 

 **对接口进行代理**

让我们看一个关于ProxyFactoryBean的简单例子。这个例子涉及： 

* 一个将被代理的目标bean。在下面的例子里这个bean是“personTarget”。

* 被用来提供通知的一个advisor和一个拦截器。

* 一个AOP代理bean的定义，它说明了目标对象（personTarget bean）以及需要代理的接口，还包括需要被应用的通知。

```java
<bean id="personTarget" class="com.mycompany.PersonImpl">
    <property name="name" value="Tony"/>
    <property name="age" value="51"/>
</bean>

<bean id="myAdvisor" class="com.mycompany.MyAdvisor">
    <property name="someProperty" value="Custom string property value"/>
</bean>

<bean id="debugInterceptor" class="org.springframework.aop.interceptor.DebugInterceptor">
</bean>

<bean id="person"
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="proxyInterfaces" value="com.mycompany.Person"/>

    <property name="target" ref="personTarget"/>
    <property name="interceptorNames">
        <list>
            <value>myAdvisor</value>
            <value>debugInterceptor</value>
        </list>
    </property>
</bean>
```
注意interceptorNames属性接受一组字符串：当前工厂中拦截器或advisorbean的名字。拦截器，advisor，前置, 后置和异常通知对象都可以在这里被使用。这里advisor的顺序是很重要的。

**注意：**你也许很奇怪为什么这个列表不保存bean的引用。理由是如果ProxyFactoryBean的singleton属性被设置为false，它必须返回独立的代理实例。如果任何advisor本身是一个原型，则每次都返回一个独立实例，因此它必须能够从工厂里获得原型的一个实例；保存一个引用是不够的。 

上面“person” bean的定义可以被用来取代一个Person接口的实现，就像下面这样：
```java
Person person = (Person) factory.getBean("person");
```
在同一个IoC上下文中其它的bean可以对这个bean有基于类型的依赖，就像对一个普通的Java对象那样：
```java
<bean id="personUser" class="com.mycompany.PersonUser">
    <property name="person"><ref bean="person"/></property>
</bean>
```
这个例子里的PersonUser类将暴露一个类型为Person的属性。就像我们关心的那样，AOP代理可以透明地取代一个“真实”的person接口实现。然而，它的类将是一个动态代理类。 它可以被转型成Advised接口（将在下面讨论）。

就像下面这样，你可以使用一个匿名内部bean来隐藏目标和代理之间的区别。仅仅ProxyFactoryBean的定义有所不同；通知的定义只是由于完整性的原因而被包括进来：
```java
<bean id="myAdvisor" class="com.mycompany.MyAdvisor">
    <property name="someProperty" value="Custom string property value"/>
</bean>

<bean id="debugInterceptor" class="org.springframework.aop.interceptor.DebugInterceptor"/>

<bean id="person" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="proxyInterfaces" value="com.mycompany.Person"/>
    <!-- Use inner bean, not local reference to target -->
    <property name="target">
        <bean class="com.mycompany.PersonImpl">
            <property name="name" value="Tony"/>
            <property name="age" value="51"/>
        </bean>
    </property>
    <property name="interceptorNames">
        <list>
            <value>myAdvisor</value>
            <value>debugInterceptor</value>
        </list>
    </property>
</bean>
```
对于只需要一个Person类型对象的情况，这是有好处的：如果你希望阻止应用程序上下文的用户获取一个指向未通知对象的引用或者希望避免使用Spring IoC 自动织入 时的混淆。 按理说ProxyFactoryBean定义还有一个优点是它是自包含的。然而，有时能够从工厂里获取未通知的目标也是一个优点：例如，在某些测试场景里。

**对类进行代理**

如果你需要代理一个类而不是代理一个或是更多接口，那么情况将是怎样？

想象在我们上面的例子里，不存在Person接口：我们需要通知一个叫做Person的类，它没有实现任何业务接口。在这种情况下，你可以配置Spring使用CGLIB代理，而不是动态代理。 这只需简单地把上面ProxyFactoryBean的proxyTargetClass属性设为true。虽然最佳方案是面向接口编程而不是类，但在与遗留代码一起工作时，通知没有实现接口的类的能力是非常有用的。（通常情况下，Spring没有任何规定。它只是让你很容易根据实际情况选择最好的解决方案，避免强迫使用特定方式）。 

也许你希望你能够在任何情况下都强制使用CGLIB，甚至在你使用接口的时候也这样做。

CGLIB通过在运行时生成一个目标类的子类来进行代理工作。Spring配置这个生成的子类对原始目标对象的方法调用进行托管：子类实现了装饰器（Decorator）模式，把通知织入。

CGLIB的代理活动应当对用户是透明的。然而，有一些问题需要被考虑：

* Final方法不可以被通知，因为它们不能被覆盖。

* 你需要在你的类路径里有CGLIB 2的库；使用动态代理的话只需要JDK。没有必要将CGLIB添加到您的类路径中。 从Spring 3.2开始，CGLIB被重新包装并包含在spring核心JAR中。 换句话说，基于CGLIB的AOP将像JDK动态代理一样工作“开箱即用”,无需在类路径里添加cglib库。

在CGLIB代理和动态代理之间的速度差别是很小的。在Spring 1.0中，动态代理会快一点点。但这点可能在将来被改变。这种情况下，选择使用何种代理时速度不应该成为决定性的理由。 

**使用“全局”advisor**

通过在一个拦截器名后添加一个星号，所有bean名字与星号之前部分相匹配的通知都将被加入到advisor链中。这让你很容易添加一组标准的“全局”advisor：
```java
bean id="proxy" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target" ref="service"/>
    <property name="interceptorNames">
        <list>
            <value>global*</value>
        </list>
    </property>
</bean>

<bean id="global_debug" class="org.springframework.aop.interceptor.DebugInterceptor"/>
<bean id="global_performance" class="org.springframework.aop.interceptor.PerformanceMonitorInterceptor"/>
```
**简化代理定义**

你也许需要许多相似的代理定义，特别是定义事务性代理的时候。使用父子bean定义，以及内部bean定义，可以让代理定义大大得到极大的简化。

首先从父bean开始，为代理bean创建bean定义模版：
```java
<bean id="txProxyTemplate" abstract="true"
        class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
    <property name="transactionManager" ref="transactionManager"/>
    <property name="transactionAttributes">
        <props>
            <prop key="*">PROPAGATION_REQUIRED</prop>
        </props>
    </property>
</bean>
```
这个bean本身将永远不会被初始化，所以实际上是不完整的。而后每个需要创建的代理都是这个bean定义的子bean定义，它们把代理的目标类包装为一个内部bean定义，因为目标对象本身将不会被单独使用。 
```java
<bean id="myService" parent="txProxyTemplate">
    <property name="target">
        <bean class="org.springframework.samples.MyServiceImpl">
        </bean>
    </property>
</bean>
```
当然你可以覆盖从模版中继承的属性，例如在下面这个例子里的事务传播设置：
```java
<bean id="mySpecialService" parent="txProxyTemplate">
    <property name="target">
        <bean class="org.springframework.samples.MySpecialServiceImpl">
        </bean>
    </property>
    <property name="transactionAttributes">
        <props>
            <prop key="get*">PROPAGATION_REQUIRED,readOnly</prop>
            <prop key="find*">PROPAGATION_REQUIRED,readOnly</prop>
            <prop key="load*">PROPAGATION_REQUIRED,readOnly</prop>
            <prop key="store*">PROPAGATION_REQUIRED</prop>
        </props>
    </property>
</bean>
```
要注意上面例子中我们已经明确地通过设定abstract属性把父bean定义标注为abstract，在前面的章节里有描述，这样它实际上不能被初始化。 缺省情况下应用程序上下文（不仅仅是bean工厂）将预先初始化所有的实例为单例。因此下面这点是很重要的（至少对于单例bean来说），如果你有一个（父）bean定义你希望仅仅作为模版使用，而这个定义说明了一个类，你必须把abstract参数设置为true，否则应用程序上下文将试图预先初始化它。

**使用ProxyFactory通过编程创建AOP代理**

使用Spring通过编程创建AOP代理是很容易的。这使你可以使用Spring AOP而不必依赖于Spring IoC。

下面的清单显示了如何使用一个拦截器和一个advisor来为一个目标对象来创建一个代理。目标对象实现的接口将被自动代理：
```java
ProxyFactory factory = new ProxyFactory(myBusinessInterfaceImpl);
factory.addInterceptor(myMethodInterc ptor);
factory.addAdvisor(myAdvisor);
MyBusinessInterface tb = (MyBusinessInterface) factory.getProxy();
```
第一步是创建一个类型为org.springframework.aop.framework.ProxyFactory的对象。你可以像上面例子里那样使用一个目标对象来创建它，或者在一个可选的构造器里说明需要被代理的接口。

你可以添加拦截器或advisor，并在ProxyFactory的生命周期里操作它们。如果你加入一个IntroductionInterceptionAroundAdvisor，你可以让代理实现额外的接口。

在ProxyFactory里也有很方便的方法（继承自AdvisedSupport）允许你加入其它的通知类型例如前置和异常通知。AdvisedSupport是ProxyFactory 和ProxyFactoryBean的共同父类。

提示：在大多数应用程序里，把AOP代理的创建和IoC框架集成是最佳实践。通常情况下我们推荐你在Java代码外进行AOP的配置。

**操作被通知对象**

在创建了AOP代理之后，你能够使用org.springframework.aop.framework.Advised接口对它们进行管理。任何AOP代理都能够被转型为这个接口，不论它实现了哪些其它接口。这个接口包括下面的方法：
```java
Advisor[] getAdvisors();

void addAdvice(Advice advice) throws AopConfigException;

void addAdvice(int pos, Advice advice) throws AopConfigException;

void addAdvisor(Advisor advisor) throws AopConfigException;

void addAdvisor(int pos, Advisor advisor) throws AopConfigException;

int indexOf(Advisor advisor);

boolean removeAdvisor(Advisor advisor) throws AopConfigException;

void removeAdvisor(int index) throws AopConfigException;

boolean replaceAdvisor(Advisor a, Advisor b) throws AopConfigException;

boolean isFrozen();
```
getAdvisors()方法将为每个已经被加入工厂的advisor，拦截器或者其它通知类型返回一个advisor。如果你曾经添加一个advisor，那么所返回的advisor将是你加入的对象。 如果你曾经加入一个拦截器或者其它通知类型，Spring将把它们包装在一个advisor里，后者使用一个永远返回true的切入点。因此如果你曾经加入一个MethodInterceptor， 返回的advisor将是一个DefaultPointcutAdvisor，它可以返回你加入的MethodInterceptor和一个匹配所有类和方法的切入点。 

addAdvisor()方法可以用来添加任何advisor。通常保存切入点和通知的advisor是DefaultPointcutAdvisor，它可以用于任何通知或切入点（但不包括引入类型）。

缺省情况下，你可以加入或移除advisor或者拦截器甚至当代理已经被创建之后。唯一的限制是无法加入或者移除一个引入advisor，因为工厂中获得的已有代理不能显示接口的改变（你可以通过从工厂里获取一个新的代理来避免这个问题）。

下面是一个简单的例子，它把一个AOP代理转型为Advised接口，检查并操作它的通知：
```java
Advised advised = (Advised) myObject;
Advisor[] advisors = advised.getAdvisors();
int oldAdvisorCount = advisors.length;
System.out.println(oldAdvisorCount + " advisors");

// Add an advice like an interceptor without a pointcut
// Will match all proxied methods
// Can use for interceptors, before, after returning or throws advice
advised.addAdvice(new DebugInterceptor());

// Add selective advice using a pointcut
advised.addAdvisor(new DefaultPointcutAdvisor(mySpecialPointcut, myAdvice));

assertEquals("Added two advisors", oldAdvisorCount + 2, advised.getAdvisors().length);
```
注意：在一个实际运行的系统里，修改一个业务对象上的通知是否明智是个问题，虽然无疑在某些情况下这样做是合理的。然而这在开发中是很有用的： 例如，在测试的时候。对于希望测试的方法调用，有时我发现把测试代码加入到一个拦截器或者其它通知里是非常有用的。（例如，通知可以与目标方法存在于同一个事务里，在把事务标记为回滚之前可以用SQL来检查数据库是否被正确的更新了。）

依赖于你怎样创建代理，你通常可以设置一个 frozen标志，在这种情况下 Advised的isFrozen()方法将返回true，任何增加或者移除通知的修改都会导致一个AopConfigException异常。 在某些情况下这种冻结被通知对象状态的能力是很有用的：例如，防止调用代码来移除一个进行安全检查的拦截器。在Spring 1.1中它也被用来允许激进优化，如果已经知道不需要运行时对通知进行修改的话。 

**使用“自动代理（autoproxy）”功能**

到目前为止我们已经考虑了如何使用ProxyFactoryBean或者类似的工厂bean来显式创建AOP代理。

Spring也允许我们使用“自动代理”的bean定义，可以自动对被选中的bean定义进行代理。这建立在Spring的“bean post processor”功能上，后者允许在容器加载时修改任何bean的定义。

在这个模型下，你在你的XML bean定义文件中建立一些特定的bean定义来配置自动代理功能。这允许你仅仅声明那些将被自动代理的适当目标：你不需要使用ProxyFactoryBean。 

有两种方式可以做到这点：

* 使用一个引用当前上下文中特定bean的自动代理创建器。

* 一个专用自动代理的创建需要被单独考虑；自动代理创建由源代码级别的元数据属性驱动。

_BeanNameAutoProxyCreator_

BeanNameAutoProxyCreator为名字匹配字符串或者通配符的bean自动创建AOP代理。
```java
<bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
    <property name="beanNames" value="jdk*,onlyJdk"/>
    <property name="interceptorNames">
        <list>
            <value>myInterceptor</value>
        </list>
    </property>
</bean>
```
和ProxyFactoryBean一样，这里有一个interceptorNames属性而不是一个拦截器的列表，这允许使用原型（prototype）advisor。这里的“拦截器”可以是advisor或任何通知类型。

与通常的自动代理一样，使用BeanNameAutoProxyCreator的主要目的是把相同的配置一致地应用到多个对象，并且使用最少量的配置。一个流行的选择是把声明式事务应用到多个对象上。

那些名字匹配的Bean定义，例如上面的例子里的“jdkMyBean”和“onlyJdk”，本身只是目标类的普通bean定义。一个AOP对象将被BeanNameAutoProxyCreator自动创建。 相同的通知将被应用到全部匹配的bean上。注意如果advisor被使用（而不是像上面例子里那样使用拦截器），对于不同bean可以应用不同的切入点。

_DefaultAdvisorAutoProxyCreator_

一个更加通用而且强大得多的自动代理创建器是DefaultAdvisorAutoProxyCreator。它自动应用当前上下文中适当的advisor，无需在自动代理advisor的bean定义中包括bean的名字。 比起BeanNameAutoProxyCreator，它提供了同样关于一致性配置的优点而避免了前者的重复性。 

使用这个功能将涉及：

* 说明一个 DefaultAdvisorAutoProxyCreator的bean定义

* 在同一个或者相关的上下文中说明任意数量的advisor。注意这些必须是advisor而不仅仅是拦截器或者其它通知。这点是必要的因为必须有一个切入点被评估，以便检查每个通知候选bean定义的合适性。 

DefaultAdvisorAutoProxyCreator将自动评估包括在每个advisor中的切入点，来看看它应当应用哪个（如果有的话）通知到每个业务对象（例如例子里的“businessObject1”和“businessObject2”）。

这意味着可以向每个业务对象应用任意数量的advisor。对于一个业务对象，如果没有任何advisor中的切入点匹配它的任何方法，这个对象将不会被代理。当为新的业务对象加入bean定义时，如果有必要它们将自动被代理。

通常自动代理的好处是它让调用者或者被依赖对象不能得到一个没有通知过的对象。在这个ApplicationContext上调用getBean("businessObject1")将返回一个AOP代理，而不是目标业务对象。（前面显示的“内部bean”也提供了同样的优点。） 
```java
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>

<bean class="org.springframework.transaction.interceptor.TransactionAttributeSourceAdvisor">
    <property name="transactionInterceptor" ref="transactionInterceptor"/>
</bean>

<bean id="customAdvisor" class="com.mycompany.MyAdvisor"/>

<bean id="businessObject1" class="com.mycompany.BusinessObject1">
    <!-- Properties omitted -->
</bean>

<bean id="businessObject2" class="com.mycompany.BusinessObject2"/>
```
如果你想要把相同的通知一致性地应用到许多业务对象上，DefaultAdvisorAutoProxyCreator是非常有用的。一旦框架的定义已经完成，你可以简单地加入新的业务对象而不必包括特定的代理配置。你也可以很容易的去掉额外的切面--例如，跟踪或者性能监视切面--仅仅对配置作很小的修改。 

DefaultAdvisorAutoProxyCreator支持过滤（通过使用一个命名约定让只有特定的advisor被评估，允许在同一个工厂里使用多个不同配置的AdvisorAutoProxyCreator）和排序。advisor可以实现org.springframework.core.Ordered接口来确保以正确的顺序被应用。 上面例子里的TransactionAttributeSourceAdvisor 有一个可配置的序号值；缺省情况下是没有排序的。

_AbstractAdvisorAutoProxyCreator_

这是DefaultAdvisorAutoProxyCreator的父类。如果在某些情况下框架提供的DefaultAdvisorAutoProxyCreator不能满足你的需要，你可以通过继承这个类来创建你自己的自动代理创建器。

**使用元数据驱动的自动代理**
  
一个非常重要的自动代理类型是由元数据驱动的。这提供了一种和.NET ServicedComponents相似的编程模型。作为使用类似EJB里的XML描述符的替代，对于事务管理和其它企业服务的配置都将被保存在源代码级别的属性里。 

在这个情况下，你使用DefaultAdvisorAutoProxyCreator和可以理解元数据属性的advisor。元数据被保存在候选advisor里的切入点部分，而不是在自动代理创建类本身。

这是一个DefaultAdvisorAutoProxyCreator的特殊例子，它本身没有什么特别。（元数据的相关代码保存在advisor内的切入点里，而不是AOP框架本身）。 

JPetStore示例应用程序的/attributes 目录显示了如何使用参数驱动的自动代理。在这个例子里，不需要使用TransactionProxyFactoryBean。因为使用了元数据相关的切入点， 所以简单在业务对象上定义事务属性就足够了。在/WEB-INF/declarativeServices.xml里的bean定义包括了下面的片断，注意这是通用的，可以被用在JPetStore以外的地方：
```java
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>

<bean class="org.springframework.transaction.interceptor.TransactionAttributeSourceAdvisor">
    <property name="transactionInterceptor" ref="transactionInterceptor"/>
</bean>

<bean id="transactionInterceptor"
        class="org.springframework.transaction.interceptor.TransactionInterceptor">
    <property name="transactionManager" ref="transactionManager"/>
    <property name="transactionAttributeSource">
        <bean class="org.springframework.transaction.interceptor.AttributesTransactionAttributeSource">
            <property name="attributes" ref="attributes"/>
        </bean>
    </property>
</bean>

<bean id="attributes" class="org.springframework.metadata.commons.CommonsAttributes"/>
```
DefaultAdvisorAutoProxyCreator bean定义（名字是不重要的，因此甚至可以在定义里省略它）将在当前应用程序上下文中查找所有合适的切入点。在这个例子里，TransactionAttributeSourceAdvisor类型的“transactionAdvisor”bean定义将应用到带有一个事务属性的类或方法上。 TransactionAttributeSourceAdvisor的构造器依赖于一个TransactionInterceptor。这个例子里通过自动织入解决了这个问题。AttributesTransactionAttributeSource依赖于一个org.springframework.metadata.Attributes接口的实现。 在这个代码片断里，“attributes”bean使用Jakarta Commons Attributes API来获取属性信息以满足这个要求。（应用程序代码必须已经使用Commons Attribut来es的编译任务编译过了。） 

JPetStore示例应用程序的 /annotation 目录包括了一个由JDK 1.5+注解驱动的自动代理的模拟例子。下面的配置允许自动检测Spring的Transactional注解，这可以为包含注解的bean提供隐式代理： 
```java
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>

<bean class="org.springframework.transaction.interceptor.TransactionAttributeSourceAdvisor">
    <property name="transactionInterceptor" ref="transactionInterceptor"/>
</bean>

<bean id="transactionInterceptor"
        class="org.springframework.transaction.interceptor.TransactionInterceptor">
    <property name="transactionManager" ref="transactionManager"/>
    <property name="transactionAttributeSource">
        <bean class="org.springframework.transaction.annotation.AnnotationTransactionAttributeSource"/>
    </property>
</bean>
```
这里定义的TransactionInterceptor依赖于一个PlatformTransactionManager定义，后者没有被包括在这个通用的文件里（虽然它可以被包括在这里）因为它在应用程序的事务需求规范中指定（在这个例子里使用JTA，而在其它情况下，可以是Hibernate，JDO或者JDBC）： 
```java
<bean id="transactionManager" class="org.springframework.transaction.jta.JtaTransactionManager"/>
```
**提示：**如果你只需要声明式事务管理，使用这些通用的XML定义将导致Spring自动代理所有带有事务属性的类或者方法。你将不需要直接使用AOP工作， 这个编程模型和.NET的ServicedComponents相似。

这个架构是可以扩展的。可以在自定义属性的基础上进行自动代理。你所需要做的是：

* 定义你自己的自定义属性

* 使用必要的通知说明一个advisor，也包括一个切入点，后者可以被类或者方法上的自定义属性触发。你也许能够使用已有的通知，而仅仅实现一个能够处理自定义属性的静态切入点。 

可以让这些advisor对于每个被通知对象（例如，mixins）都是唯一的：仅仅需要在bean定义中被定义为原型而不是单例。例如，在上面所显示的Spring测试集中的LockMixin引入拦截器可以和一个属性驱动的切入点联合定位一个mixin， 像这里显示的这样。我们使用通用的DefaultPointcutAdvisor，使用JavaBean属性进行配置： 
```
<bean id="lockMixin" class="test.mixin.LockMixin" scope="prototype"/>

<bean id="lockableAdvisor" class="org.springframework.aop.support.DefaultIntroductionAdvisor"
        scope="prototype">
    <constructor-arg ref="lockMixin"/>
</bean>
```
如果参数相关的切入点匹配anyBean或其它bean定义里的任何方法，mixin将被应用。注意lockMixin和lockableAdvisor的定义都是原型。myAttributeAwarePointcut切入点可以是个单例，因为它没有为单个被通知对象保持状态。

**使用TargetSources**

Spring提供了TargetSource的概念，由org.springframework.aop.TargetSource接口进行描述。这个接口负责返回一个实现连接点的“目标对象（target object）”。每当AOP代理处理一个方法调用时都会向TargetSource的实现请求一个目标实例。 

使用Spring AOP的开发者通常不需要直接和TargetSource打交道，但这提供了一种强大的方式来支持池化（pooling），热交换（hot swappable）和其它高级目标。例如，一个使用池来管理实例的TargetSource可以为每个调用返回一个不同的目标实例。 

如果你不指定一个TargetSource，一个缺省实现将被使用，它包装一个本地对象。对于每次调用它将返回相同的目标（像你期望的那样）。

让我们看看Spring提供的标准目标源（target source）以及如何使用它们。

**提示：**当使用一个自定义的目标源，你的目标通常需要是一个原型而不是一个单例的bean定义。这允许Spring在必要时创建新的目标实例。

_热交换目标源_

org.springframework.aop.target.HotSwappableTargetSource允许当调用者保持引用的时候，切换一个AOP代理的目标。 

修改目标源的目标将立即生效。 HotSwappableTargetSource是线程安全的。

你可以通过HotSwappableTargetSource的 swap()方法来改变目标，就像下面那样： 
```java
HotSwappableTargetSource swapper = (HotSwappableTargetSource) beanFactory.getBean("swapper");
Object oldTarget = swapper.swap(newTarget);
```
所需的XML定义看起来像下面这样：
```java
<bean id="initialTarget" class="mycompany.OldTarget"/>

<bean id="swapper" class="org.springframework.aop.target.HotSwappableTargetSource">
    <constructor-arg ref="initialTarget"/>
</bean>

<bean id="swappable" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="targetSource" ref="swapper"/>
</bean>

```
上面的swap()调用修改了swappable bean的目标。保持对这个bean的引用的客户将不知道发生了这个修改，但是将可以立即点击新的目标。

这个例子没有添加任何通知--也不必为使用一个TargetSource添加任何通知--当然任何TargetSource都可以与任意通知联合使用。 

_池化目标源_

使用一个池化目标源提供了和无状态session EJB类似的编程模型，它维护一个包括相同实例的池，方法调用结束后将把对象释放回池中。

Spring池化和SLSB池化之间的一个决定性区别是Spring池化功能可以用于任何POJO。就像Spring通常情况下那样，这个服务是非侵入式的。

Spring为Commons Pool 2.2提供了开箱即用的支持，它提供了一个相当有效的池化实现。 您需要应用程序的类路径中存在commons-pool Jar才能使用此功能。 也可以将org.springframework.aop.target.AbstractPoolingTargetSource子类化以支持任何其他池化API。

下面是示例配置：
```java
<bean id="businessObjectTarget" class="com.mycompany.MyBusinessObject"
        scope="prototype">
    ... properties omitted
</bean>

<bean id="poolTargetSource" class="org.springframework.aop.target.CommonsPool2TargetSource">
    <property name="targetBeanName" value="businessObjectTarget"/>
    <property name="maxSize" value="25"/>
</bean>

<bean id="businessObject" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="targetSource" ref="poolTargetSource"/>
    <property name="interceptorNames" value="myInterceptor"/>
</bean>
```
注意目标对象--例子里的“businessObjectTarget”--必须是个原型。这允许PoolingTargetSource的实现在必要时为目标创建新的实例来增大池的容量。查看AbstractPoolingTargetSource和你想要使用的具体子类的Javadoc获取更多关于它属性的信息：maxSize是最基础的，而且永远都要求被提供。 

在这个例子里，“myInterceptor”是一个拦截器的名字，这个拦截器需要在同一个IoC上下文中被定义。然而，定义对拦截器进行池化是不必要的。如果你想要的只是池化而没有其它通知，就不要设置interceptorNames属性。 

可以配置Spring来把任何被池化对象转型到org.springframework.aop.target.PoolingConfig接口，这通过一个introduction暴露配置以及当前池的大小。你需要像这样定义一个advisor： 
```java
<bean id="poolConfigAdvisor" class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
    <property name="targetObject" ref="poolTargetSource"/>
    <property name="targetMethod" value="getPoolingConfigMixin"/>
</bean>
```
这个advisor可以通过调用AbstractPoolingTargetSource类上的一个方便的方法来获得，因此这里使用MethodInvokingFactoryBean。这个advisor名（这里是“poolConfigAdvisor”）必须在提供被池化对象的ProxyFactoryBean里的拦截器名列表里中。 

转型看起来像这样：
```java
PoolingConfig conf = (PoolingConfig) beanFactory.getBean("businessObject");
System.out.println("Max pool size is " + conf.getMaxSize());
```
**注意：**池化无状态服务对象通常是不必要的。我们不认为这（池化）应当是缺省的选择，因为多数无状态对象是先天线程安全的，如果资源被缓存，那么对实例进行池化会引起很多问题。

使用自动代理时池化更加简单。可以为任何自动代理创建器设置所使用的TargetSource

**原型目标源**

建立一个“原型”目标源和池化TargetSource很相似。在这个例子里，当每次方法调用时，将创建一个目标的新实例。虽然在新版本的JVM中创建一个新对象的代价并不高，但是把新对象织入（满足它的IoC依赖）可能是很昂贵的。因此如果没有很好的理由，你不应该使用这个方法。

为了做到这点，你可以把上面的poolTargetSource定义修改成下面的形式。（为了清楚说明，修改了bean的名字。） 
```java
<bean id="prototypeTargetSource" class="org.springframework.aop.target.PrototypeTargetSource">
  <property name="targetBeanName" ref="businessObjectTarget"/>
</bean>

```
这里只有一个属性：目标bean的名字。TargetSource的实现使用继承来确保命名的一致性。就像池化目标源那样，目标bean必须是一个原型的bean定义。

**ThreadLocal目标源**

如果你需要为每个进来的请求（即每个线程）创建一个对象，ThreadLocal目标源是很有用的。 ThreadLocal的概念提供了一个JDK范围的功能，这可以为一个线程透明的保存资源。建立一个 ThreadLocalTargetSource的过程和其它目标源几乎完全一样：
```java
<bean id="threadlocalTargetSource" class="org.springframework.aop.target.ThreadLocalTargetSource">
    <property name="targetBeanName" value="businessObjectTarget"/>
</bean>
```
**注意：**如果不正确的在一个多线程和多类加载器的环境里使用ThreadLocal，将带来严重的问题（可能潜在地导致内存泄漏）。永远记住应该把一个threadlocal包装在其它的类里，并永远不要直接使用ThreadLocal本身（当然是除了threadlocal包装类之外）。 同时，永远记住正确的设置（set）和取消（unset）（后者仅仅需要调用ThreadLocal.set(null)）绑定到线程的本地资源。取消在任何情况下都应该进行，否则也许会导致错误的行为。Spring的ThreadLocal支持将为你处理这个问题，所以如果没有其它正确的处理代码，永远应该考虑使用这个功能。

**定义新的通知类型**

Spring AOP被设计为可扩展的。通过在内部使用拦截实现策略，你可以支持已有的环绕通知，前置通知，异常通知和后置通知之外的任意通知类型，它是开箱即用的。

org.springframework.aop.framework.adapter包是一个SPI包，它允许添加新的自定义通知类型而无需修改核心框架。对于自定义Advice类型的唯一的限制是它必须实现org.aopalliance.aop.Advice这个标记接口。 

请参考org.springframework.aop.framework.adapter包的Javadoc来了解进一步的信息。













