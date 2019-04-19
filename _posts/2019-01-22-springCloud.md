---
layout: post
title:  "springboot"
date:   2019-01-22 11:35:34
author: zhangtejun
categories: SpringCloud
---
# springCloud
#### 服务核心操作
* 服务注册（register）
* 服务下线（cancel）
* 服务租约（renew）
* 服务剔除（evict）

LeaseManager是Eureka关于服务发现定义的写操作接口，LookupService是定义查询相关的操作方法，主要用于client。
```java
public interface LeaseManager<T> {
    void register(T var1, int var2, boolean var3);// 注册实例服务信息
    boolean cancel(String var1, String var2, boolean var3);//删除实例
    boolean renew(String var1, String var2, boolean var3);//和eureka server 进行心跳等操作，维持租约
    void evict();// 剔除租约过期的服务实例
}

public interface LookupService<T> {
    Application getApplication(String var1);
    Applications getApplications();
    List<InstanceInfo> getInstancesById(String var1);
    InstanceInfo getNextServerFromEureka(String var1, boolean var2);
}
```
#### 服务注册
* 服务实例如何注册到服务中心  
* 服务如何从注册中心剔除
* 服务实例的一致性如何保持

#### 访问方式
1. 各微服务以http接口的形式对外提供服务。
2. 消费者调用时，底层通过http client的方式访问。
3. JDK原生的UrlConnection,Apache的http client，Netty的异步http client，spring的RestTemplate，springCloud Open Feign 进行服务间调用。

>Feign是什么？
>>Feign 是一个声明式的 Web Service 客户端。它的出现使开发 Web Service 客户端变得很简单。使用 Feign 只需要创建一个接口加上对应的注解，
>>比如：@FeignClient 注解。 Feign 有可插拔的注解，包括 Feign 注解和 JAX-RS 注解。Feign 也支持编码器和解码器，
>>Spring Cloud Open Feign 对 Feign 进行增强支持 Spring MVC 注解，可以像 Spring Web 一样使用 HttpMessageConverters 等。
>>
> Feign 的特性
>>1，可插拔的注解支持，包括 Feign 注解和AX-RS注解。  
>>2，支持可插拔的 HTTP 编码器和解码器。  
>>3，支持 Hystrix 和它的 Fallback。  
>>4，支持 Ribbon 的负载均衡。  
>>5，支持 HTTP 请求和响应的压缩。Feign 是一个声明式的 WebService 客户端，它的目的就是让 Web Service 调用更加简单。它整合了 Ribbon 和 Hystrix，从而不需要开发者针对 Feign 对其进行整合。Feign 还提供了 HTTP 请求的模板，通过编写简单的接口和注解，就可以定义好 HTTP 请求的参数、格式、地址等信息。Feign 会完全代理 HTTP 的请求，在使用过程中我们只需要依赖注入 Bean，然后调用对应的方法传递参数即可。  
>>
>使用
>>1. 引入Feign依赖
>>2. 启动类加入注解@EnableFeignClients 开启 Feign 扫描支持
>>3. Feign 接口编写 @FeignClient(value = "service-provider")
>>4. Controller注入@Resource/@Autowired等
>>
>@FeignClient 注解
>>name：指定 Feign Client 的名称，如果项目使用了 Ribbon，name 属性会作为微服务的名称，用于服务发现。
>>url：url 一般用于调试，手动指定 @FeignClient 调用的地址。
>>decode404：当发生404错误时，如果该字段为 true，会调用 decoder 进行解码，否则抛出 FeignException。
>>configuration：Feign 配置类，可以自定义 Feign 的 Encoder、Decoder、LogLevel、Contract。
>>fallback：定义容错的处理类，当调用远程接口失败或超时时，会调用对应接口的容错逻辑，fallback 指定的类必须实现 @FeignClient 标记的接口。
>>fallbackFactory：工厂类，用于生成 fallback 类示例，通过这个属性我们可以实现每个接口通用的容错逻辑，减少重复的代码。
>>path：定义当前 FeignClient 的统一前缀。
>>
>>Feign 工作原理
>>1.主程序入口添加 @EnableFeignClients 注解开启对 Feign Client 扫描加载处理。根据 Feign Client 的开发规范，定义接口并加 @FeignClients 注解
>>当程序启动时，会进行包扫描，扫描所有 @FeignClients 的注解的类，并将这些信息注入 Spring IOC 容器中。当定义的 Feign 接口中的方法被调用时，
>>通过JDK的代理的方式，来生成具体的 RequestTemplate。当生成代理时，Feign 会为每个接口方法创建一个 RequetTemplate 对象，该对象封装了 HTTP 请求需要的全部信息，如请求参数名、请求方法等信息都是在这个过程中确定的。
>>然后由 RequestTemplate 生成 Request，然后把 Request 交给 Client 去处理，这里指的 Client 可以是 JDK 原生的 URLConnection、Apache 的 Http Client 也可以是 Okhttp。
>>最后 Client 被封装到 LoadBalanceclient 类，这个类结合 Ribbon 负载均衡发起服务之间的调用。
>>
>>Feign开启GZIP,因为传输的数据格式都已经转换为了二进制，如果不做其他更改的话，返回将会是乱码,需要把返回值修改为 ResponseEntity<byte[]>。
```yaml
feign:  
    compression:  
      request:  
        enabled: true # 开启请求压缩  
        mime-types: text/xml,application/xml,application/json # 配置压缩的MIME TYPE  
        min-request-size: 2048 # 配置压缩数据大小的下限  
      response:  
        enabled: true # 开启响应压缩 
```
>
>Feign Client
>>Feign在默认情况下使用jdk原生的URLConnection发送http请求，没有连接池，对每个地址保存一个长连接。
>>调用优化：使用Apache的client替换默认的HttpClient,来支持连接池，超时时间等。
>>1. HTTP client替换Feign默认Client,第一步增加http依赖，然后增加yml配置
```
<!-- 使用Apache HttpClient替换Feign原生httpclient -->
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
</dependency>
<dependency>
    <groupId>com.netflix.feign</groupId>
    <artifactId>feign-httpclient</artifactId>
    <version>8.17.0</version>
</dependency>

feign:
  httpclient:
    enabled: true

```
>>2. okhttp替换默认client,first增加依赖，然后yml配置okhttp,最后创建OkHttpClient对象
```
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-okhttp</artifactId>
</dependency>
feign:
  okhttp:
    enabled: true 
```
>
>>Feign返回图片处理流，一般图片返回为字节数组，在Feign中把流转为字节数组进行传递，因Controller层不能返回byte
>>需将Feign的返回改为Response。
>
>Feign 调用传递Token
>>在调用Feign时，向请求头中添加Token，可以通过实现Feign提供的一个RequestInterceptor接口来完成。

#### Ribbon
>Ribbon是一个客户端负载均衡器，它赋予用于一些支配http和tcp行为的能力。feign和zuul默认已经集成ribbon。
>>1. 引入依赖`spring-cloud-starter-netflix-ribbon`
>>2. 注入RestTemplate
>>3. 添加注解@LoadBalance
>>4. 启动java -jar demo.jar --server.port=9000
>
>Ribbon负载均衡策略
>>1. 全局修改默认策略，需要增加一个配置类。
```java
@org.springframework.context.annotation.Configuration
public class Configuration {
    @Bean
    public IRule ribbonRule()
    {
        return new RandomRule();
    }
}
```
>>1. 基于注解的策略配置，可以针对某一个源服务进行特有策略设置。需要对全局设置的代码进行修改
```java
@Configuration
@AviodScan
public class ProviderConfiguration {
   @Autowired
   IClientConfig config;
   @Bean
    public IRule ribbonRule(IClientConfig config)
    {
        return new RandomRule();
    }
}
```
>>@AviodScan 自定义注解，用于注解扫描排除java文件。
>>IClientConfig是针对客户端的配置管理器，使用@RibbonClient或者@RibbonClients在启动类上配置对一个或者多个源服务进行制定复制均衡策略。
```java
@RibbonClient(name="service-provider",configuration = ProviderConfiguration.class)
@ComponentScan(excludeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION,value = {AviodScan.class})})
public class ServiceInvokerApplication {}


// 或者指定多个
@RibbonClients(value = {
        @RibbonClient(name="service-provider",configuration = ProviderConfiguration.class),
        @RibbonClient(name="service-provider-B",configuration = ProviderConfiguration.class)
})
@ComponentScan(excludeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION,value = {AviodScan.class})})
public class ServiceInvokerApplication {}
```
>>3. 基于配置文件的策略
>>语法格式为<client-name>.ribbon.*
```yaml
service-provider:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RoundRobinRule
```
>ribbon饥饿加载
>>ribbon在进行客户端负载均衡时，并不是启动时加载上下文，而是在实际请求的时候才会去创建。可以通过在配置文件中指定客户端的名称来，指定Ribbon开启饥饿加载。
```yaml
ribbon:
  eager-load:
    enabled: true
    clients: service-provider,service-provider-b
```
>Ribbon脱离Eureka使用
>>默认情况下ribbon客户端从Eureka注册中心读取服务列表，来实现动态负载均衡。在一种情况下，不推荐默认方式。多人共用的Eureka公共注册中心，容易产生服务入侵。
>>Ribbon脱离Eureka使用,需要在配置文件中禁用Eureka.
```yaml
# 禁用
ribbon:
  eureka:
    enabled: false
    
# 然后设置源服务地址列表
client:
  eureka:
    listOfServers: http://localhost:7000/,http://localhost:7001/
```
>Ribbon原理
>>使用ribbon来进行负载均衡，基本使用方式都需要注入一个RestTemplate的bean,并使用@LoadBalanced使其具备负载均衡能力。
```java
/**
 * Annotation to mark a RestTemplate bean to be configured to use a LoadBalancerClient
 * @author Spencer Gibb
 */
@Target({ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Qualifier
public @interface LoadBalanced {
}
```
>>1. 标记一个RestTemplate来使用LoadBalancerClient。
>>2. LoadBalancerClient 扩展至 ServiceInstanceChooser
```java
public interface LoadBalancerClient extends ServiceInstanceChooser {

	/**
	 * execute request using a ServiceInstance from the LoadBalancer for the specified service
	 * @param serviceId the service id to look up the LoadBalancer
	 * @param request allows implementations to execute pre and post actions such as
	 * incrementing metrics
	 * @return the result of the LoadBalancerRequest callback on the selected ServiceInstance
	 * 
	 * 使用来自LoadBalancer的ServiceInstance为指定服务执行请求
	 */
	<T> T execute(String serviceId, LoadBalancerRequest<T> request) throws IOException;

	/**
	 * execute request using a ServiceInstance from the LoadBalancer for the specified service
	 * @param serviceId the service id to look up the LoadBalancer
	 * @param serviceInstance the service to execute the request to
	 * @param request allows implementations to execute pre and post actions such as
	 * incrementing metrics
	 * @return the result of the LoadBalancerRequest callback on the selected ServiceInstance
	 * 
	 * 上一个方法的重载，细节的实现
	 */
	<T> T execute(String serviceId, ServiceInstance serviceInstance, LoadBalancerRequest<T> request) throws IOException;

	/**
	 * Create a proper URI with a real host and port for systems to utilize.
	 * Some systems use a URI with the logical serivce name as the host,
	 * such as http://myservice/path/to/service.  This will replace the
	 * service name with the host:port from the ServiceInstance.
	 * @param instance
	 * @param original a URI with the host as a logical service name
	 * @return a reconstructed URI
	 * 使用主机的ip和端口构建特定的URI供Ribbon内部使用
	 */
	URI reconstructURI(ServiceInstance instance, URI original);
}

public interface ServiceInstanceChooser {

    /**
     * Choose a ServiceInstance from the LoadBalancer for the specified service
     * @param serviceId the service id to look up the LoadBalancer
     * @return a ServiceInstance that matches the serviceId
     */
    ServiceInstance choose(String serviceId);// 根据serviceId，结合负载均衡器选择一个服务实例
}

```
>>Ribbon核心配置类LoadBalancerAutoConfiguration
>>@ConditionalOnBean（仅仅在当前上下文中存在某个对象时，才会实例化一个Bean）
>>@ConditionalOnClass（某个class位于类路径上，才会实例化一个Bean）
>>@ConditionalOnExpression（当表达式为true的时候，才会实例化一个Bean）
>>@ConditionalOnMissingBean（仅仅在当前上下文中不存在某个对象时，才会实例化一个Bean）
>>@ConditionalOnMissingClass（某个class类路径上不存在的时候，才会实例化一个Bean）
>>@ConditionalOnNotWebApplication（不是web应用）
```java
@Configuration
@ConditionalOnClass(RestTemplate.class)
@ConditionalOnBean(LoadBalancerClient.class)
@EnableConfigurationProperties(LoadBalancerRetryProperties.class)
public class LoadBalancerAutoConfiguration {

	@LoadBalanced
	@Autowired(required = false)
	private List<RestTemplate> restTemplates = Collections.emptyList();

	@Bean
	public SmartInitializingSingleton loadBalancedRestTemplateInitializerDeprecated(
			final ObjectProvider<List<RestTemplateCustomizer>> restTemplateCustomizers) {
		return () -> restTemplateCustomizers.ifAvailable(customizers -> {
            for (RestTemplate restTemplate : LoadBalancerAutoConfiguration.this.restTemplates) {
                for (RestTemplateCustomizer customizer : customizers) {
                    customizer.customize(restTemplate);
                }
            }
        });
	}

	@Autowired(required = false)
	private List<LoadBalancerRequestTransformer> transformers = Collections.emptyList();

    /**
    * 创建LoadBalancerRequest供LoadBalancerInterceptor使用
    */
	@Bean
	@ConditionalOnMissingBean
	public LoadBalancerRequestFactory loadBalancerRequestFactory(
			LoadBalancerClient loadBalancerClient) {
		return new LoadBalancerRequestFactory(loadBalancerClient, transformers);
	}

    /**
    * LoadBalancerInterceptorConfig维护LoadBalancerInterceptor和RestTemplateCustomizer的实例。
    * LoadBalancerInterceptor拦截每一个HTTP请求，将其绑定至负载均衡的生命周期。
    * RestTemplateCustomizer为每一个restTemplate绑定LoadBalancerInterceptor拦截器。
    * 
    */
	@Configuration
	@ConditionalOnMissingClass("org.springframework.retry.support.RetryTemplate")
	static class LoadBalancerInterceptorConfig {
		@Bean
		public LoadBalancerInterceptor ribbonInterceptor(
				LoadBalancerClient loadBalancerClient,
				LoadBalancerRequestFactory requestFactory) {
			return new LoadBalancerInterceptor(loadBalancerClient, requestFactory);
		}

		@Bean
		@ConditionalOnMissingBean
		public RestTemplateCustomizer restTemplateCustomizer(
				final LoadBalancerInterceptor loadBalancerInterceptor) {
			return restTemplate -> {
                List<ClientHttpRequestInterceptor> list = new ArrayList<>(
                        restTemplate.getInterceptors());
                list.add(loadBalancerInterceptor);
                restTemplate.setInterceptors(list);
            };
		}
	}

	@Configuration
	@ConditionalOnClass(RetryTemplate.class)
	public static class RetryAutoConfiguration {

		@Bean
		@ConditionalOnMissingBean
		public LoadBalancedRetryFactory loadBalancedRetryFactory() {
			return new LoadBalancedRetryFactory() {};
		}
	}

	@Configuration
	@ConditionalOnClass(RetryTemplate.class)
	public static class RetryInterceptorAutoConfiguration {
		@Bean
		@ConditionalOnMissingBean
		public RetryLoadBalancerInterceptor ribbonInterceptor(
				LoadBalancerClient loadBalancerClient, LoadBalancerRetryProperties properties,
				LoadBalancerRequestFactory requestFactory,
				LoadBalancedRetryFactory loadBalancedRetryFactory) {
			return new RetryLoadBalancerInterceptor(loadBalancerClient, properties,
					requestFactory, loadBalancedRetryFactory);
		}

		@Bean
		@ConditionalOnMissingBean
		public RestTemplateCustomizer restTemplateCustomizer(
				final RetryLoadBalancerInterceptor loadBalancerInterceptor) {
			return restTemplate -> {
                List<ClientHttpRequestInterceptor> list = new ArrayList<>(
                        restTemplate.getInterceptors());
                list.add(loadBalancerInterceptor);
                restTemplate.setInterceptors(list);
            };
		}
	}
}
```
>>该配置加载的条件为：1.当前上下文环境必须有RestTemplate实例，2.该上下文环境必须初始化了LoadBalancerClient的实现类。
>
```java
/**
* ClientHttpRequestInterceptor是spring维护的拦截器。
*/
public class LoadBalancerInterceptor implements ClientHttpRequestInterceptor {

	private LoadBalancerClient loadBalancer;// LoadBalancerClient该接口仅一个实现类RibbonLoadBalancerClient
	private LoadBalancerRequestFactory requestFactory;

	public LoadBalancerInterceptor(LoadBalancerClient loadBalancer, LoadBalancerRequestFactory requestFactory) {
		this.loadBalancer = loadBalancer;
		this.requestFactory = requestFactory;
	}

	public LoadBalancerInterceptor(LoadBalancerClient loadBalancer) {
		// for backwards compatibility
		this(loadBalancer, new LoadBalancerRequestFactory(loadBalancer));
	}

    /**
    * 在RestTemplate中使用的URI为http://myservice/path/to/service
    * getHost--->myservice
    */
	@Override
	public ClientHttpResponse intercept(final HttpRequest request, final byte[] body,
			final ClientHttpRequestExecution execution) throws IOException {
		final URI originalUri = request.getURI();
		String serviceName = originalUri.getHost();
		Assert.state(serviceName != null, "Request URI does not contain a valid hostname: " + originalUri);
		return this.loadBalancer.execute(serviceName, requestFactory.createRequest(request, body, execution));
	}
}
```
>LoadBalancerClient该接口仅一个实现类RibbonLoadBalancerClient
```java
public class RibbonLoadBalancerClient implements LoadBalancerClient {
    
    @Override
    	public <T> T execute(String serviceId, LoadBalancerRequest<T> request) throws IOException {
            //1. 得到一个ILoadBalancer
    		ILoadBalancer loadBalancer = getLoadBalancer(serviceId);
    		//2. 由ILoadBalancer得到一个Server，该处就是发生负载均衡的地方。
    		Server server = getServer(loadBalancer);
    		if (server == null) {
    			throw new IllegalStateException("No instances available for " + serviceId);
    		}
    		RibbonServer ribbonServer = new RibbonServer(serviceId, server, isSecure(server,
    				serviceId), serverIntrospector(serviceId).getMetadata(server));
    
    		return execute(serviceId, ribbonServer, request);
    	}
    
    	@Override
    	public <T> T execute(String serviceId, ServiceInstance serviceInstance, LoadBalancerRequest<T> request) throws IOException {
    		Server server = null;
    		if(serviceInstance instanceof RibbonServer) {
    			server = ((RibbonServer)serviceInstance).getServer();
    		}
    		if (server == null) {
    			throw new IllegalStateException("No instances available for " + serviceId);
    		}
    
    		RibbonLoadBalancerContext context = this.clientFactory
    				.getLoadBalancerContext(serviceId);
    		RibbonStatsRecorder statsRecorder = new RibbonStatsRecorder(context, server);
    
    		try {
    			T returnVal = request.apply(serviceInstance);
    			statsRecorder.recordStats(returnVal);
    			return returnVal;
    		}
    		// catch IOException and rethrow so RestTemplate behaves correctly
    		catch (IOException ex) {
    			statsRecorder.recordStats(ex);
    			throw ex;
    		}
    		catch (Exception ex) {
    			statsRecorder.recordStats(ex);
    			ReflectionUtils.rethrowRuntimeException(ex);
    		}
    		return null;
    	}
        protected Server getServer(ILoadBalancer loadBalancer) {
            if (loadBalancer == null) {
                return null;
            }
            return loadBalancer.chooseServer("default"); // TODO: better handling of key
        }
        public Server chooseServer(Object key) {
            if (counter == null) {
                counter = createCounter();
            }
            counter.increment();
            if (rule == null) {
                return null;
            } else {
                try {
                    // rule 就是IRule，即负载均衡选择
                    return rule.choose(key);
                } catch (Exception e) {
                    logger.warn("LoadBalancer [{}]:  Error choosing server for key {}", name, key, e);
                    return null;
                }
            }
        }
}
```

>hystrix
>>hystrix是针对分布式系统`容错处理`的开源组件，hystrix是一个延迟和容错库，旨在隔离远程系统，服务和第三方库，阻止级联故障，在复杂的分布式系统中实现故障恢复。
>Hystrix被设计的目标是:
>>1. 对通过第三方客户端库访问的依赖项（通常是通过网络）的延迟和故障进行保护和控制。
>>2. 在一个复杂的分布式系统中阻止级联故障。
>>3. 快速失败，快速恢复。
>>4. 在合理的情况下回退和优雅地降级。
>>5. 启用近实时监控、警报和操作控制。
>>hystrix底层大量使用Rxjava  --------> angular RxJS

>使用
>>1. 加入依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
>>2. 启动断路器模式,在启动类上添加@EnableHystrix
>>3. 在需要的方法上增加@HystrixCommand和降级fallback
```java
@Service
public class UserService {
    @HystrixCommand(fallbackMethod = "defaultUser")
    public String getUser(String userName) throws Exception {
        if ("S".equals(userName)){
            return  "okkk";
        }else{
            throw  new Exception();
        }
    }
    /**
     * getUser 出错时调用该方法，预返回友好提示
     * @param userName
     * @return
     */
    public String defaultUser(String userName){
        return "1221323--->"+userName;
    }
}
```
>Feign中使用断路器
>>在Feign中，默认是自带hystrix功能的，在老版本中是默认开启的，后续版本中已默认关闭，需要在配置文件中打开。
>>1. 使用@FeignClient定义接口，配置降级回退类ClientServiceFallBack
>>2. 启用hystrix，feign-hystrix-enabled = true
>>3. 创建降级fallback类
##### hystrix异常机制和处理
Hystrix 的异常处理中，有5种出错的情况下会被 fallback 所截获，从而触发 fallback：
* FAILURE：执行失败，抛出异常。
* TIMEOUT：执行超时。
* SHORT_CIRCUITED：断路器打开。
* THREAD_POOL_REJECTED：线程池拒绝。
* SEMAPHORE_REJECTED：信号量拒绝。
有一种类型的异常是不会触发 fallback 且不会被计数进入熔断的，它是 BAD_REQUEST，会抛出 HystrixBadRequestException，
这种异常一般对应的是由非法参数或者一些非系统异常引起的，对于这类异常可以根据响应创建对应的异常进行异常封装或者直接处理。

##### hystrix 请求缓存
使用HystrixCommand有2种方式，第一种是继承，另一种是直接注解。
hystrix的缓存在一次请求内有效，这要求请求在一个hystrix上下文中，故需要先初始化hystrix上下文，可以使用filter和intercept进行
初始化。
1. 初始化HystrixRequestContext
2. 使用类开启缓存，只要继承HystrixCommand，重新getCache，增加clean方法。
3. 注册拦截器
```java
//1. 初始化 HystrixRequestContext 本次通过拦截器方式，还可以通过filter来实现
public class CacheInterceptor implements HandlerInterceptor {
    //初始化Hystrix请求上下文
    HystrixRequestContext context ;
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        context= HystrixRequestContext.initializeContext();
        log.info("--------》 HystrixRequestContext.initializeContext end ");
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        context.shutdown();
        log.info("--------》 HystrixRequestContext.initializeContext shutdown ");
    }
}

// 2. 使用类开启缓存，只要继承HystrixCommand，重新getCache，增加clean方法。
public class CacheCommand extends HystrixCommand {
    private static final HystrixCommandKey GETTER_KEY= HystrixCommandKey.Factory.asKey("CommandKey");
    private RestTemplate restTemplate;
    private String string;

    public CacheCommand(RestTemplate restTemplate, String string) {
        super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("myCacheCommand")).andCommandKey(GETTER_KEY));
        this.restTemplate = restTemplate;
        this.string = string;
    }
    @Override
    protected Object run() throws Exception {
        String json = restTemplate.getForObject("http://service-provider/testHystrixCache/{string}",String.class,string);
        log.info("------------>" + json);
        return json;
    }

    @Override
    protected String getCacheKey() {
        return this.string;
    }
    public static void flushCache(String string){
        HystrixRequestCache.getInstance(GETTER_KEY, HystrixConcurrencyStrategyDefault.getInstance()).clear(string);
    }
}

//3. 注册拦截器
public class WebComponent2Config {
    @Autowired
    public CacheInterceptor cacheInterceptor;

    @Bean
    public WebMvcConfigurer webMvcConfigurer(){
        WebMvcConfigurer webMvcConfigurer = new WebMvcConfigurer() {
            @Override
            public void addInterceptors(InterceptorRegistry registry) {
                registry.addInterceptor(cacheInterceptor).addPathPatterns("/**");
            }
        };
        return webMvcConfigurer;
    }
}

// 4.1 test 使用类开启缓存
public class CacheController {
    @Autowired
    private RestTemplate restTemplate;
    @GetMapping(value = "getFromCache/{string}")
    public String getFromCache(@PathVariable("string") String string){
        CacheCommand cacheCommand = new CacheCommand(restTemplate,string);
        cacheCommand.execute();
        log.info("from cache : "+ cacheCommand.isResponseFromCache());// false
        CacheCommand cacheCommand1 = new CacheCommand(restTemplate,string);
        cacheCommand1.execute();
        log.info("from cache : "+ cacheCommand1.isResponseFromCache()); // true
        return "ok";
    }
}

// 4.2 test 使用注解开启缓存
// 4.2.1 新增CacheService
public class CacheService {
    @Autowired
    public RestTemplate restTemplate;

    @HystrixCommand(commandKey = "AAA")
    @CacheResult
    public String getFromCache(@CacheKey String str){
        String json = restTemplate.getForObject("http://service-provider/testHystrixCache/{string}",String.class,str);
        log.info("@@@@@@@@@@@@@@@@@@------------>" + json);
        return json;
    }
    @CacheRemove(commandKey = "AAA")
    @HystrixCommand
    public String update(@CacheKey String str){
        return "okk";
    }
}
// 4.2.2 test 使用注解开启缓存
@RestController
@Slf4j
public class CacheController {
    @Autowired
    private RestTemplate restTemplate;
    @Autowired
    private CacheService cacheService;

    @GetMapping(value = "getFromCache/{string}")
    public String getFromCache(@PathVariable("string") String string){
        CacheCommand cacheCommand = new CacheCommand(restTemplate,string);
        cacheCommand.execute();
        log.info("from cache : "+ cacheCommand.isResponseFromCache());
        CacheCommand cacheCommand1 = new CacheCommand(restTemplate,string);
        cacheCommand1.execute();
        log.info("from cache : "+ cacheCommand1.isResponseFromCache());
        return "ok";
    }

    @GetMapping(value = "getFromCacheByAnnotation/{string}")
    public String getFromCacheByAnnotation(@PathVariable("string") String string){
        cacheService.getFromCache(string);
        cacheService.getFromCache(string);
        cacheService.update(string);//@CacheRemove(commandKey = "AAA") 清除缓存
        cacheService.getFromCache(string);
        cacheService.getFromCache(string);
        return "ok";
    }
}

// 5. filter 初始化 HystrixRequestContext
/**
 * 2 种注册方式
 * 1. @WebFilter 注解 启动类增加 @ServletComponentScan
 * 2. 通过@Configuration注册
 */
@Slf4j
@WebFilter(filterName = "hystrixRequestContextServletFilter",urlPatterns = "/*",asyncSupported = true)
public class HystrixRequestContextServletFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) {}
    @Override
    public void destroy() {}
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws IOException, ServletException {
        HystrixRequestContext context = HystrixRequestContext.initializeContext();
        try {
            log.info("@@@@@@@@@->HystrixRequestContextServletFilter");
            filterChain.doFilter(request, response);
        } finally {
            context.shutdown();
        }
    }
}

```
##### Hystrix线程传递及并发策略
Hystrix会对请求进行封装，然后管理请求的调用，从而实现断路等功能。
Hystrix提供2种`隔离级别`来进行请求的操作。
* 信号量
* 线程隔离
如果信号量，则Hystrix在请求时会获取一个信号量，如果获取成功则继续执行请求，请求在一个线程中完成。  
如果是线程隔离，Hystrix会把请求放到线程池中执行，这时可能产生线程变化，从而导致线程1的上下文数据在线程2里不能正常获取到。
```java
// 1. 定义测试Controller
@RestController
@Slf4j
public class ThreadContextController {
    @Autowired
    private ThreadContextService threadContextService;
    @GetMapping("/getUser/{id}")
    public String getUser(@PathVariable("id") Integer id){
        // 第一个种情况，放入上下文
        HystrixThreadLocal.threadLocal.set("userId : "+ id);
        // 第二种情况 用RequestContextHolder 存放
        RequestContextHolder.currentRequestAttributes().setAttribute("userId","userId : "+id, RequestAttributes.SCOPE_REQUEST);
        log.info("ThreadContextController ,Current thread : " + Thread.currentThread().getName() + " : id--->  " + Thread.currentThread().getId());
        log.info("ThreadContextController ,ThreadContext object  : " + HystrixThreadLocal.threadLocal.get());
        log.info("ThreadContextController "+RequestContextHolder.currentRequestAttributes().getAttribute("userId",RequestAttributes.SCOPE_REQUEST));
        return threadContextService.getUser(id);
    }
}

// 2. 测试Service，getUser方法是否增加@HystrixCommand
/**
* 2.1 getUser方法不增加@HystrixCommand
* ThreadContextController ,Current thread : http-nio-5000-exec-1 : id--->  42
  ThreadContextController ,ThreadContext object  : userId : 5555
  ThreadContextController userId : 5555
  ThreadContextService ,Current thread : http-nio-5000-exec-1 : id--->  42
  ThreadContextService ,ThreadContext object  : userId : 5555
  ThreadContextService userId : 5555
  
* 2.2 getUser方法增加@HystrixCommand
* ThreadContextController ,Current thread : http-nio-5000-exec-1 : id--->  41
  ThreadContextController ,ThreadContext object  : userId : 5555
  ThreadContextController userId : 5555
  ThreadContextService ,Current thread : hystrix-ThreadContextService-1 : id--->  68
  ThreadContextService ,ThreadContext object  : null
  java.lang.IllegalStateException: No thread-bound request found
  
  2.3 加上HystrixCommand注解后，说明线程隔离已生效，线程id由41变化为68，是重新新建线程去请求的，线程变量也丢失了。
  2.4 解决方案： 
               A: Hystrix默认是线程隔离，可以修改Hystrix隔离策略，使用信号量，可通过修改配置文件实现。不推荐，因为真实项目大部分均使用线程隔离。（hystrix.command.default.execution.isolation.strategy= THREAD|SEMAPHORE）
               B: 官方推荐的一种方式：使用HystrixConcurrencyStrategy
*/          
@Slf4j 
@Service
public class ThreadContextService {
    @Autowired
    private RestTemplate restTemplate;
    @HystrixCommand
    public String getUser(Integer id) {
        log.info("ThreadContextService ,Current thread : " + Thread.currentThread().getName() + " : id--->  " + Thread.currentThread().getId());
        log.info("ThreadContextService ,ThreadContext object  : " + HystrixThreadLocal.threadLocal.get());
        log.info("ThreadContextService "+RequestContextHolder.currentRequestAttributes().getAttribute("userId",RequestAttributes.SCOPE_REQUEST));
        String json = restTemplate.getForObject("http://service-provider/testHystrixCache/{string}", String.class, id);
        log.info("@@@@@@@@@@@@@@@@@@------------>ThreadContextService :" + json);
        return json;
    }
}
```
##### 使用HystrixConcurrencyStrategy
我们可以声明一个定制化的HystrixConcurrencyStrategy实例，并通过HystrixPlugins注册。
被@HystrixCommand注解的方法，其执行源Callable可以通过wrapCallable方法进行定制化装饰，加入附加的行为。

编写自定义并发策略只需编写一个类，让其继承HystrixConcurrencyStrategy ，并重写wrapCallable 方法即可。可以
提供重写这个方法来封装自己想要的线程参数。
该类提供一个在请求执行前包装的计划，可以注入自己定义的动作，比如复制线程状态等附加行为。
HystrixPlugins帮助我们注册自定义的Plugin。
1. 定义一个TestHystrixConcurrencyStrategy实现HystrixConcurrencyStrategy接口，并重写其wrapCallable方法：
```java
@Component
@Slf4j
public class TestHystrixConcurrencyStrategy extends HystrixConcurrencyStrategy {
    @Override
    public <T> Callable<T> wrapCallable(Callable<T> callable) {
        return new HystrixThreadCallable<>(callable, RequestContextHolder.getRequestAttributes(), HystrixThreadLocal.threadLocal.get());
    }

    static class HystrixThreadCallable<T> implements Callable<T> {
        private final Callable<T> target;
        private final RequestAttributes requestAttributes;
        private String s ;
        public HystrixThreadCallable(Callable<T> target, RequestAttributes requestAttributes, String s) {
            this.target = target;
            this.requestAttributes = requestAttributes;
            this.s = s;
        }
        @Override
        public T call() throws Exception {
            try {
                RequestContextHolder.setRequestAttributes(requestAttributes);
                HystrixThreadLocal.threadLocal.set(s);
                log.warn("----------------> TestHystrixConcurrencyStrategy call()");
                return target.call();
            }
            finally {
                RequestContextHolder.resetRequestAttributes();
                HystrixThreadLocal.threadLocal.remove();
            }
        }
    }
}
```
2. 同样在任意配置类中添加如下代码段即可，通过HystrixPlugins注册RequestContextHystrixConcurrencyStrategy：
```java
@PostConstruct
public void init() {
    HystrixPlugins.getInstance().registerConcurrencyStrategy(new TestHystrixConcurrencyStrategy());
}
```
至此隔离策略为线程隔离时，也能正常请求。
[参考文章](https://blog.csdn.net/songhaifengshuaige/article/details/80345012)  
HystrixPlugins的registerConcurrencyStrategy方法只能被调用一次，将导致无法和其他并发策略一起使用。
```java
public void registerConcurrencyStrategy(HystrixConcurrencyStrategy impl) {
    if (!concurrencyStrategy.compareAndSet(null, impl)) {
        throw new IllegalStateException("Another strategy was already registered.");
    }
}
```
如何达到其他策略并存？  
具体做法是： 在构造此并发策略时，找到之前已经存在的并发策略，并保留在类的属性中，在调用过程中，返回之前并发策略的相关信息，如 请求变量
，连接池，阻塞队列等请求进来时，既不影响之前的并发策略，也可以包装需要的请求信息。  
参考：`HystrixSecurityAutoConfiguration`,`SecurityContextConcurrencyStrategy`  
```java
/**
 * 将现有的并发策略作为新并发策略的成员变量
 * 在新并发策略中，返回现有并发策略的线程池、Queue等
 */
@Component
@Slf4j
public class TestHystrixConcurrencyStrategy extends HystrixConcurrencyStrategy {
    private HystrixConcurrencyStrategy delegate ;

    public TestHystrixConcurrencyStrategy(){

        this.delegate  = HystrixPlugins.getInstance().getConcurrencyStrategy();
        if (this.delegate instanceof TestHystrixConcurrencyStrategy) {
            return;
        }
        // Keeps references of existing Hystrix plugins.
        HystrixEventNotifier eventNotifier = HystrixPlugins.getInstance()
                .getEventNotifier();
        HystrixMetricsPublisher metricsPublisher = HystrixPlugins.getInstance()
                .getMetricsPublisher();
        HystrixPropertiesStrategy propertiesStrategy = HystrixPlugins.getInstance()
                .getPropertiesStrategy();
        HystrixCommandExecutionHook commandExecutionHook = HystrixPlugins.getInstance()
                .getCommandExecutionHook();

        HystrixPlugins.reset();

        // Registers existing plugins excepts the Concurrent Strategy plugin.
        HystrixPlugins.getInstance().registerConcurrencyStrategy(this);
        HystrixPlugins.getInstance().registerEventNotifier(eventNotifier);
        HystrixPlugins.getInstance().registerMetricsPublisher(metricsPublisher);
        HystrixPlugins.getInstance().registerPropertiesStrategy(propertiesStrategy);
        HystrixPlugins.getInstance().registerCommandExecutionHook(commandExecutionHook);
    }

    @Override
    public <T> Callable<T> wrapCallable(Callable<T> callable) {
        return new HystrixThreadCallable<>(callable, RequestContextHolder.getRequestAttributes(), HystrixThreadLocal.threadLocal.get());
    }

    static class HystrixThreadCallable<T> implements Callable<T> {
        private final Callable<T> target;
        private final RequestAttributes requestAttributes;
        private String s ;
        public HystrixThreadCallable(Callable<T> target, RequestAttributes requestAttributes, String s) {
            this.target = target;
            this.requestAttributes = requestAttributes;
            this.s = s;
        }
        @Override
        public T call() throws Exception {
            try {
                RequestContextHolder.setRequestAttributes(requestAttributes);
                HystrixThreadLocal.threadLocal.set(s);
                log.warn("----------------> TestHystrixConcurrencyStrategy call()");
                return target.call();
            }
            finally {
                RequestContextHolder.resetRequestAttributes();
                HystrixThreadLocal.threadLocal.remove();
            }
        }
    }

    @Override
    public BlockingQueue<Runnable> getBlockingQueue(int maxQueueSize) {
        return this.delegate.getBlockingQueue(maxQueueSize);
    }

    @Override
    public ThreadPoolExecutor getThreadPool(HystrixThreadPoolKey threadPoolKey, HystrixThreadPoolProperties threadPoolProperties) {
        return delegate.getThreadPool(threadPoolKey, threadPoolProperties);
    }

    @Override
    public ThreadPoolExecutor getThreadPool(HystrixThreadPoolKey threadPoolKey, HystrixProperty<Integer> corePoolSize, HystrixProperty<Integer> maximumPoolSize, HystrixProperty<Integer> keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
        return delegate.getThreadPool(threadPoolKey, corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
    }

    @Override
    public <T> HystrixRequestVariable<T> getRequestVariable(HystrixRequestVariableLifecycle<T> rv) {
        return this.delegate.getRequestVariable(rv);
    }
}
```
##### Hystrix命令注解
*  HystrixCommand
用于封装执行代码，然后具有故障延迟容错，断路器等功能，但它是阻塞命令，可以和Obervable共用。
* HystrixObservableCommand
同HystrixCommand，主要区别是非阻塞的调用模式。

都支持故障延迟容错，断路器，指标统计等功能。

区别：
* HystrixCommand默认是阻塞的，可以支持同步和异步2种方式，HystrixObservableCommand默认是非阻塞，默认只能异步。
* HystrixCommand 的方法是run,HystrixObservableCommand执行的是construct。
* HystrixCommand 的一个实例只能发出一条数据，HystrixObservableCommand 可以发送多条数据。

>HystrixCommand 属性：
>>commandKey 全局唯一标志符，如果不配置，默认方法名。  
>>defaultFallback 默认的fallback方法，该函数不能有入参，返回值和Hystrix方法保持一致，但是fallbackMethod优先级更高。
>>FallbackMethod 方法执行时熔断、错误、超时时会执行的回退方法，需要保持此方法与 Hystrix 方法(@HystrixCommand被该注解的方法)的签名和返回值一致。
>>ignoreExceptions 哪些异常需要被忽略，不进行fallback。
>>commandProperties 配置一些命令的属性，如隔离级别等。
>>threadPoolProperties 配置hystrix依赖的线程池相关参数。
>>groupKey 全局唯一标识服务分组的名称，一般会再创建HystrixCommand时指定命令组来实现默认的线程池划分。
>>threadPoolKey 对服务线程池信息进行设置，用于HystrixThreadPool 监控，缓存等。

##### Spring cloud Zuul
Zuul作为微服务网关，是一个面向服务治理，访问编排的组件。
Zuul是从设备和网站到后端程序所有请求的前门，为内部服务提供可配置的对外URL到服务的映射关系。
* 认证和鉴权
* 压力控制
* 金丝雀测试
* 动态路由
* 负载消弱
* 静态响应处理
* 主动流量管理
其底层基于Servlet，本质组件是一系列有Filter构成的责任链。zuul 1.x   2.x（底层采用netty）
1. 导入pom依赖
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-start-netflix-zuul</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
</dependencies>
```
2. 编写启动类`@EnableDiscoveryClient,@EnableZuulProxy`
3. 配置文件
>serviceId单实例配置
>>
```yaml
zuul:
       routes:
         service-invoker:
                     path: /client/**
                     serviceId: service-invoker
# 可以简化为
zuul:
       routes:
         service-invoker: /client/**

zuul:
       routes:
         service-invoker:
# 等价于
zuul:
       routes:
         service-invoker:
                     path: /service-invoker/**
                     serviceId: service-invoker
```
>单实例URL配置
>>除了路由到服务外，还可以路由到物理地址，将serviceId替换为url
```yaml
zuul:
       routes:
         service-invoker:
                     path: /client/**
                     url: http://localhost:5000 # service-invoker地址
```
>多实例路由
>>默认会使用Eureka中集成的负载均衡功能，如果想要使用Ribbon提供的负载均衡功能，就需要指定一个serviceId,此操作需要禁止Ribbon使用Eureka。

>forward本地跳转,client1为zuul-server的mapping
```yaml
zuul:
       prefix: /pre #指定前缀
       routes:
         service-invoker:
                     path: /client1/**
                     url: forward:/client1
                     stripPrefix: false # 禁用prefix
```
>路由通配符/** 匹配任意数量的路径和字符，/*匹配任意数量的字符，/？匹配单个字符。

###### 路由前缀
```yaml
zuul:
       prefix: /pre #指定前缀
       routes:
         service-invoker:
                     path: /client1/**
                     url: forward:/client1
                     stripPrefix: false # 禁用prefix
```
###### 服务和路径屏蔽
可以避免服务或者路径的侵入
```yaml
```
###### 敏感头信息
zuul可以配置敏感头信息，切断它和下成服务的交互。

###### 重定向 add-host-header: true
```yaml
zuul:
  prefix: /pre #指定前缀
  routes:
    service-invoker:
                path: /client/**
                serviceId: service-invoker
                stripPrefix: false # 禁用prefix
  ignored-patterns: /**/AA/** # 忽略的接口
  ignore-service: serviceName # 忽略的服务
  add-host-header: true
```
###### 重试机制
Zuul可以配合Ribbon(默认集成)来做重试，需要接口保持幂等。
```yaml
zuul:
  retryable: true #开启重试

ribbon:
      MaxAutoRetries: 1 # 同一个服务的重试次数，不包含首次
      MaxAutoRetriesNextServer: 1 #切换相同服务数量
```

网关层需要做的有鉴权，限流，动态路由，文件上传，参数转换，其他逻辑和业务处理。

##### zuul filter 链
>zuul工作原理
>>zuul的核心是由一系列紧密配合工作的Filter来实现，他们可以在http请求或者响应的时候执行相关操作。

>zuul Filter 特性
>>Filter的类型：Filter的类型决定了此Filter在Filter链中的执行顺序。可能是路由动作发生前或者路由动作发生后，可能是路由动作发生时，可能是路由过程中发生异常时，
>>Filter的执行顺序：同一种类型的Filter可以通过设置filterOrder()方法来设置执行顺序。
>>Filter的执行条件：Filter运行需要的标准或者条件。
>>Filter的执行效果：符合某个Filter执行条件，产生的执行效果。

>Zuul内部提供一个动态读取，编译和运行这些Filter的机制。Filter间不能直接通信，在请求线程中会通过RequestContext来共享状态，内部由ThreaLocal实现。

##### zuul filter的请求周期
zuul filter的核心执行流程
```java
try {
      //.....
            try {
                this.preRoute();
            } catch (ZuulException var13) {
                this.error(var13);
                this.postRoute();
                return;
            }

            try {
                this.route();
            } catch (ZuulException var12) {
                this.error(var12);
                this.postRoute();
                return;
            }

            try {
                this.postRoute();
            } catch (ZuulException var11) {
                this.error(var11);
            }
        }
```
Zuul一共有4种生命周期的Filter：
* pre 在Zuul按照规则路由到下级服务之前执行。如果需要对请求进行预处理，鉴权，限流等，应该在此类Filter中实现。
* route 这类Filter是Zuul路由的执行者，是Apache HttpClient或者Netflix Ribbon构建和发送原始http请求的地方，也支持了okHttp.
* post 这类Filter在源服务返回结果或者异常信息发生后执行的，如果需要对返回信息做一些处理，则在此类Filter进行处理。
* error 整个生命周期内如果发送异常，则会进入error Filter,可作为全局异常处理。

Zuul 原生Filter
>Zuul Server 如果使用@EnableZuulProxy注解搭配Springboot Actuator,会多2个管控端点。
>>/routes  返回所有已生成的映射规则
>>/filters 返回当前Zuul Server中所有已注册的生效的Filter

>禁用原生Filter
>>`zuul.<SimpleClassName>.<FilterType>.disable = true`
>>禁用SendErrorFilter,`zuul.SendErrorFilter.error.disable = true`

自定义Filter
>只需要继承ZuulFilter即可，ZuulFilter是一个抽象类，有如下方法需实现：
>>String filterType(): 使用返回值设定Filter类型，可以设置为pre,route,post,error类型。
>>int filterOrder(): 设置Filter的执行顺序
>>boolean shouldFilter： 设置该Filter是否执行，可以作为开关使用
>>Object run()： 核心执行逻辑，业务处理等操作

##### 鉴权
1. 加入pom.xml依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>
```
2. 配置application文件
```yaml
```

##### Zuul 限流
在Hystrix中，可以通过熔断器来实现，到达某个阈值来对异常流量进行降级处理。其实，除了降级处理，我们还可以进行排队，限流，分流等
###### 限流算法
在Zuul中实现限流最简单的方式是使用Filter加上限流算法。其中可能考虑到zuul多节点部署，这时需要一个K/V存储工具。单节点 完全可以放到内存中处理。
**漏桶**
可以认为就是注水漏水过程，桶以一定速率流出水，以任意速率（入大于出）流入水，当水超过桶流量则丢弃，因为桶容量是不变的，保证了整体的速率。
当在入小于出的情况下，漏桶则不起作用。溢出的流量可以被利用起来，比如放到一个队列里，做流量排队。
**令牌桶**
1. 所有的请求在处理之前都需要拿到一个可用的令牌才会被处理；
2. 根据限流大小，设置按照一定的速率往桶里添加令牌；
3. 桶设置最大的放置令牌限制，当桶满时、新添加的令牌就被丢弃活着拒绝；
4. 请求达到后首先要获取令牌桶中的令牌，拿着令牌才可以进行其他的业务逻辑，处理完业务逻辑之后，将令牌直接删除；
5. 令牌桶有最低限额，当桶中的令牌达到最低限额的时候，请求处理完之后将不会删除令牌，以此保证足够的限流；
6. 由于令牌桶中有一定流量，那么就有可能存在一定程度上的流量突发。

**Zuul使用技巧**
1. 饥饿加载 Zuul内部默认使用Ribbon来调用远程服务，初始化ribbon负载信息，是一种懒加载策略。故可以在配置文件中开启饥饿加载。
2. 请求体修改：客户端对Zuul发送post请求后，在请求到下游服务前，需要对请求体进行修改，可以通过新增一个PRE类型的Filter对请求体进行修改。

##### 重试机制
在Spring cloud中有多种发送http请求的方式可以和zuul结合，Ribbon,Feign或者RestTemplate。无论是哪种都可能出现失败的情况，zuul中的重试机制是配合Spring retry
和Ribbon来使用的。
```
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
```

**Header传递**
在使用Zuul构建网关时，在zuul中对请求做了一些处理，需要把处理结果发给下游服务，但是又不能影响原始请求体，
```java
public class HeaderDeliverFilter extends ZuulFilter {
    @Override
    public Object run() throws ZuulException {
        RequestContext context = RequestContext.getCurrentContext();
        context.addZuulRequestHeader("A","B");
        return null;
    }
    // ......
}
```

**OpenResty + Lua 动态增加 Zuul 节点**


```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
默认开启2个。`Exposing 2 endpoint(s) beneath base path '/actuator'`。

@RefreshScope注解，修饰的bean都是延迟加载的，只在第一次访问才初始化。刷新bean也是同理。当config-info配置发生改变时，需要先执行`http://localhost:7000/actuator/refresh`
端点刷新，即**手动刷新配置中心**。

自动刷新配置中心（Spring cloud bus ）
大致流程：用户更新配置文件时，检测到git hook变化，触发git hook配置的地址，config serve 接收到消息并发布消息，Bus将消息发送到client,client接收到消息后，
会重新发送请求加载配置信息。









----174--656





















  














