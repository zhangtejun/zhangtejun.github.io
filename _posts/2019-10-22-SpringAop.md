---
layout: post
title:  "springAop"
date:   2019-10-22 10:34:23
author: zhangtejun
categories: Java
---
#####
AOP（目标+切面+配置）：
* 目标：可以视为待增强的`目标对象`
* 切面：通常包含对`目标`的增强逻辑
* 配置：可以看成一种编织，通过AOP体系提供的配置环境，可以把`目标`和`切面`结合起来，从而完成`切面`对`目标对象`的编织实现。

在SpringAOP的实现中，使用java语言来增强目标对象和切面增强，并为这2者结合提供环境配置。

对于编织配置，毫无疑问，可以使用IOC容器来完成，对于POJO的配置，本来就是IOC的强项。对于其他AOP的实现方案来说，可能需要
特定的实现语言和环境，或特定的编译环境。例如AspectJ中，尽管其增强的目标也算java对象，却需要特定的Aspect语言和编译环境来支持。

##### Advice通知
Advice定义在连接点做什么，为切面增强提供织入接口。在Spring AOP中，它主要描述Spring AOP围绕方法调用而注入的切面行为。

Advice是AOP联盟定义的一个接口，Spring对其做了扩展，提供了更加具体通知类型，比如AfterAdvice，BeforeAdvice，ThrowsAdvice等

**BeforeAdvice**
```
public interface MethodBeforeAdvice extends BeforeAdvice {
  // 会在调用目标方法时被回调，Method：目标方法的反射对象，args：目标方法的输入参数，
	void before(Method method, Object[] args, @Nullable Object target) throws Throwable;
}

public class CountingBeforeAdvice extends MethodCounter implements MethodBeforeAdvice {
    @Override
    public void before(Method method, Object[] objects, Object o) throws Throwable {
        log.info("invoke CountingBeforeAdvice .... by methodName：{}",method.getName());
        count(method.getName());
    }
}

public class MethodCounter {
    private ConcurrentHashMap<String,Long> map = new ConcurrentHashMap();
    private long allCount = 0;// 所有方法的调用次数（包括不同方法名）

    // CountingBeforeAdvice 入口
    protected void count(String methodName){
        Long  count = map.getOrDefault(methodName,0L);
        map.put(methodName,count.longValue()+1);
        ++allCount;
    }
    // 根据方法取得调用次数
    public Long getCountByMethodName(String methodName){
        return map.getOrDefault(methodName,0L);
    }

    public  long getAllCounts(){
        return this.allCount;
    }
 }
```

**AfterAdvice**
同BeforeAdvice,不同点是在目标方法执行成功后调用。

**ThrowsAdvice**
该通知并没有指定需要实现的接口方法，它在抛出异常时被回调，这个回调是AOP使用反射机制来完成的。


##### Pointcut切点
Pointcut决定Advice通知应该作用于哪个连接点，也就是说通过Pointcut来定义需要增强的方法的集合,这些集合的选择可以按照一定的规则来完成。
在这种情况下，Pointcut通常意味著标志方法，例如：这些需要增强的地方可以由某个正则表达式进行标识或根据某个方法名进行匹配。

```
public interface Pointcut {
	ClassFilter getClassFilter();
	MethodMatcher getMethodMatcher();
	Pointcut TRUE = TruePointcut.INSTANCE;
}
```

从Pointcut接口定义中可以看出，需要返回一个MethodMatcher。对于point的匹配功能，具体由这个MethodMatcher来完成。
换句话说，就说由MethodMatcher来判断是否需要对当前方法进行增强，或者是否需要对当前方法应用已配置好的Advice通知。

##### Advisor通知器
完成了目标对象方法的切面增强设计（Advice）和关注点的设计（Pointcut）后，需要y一个对象它们结合起来，完成这个作用的就是Advisor。
通过Advisor可以定义应该使用哪个通知并且在哪个关注点使用它。

**DefaultPointcutAdvisor**
```
public class DefaultPointcutAdvisor extends AbstractGenericPointcutAdvisor implements Serializable {
  // 单例模式（public static final TruePointcut INSTANCE = new TruePointcut()） 使用static关键字来持有实例，
  // 私有化构造器 保证不会被再次创建和实例化
	private Pointcut pointcut = Pointcut.TRUE;
	public DefaultPointcutAdvisor() {
	}
	public DefaultPointcutAdvisor(Advice advice) {
		this(Pointcut.TRUE, advice);
	}
	public DefaultPointcutAdvisor(Pointcut pointcut, Advice advice) {
		this.pointcut = pointcut;
		setAdvice(advice);
	}

	public void setPointcut(@Nullable Pointcut pointcut) {
		this.pointcut = (pointcut != null ? pointcut : Pointcut.TRUE);
	}
	@Override
	public Pointcut getPointcut() {
		return this.pointcut;
	}
	@Override
	public String toString() {
		return getClass().getName() + ": pointcut [" + getPointcut() + "]; advice [" + getAdvice() + "]";
	}

}

//2.
final class TruePointcut implements Pointcut, Serializable {

	public static final TruePointcut INSTANCE = new TruePointcut();
	private TruePointcut() {
	}
	@Override
	public ClassFilter getClassFilter() {
		return ClassFilter.TRUE;
	}
	@Override
	public MethodMatcher getMethodMatcher() {
	  // 使用TrueMethodMatcher作为方法匹配器（ public static final TrueMethodMatcher INSTANCE = new TrueMethodMatcher()）
		return MethodMatcher.TRUE;
	}

	private Object readResolve() {
		return INSTANCE;
	}

	@Override
	public String toString() {
		return "Pointcut.TRUE";
	}

}
```
DefaultPointcutAdvisor有属性Advice和Pointcut，用于分别配置Advice和Pointcut。TruePointcut的实现中使用TrueMethodMatcher作为方法匹配器，
这个方法匹配器对任何方法匹配都要求返回`true`的结果

##### Spring Aop 设计和实现
**JVM动态代理特性**

在Spring AOP的模块中，一个主要的部分是代理对象的生成，而对于Spring应用来说，是通过配置和调用Spring的ProxyFactoryBean来完成这个任务的。

在ProxyFactoryBean中，封装了主要代理对象的生成过程（包含JDK动态代理和CGLIB两种方式）。

AspectJProxyFactory，ProxyFactory，ProxyFactoryBean它们都在同一个类的继承下，都是继承至ProxyConfig,AdvisedSupport和ProxyCreatorSupport的子类。

ProxyConfig可以看作是一个数据基类，这个数据基类为ProxyFactoryBean这样的子类提供了配置属性，在另一个基类AdvisedSupport的实现中封装了AOP对通知和通知器的相关操作。
这些操作对于不同的AOP生成的代理对象都是一样的，但是具体AOP代理对象的创建，AdvisedSupport将其交由其子类去实现。
对于ProxyCreatorSupport可以将其看作是子类创建AOP代理对象的一个辅助类。

具体的AOP代理生成,根据不同的需要，分别由ProxyFactory，ProxyFactoryBean，AspectJProxyFactory来完成，对于需要使用AspectJ的AOP应用，AspectJProxyFactory
起到集成Spring和AspectJ的作用。对于使用Spring AOP的应用ProxyFactory，ProxyFactoryBean都提供了AOP功能的封装，区别是：使用ProxyFactoryBean，可以
在IoC容器中完成声明式配置，而使用ProxyFactory需要编程式的使用AOP功能。

#### 使用AOP的几种方式
**配置ProxyFactoryBean**
在基于XML配置Spring的Bean时，往往需要一系列配置步骤来使用ProxyFactoryBean和AOP。
1. 定义需要使用的通知器Advisor,该通知器作为一个bean来定义，通知器实现定义了目标对象需要增强的行为，即Advice通知。
2. 定义ProxyFactoryBean，它封装了AOP功能的主要类，在配置时，需要设置proxyInterface，interceptorNames（需要定义的通知器）和Target等。
3. 定义Target属性，作为target属性注入的bean,是需要用AOP通知器中的切面应用来增强的对象。

1. 定义一个接口
```java
public interface TestAopService {
    void method(Object obj); //AOP需要拦截代理的方法
    void methodNew(Object obj);
}
```
2. 定义接口的业务实现类
```
public class TestAopServiceImpl  implements TestAopService{
    @Override
    public void method(Object obj) {
        System.out.println("invoke method....");
    }

    @Override
    public void methodNew(Object obj) {
        System.out.println("invoke methodNew....");
    }
}
```
3. 业务实现类外需要的逻辑
```
@Slf4j
public class TestAopServiceAdvice implements MethodBeforeAdvice,AfterReturningAdvice {
    @Override
    public void afterReturning(Object o, Method method, Object[] objects, Object o1) throws Throwable {
        log.info("afterReturning .... {}",method.getName());
    }
    @Override
    public void before(Method method, Object[] objects, Object o) throws Throwable {
        log.info("before .... {}",method.getName());
    }
}
```

4. Spring核心配置文件
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 定义被代理目标 -->
    <bean id="testAopServiceImpl" class="com.example.demo.aop.test.aop.TestAopServiceImpl"></bean>
    <!-- 定义通知内容，也就是切入点执行前后需要做的事情 -->
    <bean id="testAopServiceAdvice" class="com.example.demo.aop.test.aop.TestAopServiceAdvice"></bean>
    <!-- 定义切入点位置，这里定义到了method方法上 -->
    <bean id="pointcut" class="org.springframework.aop.support.JdkRegexpMethodPointcut">
        <property name="pattern" value=".*method"></property>
    </bean>
    <!-- 通知器：使切入点与通知相关联，完成切面配置 -->
    <bean id="testAopServiceAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
        <property name="advice" ref="testAopServiceAdvice"></property>
        <property name="pointcut" ref="pointcut"></property>
    </bean>
    <!-- 设置代理 -->
    <bean id="proxy" class="org.springframework.aop.framework.ProxyFactoryBean">
        <!-- 代理的对象，增加非业务逻辑的执行 -->
        <property name="target" ref="testAopServiceImpl"></property>
        <!-- 使用切面 -->
        <property name="interceptorNames" value="testAopServiceAdvisor"></property>
        <!-- 代理接口，hw接口 -->
        <property name="proxyInterfaces" value="com.example.demo.aop.test.aop.TestAopService"></property>
    </bean>
</beans>
```

在ProxyFactoryBean中，通过interceptorNames属性来配置已经定义好的通知器Advisor。名字虽然是interceptorNames，实际上是AOP配置通知器的地方。
还需要为target目标对象生成Proxy代理对象，从而为AOP的横切面做好编织准备工作。

在ProxyFactoryBean中的AOP实现依赖JDK或者CGLIB提供的Proxy特性。从factoryBean中获取对象，以getObject()方法作为入口完成的。

对于ProxyFactoryBean来说，把需要对target目标对象增加的增强处理，都通过getObject方法进行了封装。
```
public Object getObject() throws BeansException {
    // 1. 对拦截器链进行初始化
		initializeAdvisorChain();
		// 2. 生成不同类型的bean
		if (isSingleton()) {
			return getSingletonInstance();
		}
		else {
			if (this.targetName == null) {
				logger.info("Using non-singleton proxies with singleton targets is often undesirable. " +
						"Enable prototype proxies by setting the 'targetName' property.");
			}
			return newPrototypeInstance();
		}
	}
  // 为Proxy代理对象配置Advisor链
	private synchronized void initializeAdvisorChain() throws AopConfigException, BeansException {
  		if (this.advisorChainInitialized) {// 是否已初始化（第一次初始化发生在第一次通过ProxyFactoryBean获取代理对象的时候）
  			return;
  		}

  		if (!ObjectUtils.isEmpty(this.interceptorNames)) {
  			if (this.beanFactory == null) {
  				throw new IllegalStateException("No BeanFactory available anymore (probably due to serialization) " +
  						"- cannot resolve interceptor names " + Arrays.asList(this.interceptorNames));
  			}

  			// Globals can't be last unless we specified a targetSource using the property...
  			if (this.interceptorNames[this.interceptorNames.length - 1].endsWith(GLOBAL_SUFFIX) &&
  					this.targetName == null && this.targetSource == EMPTY_TARGET_SOURCE) {
  				throw new AopConfigException("Target required after globals");
  			}

  			// Materialize interceptor chain from bean names.
  			for (String name : this.interceptorNames) {
  				if (logger.isTraceEnabled()) {
  					logger.trace("Configuring advisor or advice '" + name + "'");
  				}

  				if (name.endsWith(GLOBAL_SUFFIX)) {
  					if (!(this.beanFactory instanceof ListableBeanFactory)) {
  						throw new AopConfigException(
  								"Can only use global advisors or interceptors with a ListableBeanFactory");
  					}
  					addGlobalAdvisor((ListableBeanFactory) this.beanFactory,
  							name.substring(0, name.length() - GLOBAL_SUFFIX.length()));
  				}

  				else {
  					// If we get here, we need to add a named interceptor.
  					// We must check if it's a singleton or prototype.
  					Object advice;
  					if (this.singleton || this.beanFactory.isSingleton(name)) {
  						// Add the real Advisor/Advice to the chain.
  						// 读取配置中出现的所有通知器
  						advice = this.beanFactory.getBean(name);
  					}
  					else {
  						// It's a prototype Advice or Advisor: replace with a prototype.
  						// Avoid unnecessary creation of prototype bean just for advisor chain initialization.
  						advice = new PrototypePlaceholderAdvisor(name);
  					}
  					// 通知器加入拦截器链
  					addAdvisorOnChainCreation(advice, name);
  				}
  			}
  		}
  		this.advisorChainInitialized = true;
  	}
```


##### Spring AOP拦截器调用的实现
使用JDK动态代理来生成代理对象，那么需要通过InvocationHandler来设置拦截器回调；而如果使用CGLIB来生成代理对象，
就需要通过DynamicAdvisedInterceptor来完成回调。

AopProxy代理对象的生成：`Proxy.newProxyInstance(classLoader,proxiedInterfaces,this)`,this代表InvocationHandler对象，
器方法invoke作为拦截回调的入口，在JdkDynamicAopProxy实现类该接口，即当Proxy对象的代理方法被调用时，该invoke方法作为Proxy对象的回调函数被触发，
通过invoke来完成对目标方法的增强或者说是拦截。

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		MethodInvocation invocation;
		Object oldProxy = null;
		boolean setProxyContext = false;

		TargetSource targetSource = this.advised.targetSource;
		Object target = null;

		try {
			if (!this.equalsDefined && AopUtils.isEqualsMethod(method)) {
				// 目标对象没有实现Object方法
				return equals(args[0]);
			}
			else if (!this.hashCodeDefined && AopUtils.isHashCodeMethod(method)) {
				// 目标对象没有实现Object的基本方法：hashCode
				return hashCode();
			}
			else if (method.getDeclaringClass() == DecoratingProxy.class) {
        // 根据代理对象的配置来调用服务
				return AopProxyUtils.ultimateTargetClass(this.advised);
			}
			else if (!this.advised.opaque && method.getDeclaringClass().isInterface() &&
					method.getDeclaringClass().isAssignableFrom(Advised.class)) {
				// Service invocations on ProxyConfig with the proxy config...
				return AopUtils.invokeJoinpointUsingReflection(this.advised, method, args);
			}

			Object retVal;

			if (this.advised.exposeProxy) {
				// Make invocation available if necessary.
				oldProxy = AopContext.setCurrentProxy(proxy);
				setProxyContext = true;
			}

			// 得到目标对象
			target = targetSource.getTarget();
			Class<?> targetClass = (target != null ? target.getClass() : null);

			List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);

			if (chain.isEmpty()) {
				// 如果没有拦截器 直接调用target对应的方法
				Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
				retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
			}
			else {
				// 如果有拦截器，需要先执行拦截器，然后在执行目标对象的相应方法
				invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
				// Proceed to the joinpoint through the interceptor chain.
				retVal = invocation.proceed();
			}
			Class<?> returnType = method.getReturnType();
			if (retVal != null && retVal == target &&
					returnType != Object.class && returnType.isInstance(proxy) &&
				retVal = proxy;
			}
			else if (retVal == null && returnType != Void.TYPE && returnType.isPrimitive()) {
				throw new AopInvocationException(
						"Null return value from advice does not match primitive return type for: " + method);
			}
			return retVal;
		}
		finally {
			if (target != null && !targetSource.isStatic()) {
				// Must have come from TargetSource.
				targetSource.releaseTarget(target);
			}
			if (setProxyContext) {
				// Restore old proxy.
				AopContext.setCurrentProxy(oldProxy);
			}
		}
	}
```

**Cglib2AopProxy的intercept拦截**
Cglib2AopProxy的intercept回调方法和JdkDynamicAopProxy的回调实现是类似的，Cglib2AopProxy在构造CglibMethodInvocation对象来
完成拦截器的调用，而JdkDynamicAopProxy是通过构造ReflectiveMethodInvocation对象来完成的。
```
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
				// 从advised中获取已配置好的通知
				List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
				Object retVal;
				if (chain.isEmpty() && Modifier.isPublic(method.getModifiers())) {
					// 如果没有AOP通知设置， 直接调用target对象的对应方法
					Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
					retVal = methodProxy.invoke(target, argsToUse);
				}
				else {
					// 通过CglibMethodInvocation来启动Advice通知
					retVal = new CglibMethodInvocation(proxy, target, method, args, targetClass, chain, methodProxy).proceed();
				}
				retVal = processReturnType(proxy, target, method, retVal);
				return retVal;
			}
			finally {
				if (target != null && !targetSource.isStatic()) {
					targetSource.releaseTarget(target);
				}
				if (setProxyContext) {
					// Restore old proxy.
					AopContext.setCurrentProxy(oldProxy);
				}
			}
		}
```

**目标对象方法的调用**
如果没有设置拦截器，目标方法会被直接调用，对JdkDynamicAopProxy代理对象来说，对目标方法的调用是通过AopUtils使用反射机制在
AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse)的方法中实现的，先得到调用方法的反射对象，然后使用invoke
启动对方法反射对象调用。
```
public static Object invokeJoinpointUsingReflection(@Nullable Object target, Method method, Object[] args)
			throws Throwable {

		// Use reflection to invoke the method.
		try {
			ReflectionUtils.makeAccessible(method);
			return method.invoke(target, args);
		}
		catch (InvocationTargetException ex) {
			// Invoked method threw a checked exception.
			// We must rethrow it. The client won't see the interceptor.
			throw ex.getTargetException();
		}
		catch (IllegalArgumentException ex) {
			throw new AopInvocationException("AOP configuration seems to be invalid: tried calling method [" +
					method + "] on target [" + target + "]", ex);
		}
		catch (IllegalAccessException ex) {
			throw new AopInvocationException("Could not access method [" + method + "]", ex);
		}
	}
```

对于CglibAopProxy的代理对象来说，它对目标的调用是通过CGLIB的MethodProxy对象直接完成的。即`methodProxy.invoke(target, args)`

**AOP拦截器链的调用**
Jdk动态代理和CGLIB会生成不同的AopProxy代理对象,从而构造不同的回调方法来启动对拦截器的调用，JdkDynamicAopProxy的invoke和CglibAopProxy的intercept
它们对拦截器的调用都是在ReflectMethodIntercept中的process方法阿里实现的。

在process方法中，会逐个运行拦截器的拦截方法，在运行拦截方法前需要对代理方法完成一个匹配，通过这个匹配来判断拦截器是否满足切面增强的要求。
Pointcut切点中需要进行matches的匹配过程，即matches调用对方法进行匹配判断，来决定是否s实行通知增强。

```
public Object proceed() throws Throwable {
		//	We start with an index of -1 and increment early.
		// 从索引为 -1 的拦截器开始调用
		if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
			return invokeJoinpoint();
		}

		Object interceptorOrInterceptionAdvice = this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
		if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
			// Evaluate dynamic method matcher here: static part will already have
			// been evaluated and found to match.
			InterceptorAndDynamicMethodMatcher dm =
					(InterceptorAndDynamicMethodMatcher) interceptorOrInterceptionAdvice;
			Class<?> targetClass = (this.targetClass != null ? this.targetClass : this.method.getDeclaringClass());
			// 对拦截器进行动态匹配判断
			if (dm.methodMatcher.matches(this.method, targetClass, this.arguments)) {
				return dm.interceptor.invoke(this);
			}
			else {
				// Dynamic matching failed.
				// Skip this interceptor and invoke the next in the chain.
				return proceed();
			}
		}
		else {
			// It's an interceptor, so we just invoke it: The pointcut will have
			// been evaluated statically before this object was constructed.
			return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
		}
	}
```

**配置通知器**
```
Object interceptorOrInterceptionAdvice = this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
```
interceptorOrInterceptionAdvice是获得的拦截器，具体来说它是interceptorsAndDynamicMethodMatchers持有list的一个元素，
如何配置拦截器？即list中的拦截器哪里来？
```
List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
```
获取interceptor主要是由advised对象完成的，它是一个AdvisedSupport对象,
```
public List<Object> getInterceptorsAndDynamicInterceptionAdvice(Method method, @Nullable Class<?> targetClass) {
		MethodCacheKey cacheKey = new MethodCacheKey(method);
		List<Object> cached = this.methodCache.get(cacheKey);
		if (cached == null) {
			cached = this.advisorChainFactory.getInterceptorsAndDynamicInterceptionAdvice(
					this, method, targetClass);
			this.methodCache.put(cacheKey, cached);
		}
		return cached;
	}
```
获取拦截的工作是由配置好的advisorChainFactory来完成的，
