---
layout: post
title:  "spring aop"
date:   2017-10-21 08:13:04
author: zhangtejun
categories: zhangtejun
---
##### Introduction
面向切面编程（AOP）提供另外一种思考程序结构的方式，通过这种方式弥补了面向对象编程（OOP）的不足。 
OOP模块化的关键单位是类（class），而在AOP中，模块化单位是方面。方面可以实现诸如跨多个类型和对象之间的事
务管理等问题的模块化。 （这种关注在AOP文献中通常被称为横切关注点。）

Spring的一个关键的组件就是 AOP框架(大多数AOP框架以拦截器做通知模型)。 尽管如此，Spring IoC容器并不依赖于AOP，这意味着你可以自由选择是否使用AOP，AOP提供强大的中间件解决方案，这使得Spring IoC容器更加完善。

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





