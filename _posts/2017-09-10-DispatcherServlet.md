---
layout: post
title:  "DispatcherServlet"
date:   2017-09-10 10:29:00
author: zhangtejun
categories: Spring
---
##### DispatcherServlet
用于HTTP请求处理程序/控制器的中央调度程序。

Web应用程序可以定义任意数量的DispatcherServlet。 每个servlet将在自己的命名空间中运行，
使用映射，处理程序等加载自己的应用程序上下文。只有通过加载的根应用程序上下文 ContextLoaderListener才能被共享。

![DispatcherServlet]({{ site.dispatcher | prepend: site.baseurl }})


###### HandlerMapping
public interface HandlerMapping

Interface to be implemented by objects that define a mapping between requests and handler objects.

由定义请求和处理程序对象之间的映射的对象实现的接口。

This class can be implemented by application developers, although this is not necessary, as BeanNameUrlHandlerMapping and RequestMappingHandlerMapping are included in the framework. The former is the default if no HandlerMapping bean is registered in the application context.

该类可以由应用程序开发人员实现，尽管这不是必需的，因为框架中包含BeanNameUrlHandlerMapping和RequestMappingHandlerMapping。如果在应用程序上下文中没有注册HandlerMapping bean，则前者是默认值。

HandlerMapping implementations can support mapped interceptors but do not have to. A handler will always be wrapped in a HandlerExecutionChain instance, optionally accompanied by some HandlerInterceptor instances. The DispatcherServlet will first call each HandlerInterceptor's preHandle method in the given order, finally invoking the handler itself if all preHandle methods have returned true.

HandlerMapping实现可以支持映射拦截器，但不是必须的。一个处理器（Handler）总是把请求传递到一个HandlerExecutionChain实例中，
也可以选择包含一些HandlerInterceptor（拦截器）实例。 DispatcherServlet首先按给定的顺序调用每个HandlerInterceptor
的preHandle方法，如果所有的preHandle方法都返回true，最后调用处理程序本身。

The ability to parameterize this mapping is a powerful and unusual capability of this MVC framework. For example, it is possible to write a custom mapping based on session state, cookie state or many other variables. No other MVC framework seems to be equally flexible.

参数化此映射的功能是此MVC框架的强大且不寻常的功能。例如，可以根据会话状态，Cookie状态或许多其他变量编写自定义映射。没有其他的MVC框架似乎同样灵活。

Note: Implementations can implement the Ordered interface to be able to specify a sorting order and thus a priority for getting applied by DispatcherServlet. Non-Ordered instances get treated as lowest priority.

注意，实现Ordered接口的实例可以指定一个排序的order，从而实现DispatcherServlet应用的优先级。没有指定order的实例将被当作最低优先级来处理。

##### HandlerInterceptor接口
允许自定义处理程序执行链的工作流界面。应用程序可以为某些处理程序组注册任意数量的现有或自定义拦截器，以添加常见的预处理行为，而无需修改每个处理程序实现。

HandlerInterceptor在适当的HandlerAdapter触发执行处理程序本身之前被调用。这种机制可以用于大量的预处理方面，例如用于授权检查，或常见的处理程序行为，如区域设置或主题更改。其主要目的是允许分解重复的处理程序代码。

在异步处理场景中，处理程序可以在单独的线程中执行，而主线程在不呈现或调用postHandle和afterCompletion回调的情况下退出 。当并发处理程序执行完成时，请求被发回以继续呈现模型，并且再次调用此合同的所有方法。有关进一步的选项和细节请参阅 org.springframework.web.servlet.AsyncHandlerInterceptor

通常每个HandlerMapping bean定义拦截器链，共享其粒度。为了能够将某个拦截器链应用于一组处理程序，需要通过一个HandlerMapping bean映射所需的处理程序。拦截器本身被定义为应用程序上下文中的bean，由映射bean定义通过其“interceptors”属性（在XML中：<ref>的<list>）引用。

HandlerInterceptor基本上类似于Servlet Filter，但与后者相反，它只允许自定义预处理，可以禁止执行处理程序本身以及定制后处理。过滤器更强大，例如它们允许交换链中传递的请求和响应对象。请注意，过滤器在应用程序上下文中的web.xml（一个HandlerInterceptor）中配置。

作为基本指导原则，细粒度处理程序相关的预处理任务是HandlerInterceptor实现的候选者，特别是经过考虑的常见处理程序代码和授权检查。另一方面，过滤器非常适合请求内容和查看内容处理，如多部分表单和GZIP压缩。这通常显示何时需要将过滤器映射到某些内容类型（例如图像）或所有请求。

##### HandlerExecutionChain类
```

/**
 * 处理程序执行链，由处理程序对象和任何处理程序拦截器组成。
 * 由HandlerMapping的HandlerMapping.getHandler(javax.servlet.http.HttpServletRequest)方法返回。
 * Handler execution chain, consisting of handler object and any handler interceptors.
 * Returned by HandlerMapping's {@link HandlerMapping#getHandler} method.
 */
public class HandlerExecutionChain {

	private static final Log logger = LogFactory.getLog(HandlerExecutionChain.class);

  //处理器对象
	private final Object handler;

  //所有的HandlerInterceptor的数组
	private HandlerInterceptor[] interceptors;
 
 //所有的HandlerInterceptor的链表
	private List<HandlerInterceptor> interceptorList;

	private int interceptorIndex = -1;


	/**
	 * Create a new HandlerExecutionChain.
	 * @param handler the handler object to execute
	 */
	public HandlerExecutionChain(Object handler) {
		this(handler, (HandlerInterceptor[]) null);
	}

	/**
	 * Create a new HandlerExecutionChain.
	 * @param handler the handler object to execute
	 * @param interceptors the array of interceptors to apply
	 * (in the given order) before the handler itself executes
	 */
	public HandlerExecutionChain(Object handler, HandlerInterceptor... interceptors) {
		if (handler instanceof HandlerExecutionChain) {
			HandlerExecutionChain originalChain = (HandlerExecutionChain) handler;
			this.handler = originalChain.getHandler();
			this.interceptorList = new ArrayList<HandlerInterceptor>();
			CollectionUtils.mergeArrayIntoCollection(originalChain.getInterceptors(), this.interceptorList);
			CollectionUtils.mergeArrayIntoCollection(interceptors, this.interceptorList);
		}
		else {
			this.handler = handler;
			this.interceptors = interceptors;
		}
	}


	/**
	 * Return the handler object to execute.
	 * @return the handler object
	 */
	public Object getHandler() {
		return this.handler;
	}

	public void addInterceptor(HandlerInterceptor interceptor) {
		initInterceptorList().add(interceptor);
	}

	public void addInterceptors(HandlerInterceptor... interceptors) {
		if (!ObjectUtils.isEmpty(interceptors)) {
			initInterceptorList().addAll(Arrays.asList(interceptors));
		}
	}

	private List<HandlerInterceptor> initInterceptorList() {
		if (this.interceptorList == null) {
			this.interceptorList = new ArrayList<HandlerInterceptor>();
			if (this.interceptors != null) {
				// An interceptor array specified through the constructor
				this.interceptorList.addAll(Arrays.asList(this.interceptors));
			}
		}
		this.interceptors = null;
		return this.interceptorList;
	}

	/**
	 * Return the array of interceptors to apply (in the given order).
	 * @return the array of HandlerInterceptors instances (may be {@code null})
	 */
	public HandlerInterceptor[] getInterceptors() {
		if (this.interceptors == null && this.interceptorList != null) {
			this.interceptors = this.interceptorList.toArray(new HandlerInterceptor[this.interceptorList.size()]);
		}
		return this.interceptors;
	}


	/**
	 * 执行已经注册的拦截的 preHandle()方法。
	 * 如果返回true,则执行链可以执行下一个拦截器的preHandle()方法或handler自身。
	 * Apply preHandle methods of registered interceptors.
	 * @return {@code true} if the execution chain should proceed with the
	 * next interceptor or the handler itself. Else, DispatcherServlet assumes
	 * that this interceptor has already dealt with the response itself.
	 */
	boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HandlerInterceptor[] interceptors = getInterceptors();
		if (!ObjectUtils.isEmpty(interceptors)) {
			for (int i = 0; i < interceptors.length; i++) {
				HandlerInterceptor interceptor = interceptors[i];
				if (!interceptor.preHandle(request, response, this.handler)) {
					triggerAfterCompletion(request, response, null);
					return false;
				}
				this.interceptorIndex = i;
			}
		}
		return true;
	}

	/**
	 * Apply postHandle methods of registered interceptors.
	 */
	void applyPostHandle(HttpServletRequest request, HttpServletResponse response, ModelAndView mv) throws Exception {
		HandlerInterceptor[] interceptors = getInterceptors();
		if (!ObjectUtils.isEmpty(interceptors)) {
			for (int i = interceptors.length - 1; i >= 0; i--) {
				HandlerInterceptor interceptor = interceptors[i];
				interceptor.postHandle(request, response, this.handler, mv);
			}
		}
	}

	/**
	 * Trigger afterCompletion callbacks on the mapped HandlerInterceptors.
	 * Will just invoke afterCompletion for all interceptors whose preHandle invocation
	 * has successfully completed and returned true.
	 */
	void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, Exception ex)
			throws Exception {

		HandlerInterceptor[] interceptors = getInterceptors();
		if (!ObjectUtils.isEmpty(interceptors)) {
			for (int i = this.interceptorIndex; i >= 0; i--) {
				HandlerInterceptor interceptor = interceptors[i];
				try {
					interceptor.afterCompletion(request, response, this.handler, ex);
				}
				catch (Throwable ex2) {
					logger.error("HandlerInterceptor.afterCompletion threw exception", ex2);
				}
			}
		}
	}

	/**
	 * Apply afterConcurrentHandlerStarted callback on mapped AsyncHandlerInterceptors.
	 */
	void applyAfterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response) {
		HandlerInterceptor[] interceptors = getInterceptors();
		if (!ObjectUtils.isEmpty(interceptors)) {
			for (int i = interceptors.length - 1; i >= 0; i--) {
				if (interceptors[i] instanceof AsyncHandlerInterceptor) {
					try {
						AsyncHandlerInterceptor asyncInterceptor = (AsyncHandlerInterceptor) interceptors[i];
						asyncInterceptor.afterConcurrentHandlingStarted(request, response, this.handler);
					}
					catch (Throwable ex) {
						logger.error("Interceptor [" + interceptors[i] + "] failed in afterConcurrentHandlingStarted", ex);
					}
				}
			}
		}
	}


	/**
	 * Delegates to the handler's {@code toString()}.
	 */
	@Override
	public String toString() {
		if (this.handler == null) {
			return "HandlerExecutionChain with no handler";
		}
		StringBuilder sb = new StringBuilder();
		sb.append("HandlerExecutionChain with handler [").append(this.handler).append("]");
		if (!CollectionUtils.isEmpty(this.interceptorList)) {
			sb.append(" and ").append(this.interceptorList.size()).append(" interceptor");
			if (this.interceptorList.size() > 1) {
				sb.append("s");
			}
		}
		return sb.toString();
	}

}
```


##### AbstractHandlerMapping 抽象基类
```jav
package org.springframework.web.servlet.handler;

import java.io.IOException;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Map;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanFactoryUtils;
import org.springframework.core.Ordered;
import org.springframework.util.AntPathMatcher;
import org.springframework.util.Assert;
import org.springframework.util.PathMatcher;
import org.springframework.web.HttpRequestHandler;
import org.springframework.web.context.request.WebRequestInterceptor;
import org.springframework.web.context.support.WebApplicationObjectSupport;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.CorsProcessor;
import org.springframework.web.cors.CorsUtils;
import org.springframework.web.cors.DefaultCorsProcessor;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.servlet.HandlerExecutionChain;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.HandlerMapping;
import org.springframework.web.util.UrlPathHelper;

/**
 * 实现HandlerMapping接口的抽象基类。 支持排序，默认处理程序，处理程序拦截器，包括由路径模式映射的处理程序拦截器。
 * 
 * 注意：此基类不支持HandlerMapping.PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE。 支持此属性取决于具体的子类，通常基于请求URL映射。
 * 所有HandlerMapping实现不需要该属性。基于URL的HandlerMappings通常将支持它，但处理程序不一定期望在所有情况下都存在该请求属性。
 */
public abstract class AbstractHandlerMapping extends WebApplicationObjectSupport implements HandlerMapping, Ordered {

/**
* Spring会按照order属性值 (见org.springframework.core.Ordered接口) 对context里的所有处理器映射进行排序，并且应用第一个匹配的处理器。 
* @param order
*/
	private int order = Integer.MAX_VALUE;  // default: same as non-Ordered


/**
* 默认的处理器。当没有合适的处理器可以匹配请求时，这个处理器就会被使用。
* @param defaultHandler
*/
	private Object defaultHandler;

	private UrlPathHelper urlPathHelper = new UrlPathHelper();

	private PathMatcher pathMatcher = new AntPathMatcher();

	private final List<Object> interceptors = new ArrayList<Object>();

  //拦截器数组
	private final List<HandlerInterceptor> adaptedInterceptors = new ArrayList<HandlerInterceptor>();

	private CorsProcessor corsProcessor = new DefaultCorsProcessor();

	private final UrlBasedCorsConfigurationSource corsConfigSource = new UrlBasedCorsConfigurationSource();


	/**
	 * 指定HandlerMapping bean的order值。
	 * 默认值是Integer.MAX_VALUE，这意味着它是无序的。
	 * Specify the order value for this HandlerMapping bean.
	 * Default value is {@code Integer.MAX_VALUE}, meaning that it's non-ordered.
	 * @see org.springframework.core.Ordered#getOrder()
	 */
	public final void setOrder(int order) {
	  this.order = order;
	}

	@Override
	public final int getOrder() {
	  return this.order;
	}

	/**
	 * 设置默认 handler
	 * Set the default handler for this handler mapping.
	 * This handler will be returned if no specific mapping was found.
	 * <p>Default is {@code null}, indicating no default handler.
	 */
	public void setDefaultHandler(Object defaultHandler) {
		this.defaultHandler = defaultHandler;
	}

	/**
	 * 返回此处理程序映射的默认处理程序，如果没有,则返回null 
	 * Return the default handler for this handler mapping,
	 * or {@code null} if none.
	 */
	public Object getDefaultHandler() {
		return this.defaultHandler;
	}

	/**
	 * 如果这个属性被设成true，Spring 将会使用绝对路径在当前的servlet context中寻找合适的处理器。
	 * 这个属性的默认值是false，在这种情况下，Spring会使用当前servlet context中的相对路径。
	 * 例如，如果一个servlet在servlet-mapping中用的值是/testing/*，当alwaysUseFullPath 设成true时，
	 * 处理器映射中的URL格式应该使用/testing/viewPage.html，
	 * 当这个属性设成false, 同一个URL应该写成 /viewPage.html。
	 * 
	 * Set if URL lookup should always use the full path within the current servlet
	 * context. Else, the path within the current servlet mapping is used if applicable
	 * (that is, in the case of a ".../*" servlet mapping in web.xml).
	 * <p>Default is "false".
	 * @see org.springframework.web.util.UrlPathHelper#setAlwaysUseFullPath
	 */
	public void setAlwaysUseFullPath(boolean alwaysUseFullPath) {
		this.urlPathHelper.setAlwaysUseFullPath(alwaysUseFullPath);
		this.corsConfigSource.setAlwaysUseFullPath(alwaysUseFullPath);
	}

	/**
	 * 这个属性的默认值是false。HttpServletRequest返回未解码的访问URL和URI。HttpServletRequest中请求的URL和URI还保留在HTTP协议所定义编码状态，
	 * 如果你想在HandlerMapping使用它们发现合适的处理器之前对URL进行解码，你应该把这个属性设成true （注意这需要JDK 1.4的支持）。
	 * 解码方法会选用HTTP请求中指定的编码格式，或缺省的ISO-8859-1编码方法。 HTTP请求中一般会声明编码的格式，如果没有的话，默认值是ISO-8859-1。Spring会使用相应的解码算法
	 * Set if context path and request URI should be URL-decoded. Both are returned
	 * <i>undecoded</i> by the Servlet API, in contrast to the servlet path.
	 * <p>Uses either the request encoding or the default encoding according
	 * to the Servlet spec (ISO-8859-1).
	 * @see org.springframework.web.util.UrlPathHelper#setUrlDecode
	 */
	public void setUrlDecode(boolean urlDecode) {
		this.urlPathHelper.setUrlDecode(urlDecode);
		this.corsConfigSource.setUrlDecode(urlDecode);
	}

	/**
	 * Set if ";" (semicolon) content should be stripped from the request URI.
	 * <p>The default value is {@code true}.
	 * @see org.springframework.web.util.UrlPathHelper#setRemoveSemicolonContent(boolean)
	 */
	public void setRemoveSemicolonContent(boolean removeSemicolonContent) {
		this.urlPathHelper.setRemoveSemicolonContent(removeSemicolonContent);
		this.corsConfigSource.setRemoveSemicolonContent(removeSemicolonContent);
	}

	/**
	 * 指定在分析URL时使用的UrlPathHelper。通常使用其默认值。
	 * 使用此方法可以使用自定义子类覆盖默认的UrlPathHelper，或者在多个HandlerMappings和MethodNameResolvers之间共享通用的UrlPathHelper设置。
	 * Set the UrlPathHelper to use for resolution of lookup paths.
	 * <p>Use this to override the default UrlPathHelper with a custom subclass,
	 * or to share common UrlPathHelper settings across multiple HandlerMappings
	 * and MethodNameResolvers.
	 */
	public void setUrlPathHelper(UrlPathHelper urlPathHelper) {
		Assert.notNull(urlPathHelper, "UrlPathHelper must not be null");
		this.urlPathHelper = urlPathHelper;
		this.corsConfigSource.setUrlPathHelper(urlPathHelper);
	}

	/**
	 * Return the UrlPathHelper implementation to use for resolution of lookup paths.
	 */
	public UrlPathHelper getUrlPathHelper() {
		return urlPathHelper;
	}

	/**
	 * 设置PathMatcher实现以用于匹配URL路径与注册的URL模式。默认值为AntPathMatcher。
	 * Set the PathMatcher implementation to use for matching URL paths
	 * against registered URL patterns. Default is AntPathMatcher.
	 * @see org.springframework.util.AntPathMatcher
	 */
	public void setPathMatcher(PathMatcher pathMatcher) {
		Assert.notNull(pathMatcher, "PathMatcher must not be null");
		this.pathMatcher = pathMatcher;
		this.corsConfigSource.setPathMatcher(pathMatcher);
	}

	/**
	 * Return the PathMatcher implementation to use for matching URL paths
	 * against registered URL patterns.
	 */
	public PathMatcher getPathMatcher() {
		return this.pathMatcher;
	}

	/**
	 * 设置在处理器映射中使用的拦截器数组
	 * 支持的拦截器类型是HandlerInterceptor，WebRequestInterceptor和MappedInterceptor。
	 * 映射拦截器仅适用于请求与其路径模式匹配的URL。
	 * 映射的拦截器bean也在初始化期间通过类型检测。
	 * Set the interceptors to apply for all handlers mapped by this handler mapping.
	 * <p>Supported interceptor types are HandlerInterceptor, WebRequestInterceptor, and MappedInterceptor.
	 * Mapped interceptors apply only to request URLs that match its path patterns.
	 * Mapped interceptor beans are also detected by type during initialization.
	 * @param interceptors array of handler interceptors, or {@code null} if none
	 * @see #adaptInterceptor
	 * @see org.springframework.web.servlet.HandlerInterceptor
	 * @see org.springframework.web.context.request.WebRequestInterceptor
	 */
	public void setInterceptors(Object... interceptors) {
		this.interceptors.addAll(Arrays.asList(interceptors));
	}

	/**
	 * 根据URL模式设置“全局”CORS（CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing））配置。默认情况下，第一个匹配的URL模式与处理程序的CORS配置（如果有）相结合。
	 * Configure a custom {@link CorsProcessor} to use to apply the matched
	 * {@link CorsConfiguration} for a request.
	 * <p>By default {@link DefaultCorsProcessor} is used.
	 * @since 4.2
	 */
	public void setCorsProcessor(CorsProcessor corsProcessor) {
		Assert.notNull(corsProcessor, "CorsProcessor must not be null");
		this.corsProcessor = corsProcessor;
	}

	/**
	 * Return the configured {@link CorsProcessor}.
	 */
	public CorsProcessor getCorsProcessor() {
		return this.corsProcessor;
	}

	/**
	 * Set "global" CORS configuration based on URL patterns. By default the first
	 * matching URL pattern is combined with the CORS configuration for the
	 * handler, if any.
	 * @since 4.2
	 */
	public void setCorsConfigurations(Map<String, CorsConfiguration> corsConfigurations) {
		this.corsConfigSource.setCorsConfigurations(corsConfigurations);
	}

	/**
	 * Get the CORS configuration.
	 */
	public Map<String, CorsConfiguration> getCorsConfigurations() {
		return this.corsConfigSource.getCorsConfigurations();
	}


	/**
	 * 初始化拦截器。
	 * Initializes the interceptors.
	 * @see #extendInterceptors(java.util.List)
	 * @see #initInterceptors()
	 */
	@Override
	protected void initApplicationContext() throws BeansException {
		extendInterceptors(this.interceptors);
		detectMappedInterceptors(this.adaptedInterceptors);
		initInterceptors();
	}

	/**
	 * 扩展钩子，子类可以覆盖注册附加的拦截器，给定配置的拦截器（见setInterceptors(java.lang.Object...)）。
   * 在initInterceptors()将指定的拦截器适配为HandlerInterceptor实例之前，将被调用。
   * 供子类扩展添加拦截器，目前spring没有自行实现。默认实现为空。
   *
	 * Extension hook that subclasses can override to register additional interceptors,
	 * given the configured interceptors (see {@link #setInterceptors}).
	 * <p>Will be invoked before {@link #initInterceptors()} adapts the specified
	 * interceptors into {@link HandlerInterceptor} instances.
	 * <p>The default implementation is empty.
	 * @param interceptors the configured interceptor List (never {@code null}), allowing
	 * to add further interceptors before as well as after the existing interceptors
	 */
	protected void extendInterceptors(List<Object> interceptors) {
	}

	/**
	 * 将springMVC容器或者父容器中的所有MappedInterceptor类型的Bean添加到mappedInterceptors属性中
	 * 检测bean的类型，MappedInterceptor并将它们添加到映射拦截器的列表中。
	 * 除了通过默认情况下， 从当前上下文及其祖先添加所有类型的bean 之外MappedInterceptor，还可以调用这个。子类可以覆盖并优化此策略
	 * Detect beans of type {@link MappedInterceptor} and add them to the list of mapped interceptors.
	 * <p>This is called in addition to any {@link MappedInterceptor}s that may have been provided
	 * via {@link #setInterceptors}, by default adding all beans of type {@link MappedInterceptor}
	 * from the current context and its ancestors. Subclasses can override and refine this policy.
	 * @param mappedInterceptors an empty list to add {@link MappedInterceptor} instances to
	 */
	protected void detectMappedInterceptors(List<HandlerInterceptor> mappedInterceptors) {
		mappedInterceptors.addAll(
				BeanFactoryUtils.beansOfTypeIncludingAncestors(
						getApplicationContext(), MappedInterceptor.class, true, false).values());
	}

	/**
	 * 将interceptors集合添加至adaptedInterceptors集合中
	 * 
	 * Initialize the specified interceptors, checking for {@link MappedInterceptor}s and
	 * adapting {@link HandlerInterceptor}s and {@link WebRequestInterceptor}s if necessary.
	 * @see #setInterceptors
	 * @see #adaptInterceptor
	 */
	protected void initInterceptors() {
		if (!this.interceptors.isEmpty()) {
			for (int i = 0; i < this.interceptors.size(); i++) {
				Object interceptor = this.interceptors.get(i);
				if (interceptor == null) {
					throw new IllegalArgumentException("Entry number " + i + " in interceptors array is null");
				}
				this.adaptedInterceptors.add(adaptInterceptor(interceptor));
			}
		}
	}

	/**
	 * 将给定的拦截器对象适配到HandlerInterceptor接口。
	 * Adapt the given interceptor object to the {@link HandlerInterceptor} interface.
	 * 默认情况下，受支持的拦截器类型为{@link HandlerInterceptor}和{@link WebRequestInterceptor}。
	 * 每个给定的{@link WebRequestInterceptor}将包装在{@link WebRequestHandlerInterceptorAdapter}中。
   * 可以在子类中覆盖。
	 * <p>By default, the supported interceptor types are {@link HandlerInterceptor}
	 * and {@link WebRequestInterceptor}. Each given {@link WebRequestInterceptor}
	 * will be wrapped in a {@link WebRequestHandlerInterceptorAdapter}.
	 * Can be overridden in subclasses.
	 * @param interceptor the specified interceptor object
	 * @return the interceptor wrapped as HandlerInterceptor
	 * @see org.springframework.web.servlet.HandlerInterceptor
	 * @see org.springframework.web.context.request.WebRequestInterceptor
	 * @see WebRequestHandlerInterceptorAdapter
	 */
	protected HandlerInterceptor adaptInterceptor(Object interceptor) {
		if (interceptor instanceof HandlerInterceptor) {
			return (HandlerInterceptor) interceptor;
		}
		else if (interceptor instanceof WebRequestInterceptor) {
			return new WebRequestHandlerInterceptorAdapter((WebRequestInterceptor) interceptor);
		}
		else {
			throw new IllegalArgumentException("Interceptor type not supported: " + interceptor.getClass().getName());
		}
	}

	/**
	 * 将适配的拦截器作为HandlerInterceptor数组返回。
	 *
	 * Return the adapted interceptors as {@link HandlerInterceptor} array.
	 * @return the array of {@link HandlerInterceptor}s, or {@code null} if none
	 */
	protected final HandlerInterceptor[] getAdaptedInterceptors() {
		int count = this.adaptedInterceptors.size();
		return (count > 0 ? this.adaptedInterceptors.toArray(new HandlerInterceptor[count]) : null);
	}

	/**
	 * 将所有配置的MappedInterceptors作为数组返回。
	 * Return all configured {@link MappedInterceptor}s as an array.
	 * @return the array of {@link MappedInterceptor}s, or {@code null} if none
	 */
	protected final MappedInterceptor[] getMappedInterceptors() {
		List<MappedInterceptor> mappedInterceptors = new ArrayList<MappedInterceptor>();
		for (HandlerInterceptor interceptor : this.adaptedInterceptors) {
			if (interceptor instanceof MappedInterceptor) {
				mappedInterceptors.add((MappedInterceptor) interceptor);
			}
		}
		int count = mappedInterceptors.size();
		return (count > 0 ? mappedInterceptors.toArray(new MappedInterceptor[count]) : null);
	}


	/**
	 * 查找给定请求的处理程序，如果没有找到，则返回到默认处理程序。
	 * Look up a handler for the given request, falling back to the default
	 * handler if no specific one is found.
	 * @param request current HTTP request
	 * @return the corresponding handler instance, or the default handler
	 * @see #getHandlerInternal
	 */
	@Override
	public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		//为抽象方法，供子类实现
	  Object handler = getHandlerInternal(request);
		if (handler == null) {
			handler = getDefaultHandler();
		}
		if (handler == null) {
			return null;
		}
		// Bean name or resolved handler?
		if (handler instanceof String) {
			String handlerName = (String) handler;
			handler = getApplicationContext().getBean(handlerName);
		}

		HandlerExecutionChain executionChain = getHandlerExecutionChain(handler, request);
		//cros跨域请求的处理
		if (CorsUtils.isCorsRequest(request)) {
			CorsConfiguration globalConfig = this.corsConfigSource.getCorsConfiguration(request);
			CorsConfiguration handlerConfig = getCorsConfiguration(handler, request);
			CorsConfiguration config = (globalConfig != null ? globalConfig.combine(handlerConfig) : handlerConfig);
			executionChain = getCorsHandlerExecutionChain(request, executionChain, config);
		}
		return executionChain;
	}

	/**
	 * 查找给定请求的处理程序，null如果没有找到特定请求，则返回。此方法由调用getHandler(javax.servlet.http.HttpServletRequest);
	  一个null返回值将导致默认的处理程序，如果已设置。
    在CORS pre-flight请求之前，此方法应返回匹配不适用于pre-flight前请求的匹配项，但是返回基于URL路径的预期实际请求，
    “Access-Control-Request-Method”头部的HTTP方法和头从“访问控制 - 请求 - 头部”头，从而允许通过CORS配置获得CORS配置getCorsConfigurations()，
    注意：此方法还可以返回预先构建的HandlerExecutionChain，将处理程序对象与动态确定的拦截器组合在一起。静态指定的拦截器将被合并到这样的现有链中。
	 * Look up a handler for the given request, returning {@code null} if no
	 * specific one is found. This method is called by {@link #getHandler};
	 * a {@code null} return value will lead to the default handler, if one is set.
	 * <p>On CORS pre-flight requests this method should return a match not for
	 * the pre-flight request but for the expected actual request based on the URL
	 * path, the HTTP methods from the "Access-Control-Request-Method" header, and
	 * the headers from the "Access-Control-Request-Headers" header thus allowing
	 * the CORS configuration to be obtained via {@link #getCorsConfigurations},
	 * <p>Note: This method may also return a pre-built {@link HandlerExecutionChain},
	 * combining a handler object with dynamically determined interceptors.
	 * Statically specified interceptors will get merged into such an existing chain.
	 * @param request current HTTP request
	 * @return the corresponding handler instance, or {@code null} if none found
	 * @throws Exception if there is an internal error
	 */
	protected abstract Object getHandlerInternal(HttpServletRequest request) throws Exception;

	/**
	
	 * 为给定的处理程序构建一个包含适当的拦截器HandlerExecutionChain。
	 * Build a {@link HandlerExecutionChain} for the given handler, including
	 * applicable interceptors.
	 * <p>The default implementation builds a standard {@link HandlerExecutionChain}
	 * with the given handler, the handler mapping's common interceptors, and any
	 * {@link MappedInterceptor}s matching to the current request URL. Interceptors
	 * are added in the order they were registered. Subclasses may override this
	 * in order to extend/rearrange the list of interceptors.
	 * <p><b>NOTE:</b> The passed-in handler object may be a raw handler or a
	 * pre-built {@link HandlerExecutionChain}. This method should handle those
	 * two cases explicitly, either building a new {@link HandlerExecutionChain}
	 * or extending the existing chain.
	 * <p>For simply adding an interceptor in a custom subclass, consider calling
	 * {@code super.getHandlerExecutionChain(handler, request)} and invoking
	 * {@link HandlerExecutionChain#addInterceptor} on the returned chain object.
	 * @param handler the resolved handler instance (never {@code null})
	 * @param request current HTTP request
	 * @return the HandlerExecutionChain (never {@code null})
	 * @see #getAdaptedInterceptors()
	 */
	protected HandlerExecutionChain getHandlerExecutionChain(Object handler, HttpServletRequest request) {
		HandlerExecutionChain chain = (handler instanceof HandlerExecutionChain ?
				(HandlerExecutionChain) handler : new HandlerExecutionChain(handler));

		String lookupPath = this.urlPathHelper.getLookupPathForRequest(request);
		for (HandlerInterceptor interceptor : this.adaptedInterceptors) {
			if (interceptor instanceof MappedInterceptor) {
				MappedInterceptor mappedInterceptor = (MappedInterceptor) interceptor;
				if (mappedInterceptor.matches(lookupPath, this.pathMatcher)) {
					chain.addInterceptor(mappedInterceptor.getInterceptor());
				}
			}
			else {
				chain.addInterceptor(interceptor);
			}
		}
		return chain;
	}

	/**
	 * 检索给定处理程序的CORS配置。
	 * Retrieve the CORS configuration for the given handler.
	 * @param handler the handler to check (never {@code null}).
	 * @param request the current request.
	 * @return the CORS configuration for the handler or {@code null}.
	 * @since 4.2
	 */
	protected CorsConfiguration getCorsConfiguration(Object handler, HttpServletRequest request) {
		if (handler instanceof HandlerExecutionChain) {
			handler = ((HandlerExecutionChain) handler).getHandler();
		}
		if (handler instanceof CorsConfigurationSource) {
			return ((CorsConfigurationSource) handler).getCorsConfiguration(request);
		}
		return null;
	}

	/**
	 * 更新HandlerExecutionChain以进行与CORS相关的处理。
	 * Update the HandlerExecutionChain for CORS-related handling.
	 * <p>For pre-flight requests, the default implementation replaces the selected
	 * handler with a simple HttpRequestHandler that invokes the configured
	 * {@link #setCorsProcessor}.
	 * <p>For actual requests, the default implementation inserts a
	 * HandlerInterceptor that makes CORS-related checks and adds CORS headers.
	 * @param request the current request
	 * @param chain the handler chain
	 * @param config the applicable CORS configuration, possibly {@code null}
	 * @since 4.2
	 */
	protected HandlerExecutionChain getCorsHandlerExecutionChain(HttpServletRequest request,
			HandlerExecutionChain chain, CorsConfiguration config) {

		if (CorsUtils.isPreFlightRequest(request)) {
			HandlerInterceptor[] interceptors = chain.getInterceptors();
			chain = new HandlerExecutionChain(new PreFlightHandler(config), interceptors);
		}
		else {
			chain.addInterceptor(new CorsInterceptor(config));
		}
		return chain;
	}


	private class PreFlightHandler implements HttpRequestHandler {

		private final CorsConfiguration config;

		public PreFlightHandler(CorsConfiguration config) {
			this.config = config;
		}

		@Override
		public void handleRequest(HttpServletRequest request, HttpServletResponse response) throws IOException {
			corsProcessor.processRequest(this.config, request, response);
		}
	}


	private class CorsInterceptor extends HandlerInterceptorAdapter {

		private final CorsConfiguration config;

		public CorsInterceptor(CorsConfiguration config) {
			this.config = config;
		}

		@Override
		public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
				throws Exception {

			return corsProcessor.processRequest(this.config, request, response);
		}
	}

}
```

###### 抽象类AbstractUrlHandlerMapping继承抽象类AbstractHandlerMapping
URL映射HandlerMapping 实现的抽象基类。提供用于将处理程序映射到URL和可配置URL查找的基础结构。
支持直接匹配，例如注册的“/ test”匹配“/ test”和各种Ant样式模式匹配，例如注册的“/ t *”模式匹配“/ test”和“/ team”，
“/ test / *“匹配”/ test“目录中的所有路径，”/ test / **“匹配”/ test“下的所有路径。有关详细信息，请参阅 AntPathMatcherjavadoc。
将搜索所有路径模式以查找当前请求路径的最精确匹配。最精确的匹配被定义为与当前请求路径匹配的最长路径模式。

```jav
/**
 * Abstract base class for URL-mapped {@link org.springframework.web.servlet.HandlerMapping}
 * implementations. Provides infrastructure for mapping handlers to URLs and configurable
 * URL lookup. For information on the latter, see "alwaysUseFullPath" property.
 *
 * <p>Supports direct matches, e.g. a registered "/test" matches "/test", and
 * various Ant-style pattern matches, e.g. a registered "/t*" pattern matches
 * both "/test" and "/team", "/test/*" matches all paths in the "/test" directory,
 * "/test/**" matches all paths below "/test". For details, see the
 * {@link org.springframework.util.AntPathMatcher AntPathMatcher} javadoc.
 *
 * <p>Will search all path patterns to find the most exact match for the
 * current request path. The most exact match is defined as the longest
 * path pattern that matches the current request path.
 */
public abstract class AbstractUrlHandlerMapping extends AbstractHandlerMapping {

	private Object rootHandler;

	private boolean useTrailingSlashMatch = false;

  //这个属性允许你设置是否延迟singleton处理器的初始化工作（prototype处理器的初始化都是延迟的)。 这个属性的默认值是false.
	private boolean lazyInitHandlers = false;

	private final Map<String, Object> handlerMap = new LinkedHashMap<String, Object>();


	/**
	 * 设置此处理器映射的根处理器，即要为根路径（“/”）注册的处理器。
   * 默认是null，指示没有根处理器
	 * Set the root handler for this handler mapping, that is,
	 * the handler to be registered for the root path ("/").
	 * <p>Default is {@code null}, indicating no root handler.
	 */
	public void setRootHandler(Object rootHandler) {
		this.rootHandler = rootHandler;
	}

	/**
	 * Return the root handler for this handler mapping (registered for "/"),
	 * or {@code null} if none.
	 */
	public Object getRootHandler() {
		return this.rootHandler;
	}

	/**
	 * 是否匹配URL，而不管尾部斜线是否存在。如果启用了诸如“/ users”的URL模式也与“/ users /”匹配。
	 * 默认是false
	 * Whether to match to URLs irrespective of the presence of a trailing slash.
	 * If enabled a URL pattern such as "/users" also matches to "/users/".
	 * <p>The default value is {@code false}.
	 */
	public void setUseTrailingSlashMatch(boolean useTrailingSlashMatch) {
		this.useTrailingSlashMatch = useTrailingSlashMatch;
	}

	/**
	 * 是否匹配URL，而不管尾部斜线是否存在。
	 * Whether to match to URLs irrespective of the presence of a trailing slash.
	 */
	public boolean useTrailingSlashMatch() {
		return this.useTrailingSlashMatch;
	}

	/**
	 * 设置是否懒惰地初始化处理程序。只适用于单例处理程序，因为原型总是懒惰地初始化。默认值为“false”，
	 * 因为初始化可以通过直接引用控制器对象来提高效率。
	 * 如果要允许您的控制器进行懒惰初始化，请将其设置为“lazy-init”，并将此标志设置为true。只要使它们“lazy-init”不起作用，
	 * 因为在这种情况下，通过处理程序映射的引用来初始化它们。
	 * Set whether to lazily initialize handlers. Only applicable to
	 * singleton handlers, as prototypes are always lazily initialized.
	 * Default is "false", as eager initialization allows for more efficiency
	 * through referencing the controller objects directly.
	 * <p>If you want to allow your controllers to be lazily initialized,
	 * make them "lazy-init" and set this flag to true. Just making them
	 * "lazy-init" will not work, as they are initialized through the
	 * references from the handler mapping in this case.
	 */
	public void setLazyInitHandlers(boolean lazyInitHandlers) {
		this.lazyInitHandlers = lazyInitHandlers;
	}

	/**
	 * 查找给定请求的URL路径的处理器，实现了父类的该抽象方法。
	 * Look up a handler for the URL path of the given request.
	 * @param request current HTTP request
	 * @return the handler instance, or {@code null} if none found
	 */
	@Override
	protected Object getHandlerInternal(HttpServletRequest request) throws Exception {
		String lookupPath = getUrlPathHelper().getLookupPathForRequest(request);
		Object handler = lookupHandler(lookupPath, request);
		if (handler == null) {
			// We need to care for the default handler directly, since we need to
			// expose the PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE for it as well.
			Object rawHandler = null;
			if ("/".equals(lookupPath)) {
				rawHandler = getRootHandler();
			}
			if (rawHandler == null) {
				rawHandler = getDefaultHandler();
			}
			if (rawHandler != null) {
				// Bean name or resolved handler?
				if (rawHandler instanceof String) {
					String handlerName = (String) rawHandler;
					rawHandler = getApplicationContext().getBean(handlerName);
				}
				validateHandler(rawHandler, request);
				handler = buildPathExposingHandler(rawHandler, lookupPath, lookupPath, null);
			}
		}
		if (handler != null && logger.isDebugEnabled()) {
			logger.debug("Mapping [" + lookupPath + "] to " + handler);
		}
		else if (handler == null && logger.isTraceEnabled()) {
			logger.trace("No handler mapping found for [" + lookupPath + "]");
		}
		return handler;
	}

	/**
	 * 查找给定URL路径的处理器实例。
	 * Look up a handler instance for the given URL path.
	 * <p>Supports direct matches, e.g. a registered "/test" matches "/test",
	 * and various Ant-style pattern matches, e.g. a registered "/t*" matches
	 * both "/test" and "/team". For details, see the AntPathMatcher class.
	 * <p>Looks for the most exact pattern, where most exact is defined as
	 * the longest path pattern.
	 * @param urlPath URL the bean is mapped to
	 * @param request current HTTP request (to expose the path within the mapping to)
	 * @return the associated handler instance, or {@code null} if not found
	 * @see #exposePathWithinMapping
	 * @see org.springframework.util.AntPathMatcher
	 */
	protected Object lookupHandler(String urlPath, HttpServletRequest request) throws Exception {
		// Direct match?
		Object handler = this.handlerMap.get(urlPath);
		if (handler != null) {
			// Bean name or resolved handler?
			if (handler instanceof String) {
				String handlerName = (String) handler;
				handler = getApplicationContext().getBean(handlerName);
			}
			validateHandler(handler, request);
			return buildPathExposingHandler(handler, urlPath, urlPath, null);
		}
		// Pattern match?
		List<String> matchingPatterns = new ArrayList<String>();
		for (String registeredPattern : this.handlerMap.keySet()) {
			if (getPathMatcher().match(registeredPattern, urlPath)) {
				matchingPatterns.add(registeredPattern);
			}
			else if (useTrailingSlashMatch()) {
				if (!registeredPattern.endsWith("/") && getPathMatcher().match(registeredPattern + "/", urlPath)) {
					matchingPatterns.add(registeredPattern +"/");
				}
			}
		}
		String bestPatternMatch = null;
		Comparator<String> patternComparator = getPathMatcher().getPatternComparator(urlPath);
		if (!matchingPatterns.isEmpty()) {
			Collections.sort(matchingPatterns, patternComparator);
			if (logger.isDebugEnabled()) {
				logger.debug("Matching patterns for request [" + urlPath + "] are " + matchingPatterns);
			}
			bestPatternMatch = matchingPatterns.get(0);
		}
		if (bestPatternMatch != null) {
			handler = this.handlerMap.get(bestPatternMatch);
			if (handler == null) {
				Assert.isTrue(bestPatternMatch.endsWith("/"));
				handler = this.handlerMap.get(bestPatternMatch.substring(0, bestPatternMatch.length() - 1));
			}
			// Bean name or resolved handler?
			if (handler instanceof String) {
				String handlerName = (String) handler;
				handler = getApplicationContext().getBean(handlerName);
			}
			validateHandler(handler, request);
			String pathWithinMapping = getPathMatcher().extractPathWithinPattern(bestPatternMatch, urlPath);

			// There might be multiple 'best patterns', let's make sure we have the correct URI template variables
			// for all of them
			Map<String, String> uriTemplateVariables = new LinkedHashMap<String, String>();
			for (String matchingPattern : matchingPatterns) {
				if (patternComparator.compare(bestPatternMatch, matchingPattern) == 0) {
					Map<String, String> vars = getPathMatcher().extractUriTemplateVariables(matchingPattern, urlPath);
					Map<String, String> decodedVars = getUrlPathHelper().decodePathVariables(request, vars);
					uriTemplateVariables.putAll(decodedVars);
				}
			}
			if (logger.isDebugEnabled()) {
				logger.debug("URI Template variables for request [" + urlPath + "] are " + uriTemplateVariables);
			}
			return buildPathExposingHandler(handler, bestPatternMatch, pathWithinMapping, uriTemplateVariables);
		}
		// No handler found...
		return null;
	}

	/**
	 * 根据当前请求验证给定的处理程序。
   * 默认实现为空。可以在子类中被覆盖，例如强制以URL映射表示的特定前提条件。
	 * Validate the given handler against the current request.
	 * <p>The default implementation is empty. Can be overridden in subclasses,
	 * for example to enforce specific preconditions expressed in URL mappings.
	 * @param handler the handler object to validate
	 * @param request current HTTP request
	 * @throws Exception if validation failed
	 */
	protected void validateHandler(Object handler, HttpServletRequest request) throws Exception {
	}

	/**
	 * Build a handler object for the given raw handler, exposing the actual
	 * handler, the {@link #PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE}, as well as
	 * the {@link #URI_TEMPLATE_VARIABLES_ATTRIBUTE} before executing the handler.
	 * <p>The default implementation builds a {@link HandlerExecutionChain}
	 * with a special interceptor that exposes the path attribute and uri template variables
	 * @param rawHandler the raw handler to expose
	 * @param pathWithinMapping the path to expose before executing the handler
	 * @param uriTemplateVariables the URI template variables, can be {@code null} if no variables found
	 * @return the final handler object
	 */
	protected Object buildPathExposingHandler(Object rawHandler, String bestMatchingPattern,
			String pathWithinMapping, Map<String, String> uriTemplateVariables) {

		HandlerExecutionChain chain = new HandlerExecutionChain(rawHandler);
		chain.addInterceptor(new PathExposingHandlerInterceptor(bestMatchingPattern, pathWithinMapping));
		if (!CollectionUtils.isEmpty(uriTemplateVariables)) {
			chain.addInterceptor(new UriTemplateVariablesHandlerInterceptor(uriTemplateVariables));
		}
		return chain;
	}

	/**
	 * Expose the path within the current mapping as request attribute.
	 * @param pathWithinMapping the path within the current mapping
	 * @param request the request to expose the path to
	 * @see #PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE
	 */
	protected void exposePathWithinMapping(String bestMatchingPattern, String pathWithinMapping, HttpServletRequest request) {
		request.setAttribute(HandlerMapping.BEST_MATCHING_PATTERN_ATTRIBUTE, bestMatchingPattern);
		request.setAttribute(HandlerMapping.PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE, pathWithinMapping);
	}

	/**
	 * Expose the URI templates variables as request attribute.
	 * @param uriTemplateVariables the URI template variables
	 * @param request the request to expose the path to
	 * @see #PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE
	 */
	protected void exposeUriTemplateVariables(Map<String, String> uriTemplateVariables, HttpServletRequest request) {
		request.setAttribute(HandlerMapping.URI_TEMPLATE_VARIABLES_ATTRIBUTE, uriTemplateVariables);
	}

	/**
	 * 为指定的URL路径注册指定的处理器
	 * Register the specified handler for the given URL paths.
	 * @param urlPaths the URLs that the bean should be mapped to
	 * @param beanName the name of the handler bean
	 * @throws BeansException if the handler couldn't be registered
	 * @throws IllegalStateException if there is a conflicting handler registered
	 */
	protected void registerHandler(String[] urlPaths, String beanName) throws BeansException, IllegalStateException {
		Assert.notNull(urlPaths, "URL path array must not be null");
		for (String urlPath : urlPaths) {
			registerHandler(urlPath, beanName);
		}
	}

	/**
	 * Register the specified handler for the given URL path.
	 * @param urlPath the URL the bean should be mapped to
	 * @param handler the handler instance or handler bean name String
	 * (a bean name will automatically be resolved into the corresponding handler bean)
	 * @throws BeansException if the handler couldn't be registered
	 * @throws IllegalStateException if there is a conflicting handler registered
	 */
	protected void registerHandler(String urlPath, Object handler) throws BeansException, IllegalStateException {
		Assert.notNull(urlPath, "URL path must not be null");
		Assert.notNull(handler, "Handler object must not be null");
		Object resolvedHandler = handler;

		// Eagerly resolve handler if referencing singleton via name.
		if (!this.lazyInitHandlers && handler instanceof String) {
			String handlerName = (String) handler;
			if (getApplicationContext().isSingleton(handlerName)) {
				resolvedHandler = getApplicationContext().getBean(handlerName);
			}
		}

		Object mappedHandler = this.handlerMap.get(urlPath);
		if (mappedHandler != null) {
			if (mappedHandler != resolvedHandler) {
				throw new IllegalStateException(
						"Cannot map " + getHandlerDescription(handler) + " to URL path [" + urlPath +
						"]: There is already " + getHandlerDescription(mappedHandler) + " mapped.");
			}
		}
		else {
			if (urlPath.equals("/")) {
				if (logger.isInfoEnabled()) {
					logger.info("Root mapping to " + getHandlerDescription(handler));
				}
				setRootHandler(resolvedHandler);
			}
			else if (urlPath.equals("/*")) {
				if (logger.isInfoEnabled()) {
					logger.info("Default mapping to " + getHandlerDescription(handler));
				}
				setDefaultHandler(resolvedHandler);
			}
			else {
				this.handlerMap.put(urlPath, resolvedHandler);
				if (logger.isInfoEnabled()) {
					logger.info("Mapped URL path [" + urlPath + "] onto " + getHandlerDescription(handler));
				}
			}
		}
	}

	private String getHandlerDescription(Object handler) {
		return "handler " + (handler instanceof String ? "'" + handler + "'" : "of type [" + handler.getClass() + "]");
	}


	/**
	 * 将注册的处理程序作为不可修改的Map返回，注册的路径作为关键字，处理程序对象
	 *（或在lazy-init处理程序的情况下为处理程序bean名称）为值。
	 * Return the registered handlers as an unmodifiable Map, with the registered path
	 * as key and the handler object (or handler bean name in case of a lazy-init handler)
	 * as value.
	 * @see #getDefaultHandler()
	 */
	public final Map<String, Object> getHandlerMap() {
		return Collections.unmodifiableMap(this.handlerMap);
	}

	/**
	 * Indicates whether this handler mapping support type-level mappings. Default to {@code false}.
	 */
	protected boolean supportsTypeLevelMappings() {
		return false;
	}


	/**
	 * Special interceptor for exposing the
	 * {@link AbstractUrlHandlerMapping#PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE} attribute.
	 * @see AbstractUrlHandlerMapping#exposePathWithinMapping
	 */
	private class PathExposingHandlerInterceptor extends HandlerInterceptorAdapter {

		private final String bestMatchingPattern;

		private final String pathWithinMapping;

		public PathExposingHandlerInterceptor(String bestMatchingPattern, String pathWithinMapping) {
			this.bestMatchingPattern = bestMatchingPattern;
			this.pathWithinMapping = pathWithinMapping;
		}

		@Override
		public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
			exposePathWithinMapping(this.bestMatchingPattern, this.pathWithinMapping, request);
			request.setAttribute(HandlerMapping.INTROSPECT_TYPE_LEVEL_MAPPING, supportsTypeLevelMappings());
			return true;
		}

	}

	/**
	 * Special interceptor for exposing the
	 * {@link AbstractUrlHandlerMapping#URI_TEMPLATE_VARIABLES_ATTRIBUTE} attribute.
	 * @see AbstractUrlHandlerMapping#exposePathWithinMapping
	 */
	private class UriTemplateVariablesHandlerInterceptor extends HandlerInterceptorAdapter {

		private final Map<String, String> uriTemplateVariables;

		public UriTemplateVariablesHandlerInterceptor(Map<String, String> uriTemplateVariables) {
			this.uriTemplateVariables = uriTemplateVariables;
		}

		@Override
		public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
			exposeUriTemplateVariables(this.uriTemplateVariables, request);
			return true;
		}
	}

}

```



##### MVC里最常见的三种处理器射映
HandlerMapping的基本功能是将请求传递到HandlerExecutionChain上。首先，这个HandlerExecutionChain必须包含一个能处理该请求的处理器。
其次，这个链也可以包含一系列可以拦截请求的拦截器。当收到请求时，DispatcherServlet将请求交给处理器映射，让它检查请求并找到一个适当的HandlerExecutionChain。然后，DispatcherServlet执行定义在链中的处理器和拦截器(interceptor)。

* SimpleUrlHandlerMapping

它在应用上下文中可以进行配置，并且有Ant风格的路径匹配功能。(请参考org.springframework.util.PathMatcher的JavaDoc)。
```java
<beans>
    <!-- no 'id' required, HandlerMapping beans are automatically detected by the DispatcherServlet -->
    <bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
        <property name="mappings">
           <props>
             <prop key="/*/account.form">editAccountFormController</prop>
             <prop key="/*/editaccount.form">editAccountFormController</prop>
             <prop key="/ex/view*.html">helpController</prop>
             <prop key="/**/help.html">helpController</prop>
           </props>
        </property>
    </bean>

    <bean id="helpController"
          class="org.springframework.web.servlet.mvc.UrlFilenameViewController"/>

    <bean id="editAccountFormController"
          class="org.springframework.web.servlet.mvc.SimpleFormController">
        <property name="formView" value="account"/>
        <property name="successView" value="account-created"/>
        <property name="commandName" value="Account"/>
        <property name="commandClass" value="samples.Account"/>
    </bean>
<beans>

```
这个处理器映射首先将对所有目录中文件名为help.html的请求传递给helpController。 helpController是一个UrlFilenameViewController 。对ex目录中所有以view开始，以.html 结尾的请求都会被传递给helpController。
同样的，我们也为editAccountFormController定义了两个映射。

