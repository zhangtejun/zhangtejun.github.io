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













































  














