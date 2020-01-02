---
layout: post
title:  "Spring Security"
date:   2019-10-23 14:34:23
author: zhangtejun
categories: Java
---
#####
Spring security 是一个强大的和高度可定制的身份验证和访问控制框架。Spring Security 为基于javaEE的企业应用程序提供一个全面的解决方案


用户认证：实现认证需要我们有一套用户数据源，而授权是某个用户具有的相应权限，在Spring Security可以通过重写
`protected void configure(AuthenticationManagerBuilder auth)`方法来实现定制。

用户数据源：
1. 内存中的用户
```
@Configuration
@EnableWebSecurity
public class WebSecurityConfig  extends WebSecurityConfigurerAdapter{
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 1. 创建2个用户数据源
        auth.inMemoryAuthentication()
                .withUser("test").password("123456").roles("ADMIN")
                .and()
                .withUser("test1").password("123321").roles("USER");
    }
}

@Bean
public UserDetailsService userDetailsService() {
    InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
    manager.createUser(User.withUsername("user").password("password").roles("USER").build());
    return manager;
}

```

请求授权：在Spring Security可以通过重写 `protected void configure(HttpSecurity http)`来拦截。
Spring Security使用匹配器类型：
* antMatchers: 使用ant风格的路径匹配
* regexMatchers: 使用正则表达式匹配路径

anyRequest：匹配所有请求路径

```
    /**
     * 请求授权
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests() //通过该方法来开始请求权限配置
                .antMatchers("/admin/**").hasRole("ROLE_ADMIN")// 只有ROLE_ADMIN角色的用户可以访问
                .antMatchers("/user/**").hasAnyRole("ROLE_ADMIN","ROLE_USER")// 有ROLE_ADMIN或者ROLE_USER角色都可以访问
                .anyRequest().authenticated();// 其余请求都需要认证后才可以访问
    }
```

Authentication即鉴权、通俗的说就是用户登录操作，在我们一般的设计中，登录提交和登录成功获取的都是用户记录信息；
整个鉴权服务只提供一个验证登录的接口。

1. 登录时提交的用户名和密码封装在了哪个类中？
2. 哪个接口负责接收登录提交的用户名和密码，返回的又是什么类，封装哪些信息？
3. 鉴权服务是如何判断用户名和密码是否正确？又是如何获取已有的用户信息的？

Spring Security中设计了一个Authentication类，我们输入的用户名和密码被当做用户的主体标识和用户的鉴权凭证进行封装，

```java
public interface Authentication extends Principal, Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();
    // Credentials字段则存储密码作为用户端鉴权凭证
    Object getCredentials();
    // 认证时包含的其他信息
    Object getDetails();
    // Principal 即用户名，用于唯一标识用户信息的属性
    Object getPrincipal();
    // 是否被认证，认证为true
    boolean isAuthenticated();
    // 设置是否能被认证
    void setAuthenticated(boolean var1) throws IllegalArgumentException;
}
```

如果我们不使用`密码`而是使用其他诸如`令牌`之类的也可以将其视为是的鉴权凭证。 不同的验证协议可能存在不同的请求参数格式，
Spring Security中也提供了常见的几种常见的封装，比如我们之前说到的使用`用户名密码`和使用`RememberMe`的令牌形式。

除此以外，Authentication不仅封装用于登录的属性，在完成鉴权操作后也会将它关联的权限列表全部存储在Authorities中。
整个鉴权的最初和最终状态就好比我们去食堂打饭拉卡，把饭盒和饭卡都递给打饭阿姨，只要饭卡里有钱，阿姨就会把对应的饭盒给你装满。
你递出去是饭盒和饭卡，拿回来的还是饭盒和饭卡。无论将来饭盒是两层还是三层，饭卡是实体卡还是支付宝都和打饭的过程没有了关系。

**AuthenticationManager**
该对象提供了认证方法的入口，接收一个`Authentication`对象作为参数。
鉴权服务的形式`	Authentication authenticate(Authentication authentication)`，提供最终鉴权服务的是AuthenticationProvider。
```java
public interface AuthenticationProvider {
    Authentication authenticate(Authentication var1) throws AuthenticationException;
    boolean supports(Class<?> var1);
}
```

**ProviderManager**
它是 AuthenticationManager 的一个实现类，提供了基本的认证逻辑和方法，ProviderManager中配置了各种鉴权服务了除了验证用户信息服务之外的配置。
AuthenticationManager是唯一向外的提供的用户验证服务接口。AuthenticationManager本身并不实现验证用户身份的逻辑而是委托交由其配置的AuthenticationProvider去完成。
它包含了一个` List<AuthenticationProvider>` 对象，通过 AuthenticationProvider 接口来扩展出不同的认证提供者。

```
public Authentication authenticate(Authentication authentication)
			throws AuthenticationException {
		// 获取当前的Authentication的认证类型
		Class<? extends Authentication> toTest = authentication.getClass();
		AuthenticationException lastException = null;
		AuthenticationException parentException = null;
		Authentication result = null;
		Authentication parentResult = null;
		boolean debug = logger.isDebugEnabled();
    // 遍历所有的providers使用supports方法判断该provider是否支持当前的认证类型
		for (AuthenticationProvider provider : getProviders()) {
			if (!provider.supports(toTest)) {
				continue;
			}
			if (debug) {
				logger.debug("Authentication attempt using " + provider.getClass().getName());
			}

			try {
			  // 调用provider的authenticat方法认证
				result = provider.authenticate(authentication);
				if (result != null) {
					copyDetails(authentication, result);
					break;
				}
			}
			catch (AccountStatusException e) {
				prepareException(e, authentication);
				// SEC-546: Avoid polling additional providers if auth failure is due to
				// invalid account status
				throw e;
			}
			catch (InternalAuthenticationServiceException e) {
				prepareException(e, authentication);
				throw e;
			}
			catch (AuthenticationException e) {
				lastException = e;
			}
		}

		if (result == null && parent != null) {
			// Allow the parent to try.
			try {
			  // 没有验证通过，则使用父类型AuthenticationManager进行验证
				result = parentResult = parent.authenticate(authentication);
			}
			catch (ProviderNotFoundException e) {
				// ignore as we will throw below if no other exception occurred prior to
				// calling parent and the parent
				// may throw ProviderNotFound even though a provider in the child already
				// handled the request
			}
			catch (AuthenticationException e) {
				lastException = parentException = e;
			}
		}
    // 移除敏感信息
		if (result != null) {
			if (eraseCredentialsAfterAuthentication
					&& (result instanceof CredentialsContainer)) {
				// Authentication is complete. Remove credentials and other secret data
				// from authentication
				((CredentialsContainer) result).eraseCredentials();
			}

			// If the parent AuthenticationManager was attempted and successful than it will publish an AuthenticationSuccessEvent
			// This check prevents a duplicate AuthenticationSuccessEvent if the parent AuthenticationManager already published it
			if (parentResult == null) {
				eventPublisher.publishAuthenticationSuccess(result);
			}
			return result;
		}
		if (lastException == null) {
			lastException = new ProviderNotFoundException(messages.getMessage(
					"ProviderManager.providerNotFound",
					new Object[] { toTest.getName() },
					"No AuthenticationProvider found for {0}"));
		}
		// If the parent AuthenticationManager was attempted and failed than it will publish an AbstractAuthenticationFailureEvent
		// This check prevents a duplicate AbstractAuthenticationFailureEvent if the parent AuthenticationManager already published it
		if (parentException == null) {
			prepareException(lastException, authentication);
		}
		throw lastException;
	}
```

**AuthenticationProvider**

我们使用AuthenticationProvider来为我们提供多种Provider服务,即扩展出更多的验证提供的方式。在初始化ProviderManager的时候，我们需要设置Manager所管理的所有
AuthenticationProvider的列表，然后一样调用authenticate验证方法也可以达到同样的效果。
```
List<AuthenticationProvider> providers = new ArrayList<>();
providers.add(provider);
AuthenticationManager manager = new ProviderManager(providers);
Authentication result = manager.authenticate(authentication);
System.out.println(result);
```

如果我们的系统又支持基于用户和密码的服务，其数据是从数据库中获取的，同时又支持使用Token从Redis中验证Token的用户验证服务。在这样的场景下，
我们便只要额外在AuthenticationManager的AuthenticationProvider列表增加一个基于Token验证的AuthenticationProvider
（它的UserDetailsService则是通过Redis的API实现）便可了。这样的配置修改并不会影响到核心对外部暴露的服务接口和相关Authentication参数。

其实现类为AbstractUserDetailsAuthenticationProvider和其实现类的子类DaoAuthenticationProvider。
DaoAuthenticationProvider是Spring Security中一个核心的Provider,对所有的数据库提供了基本方法和入口。

**DaoAuthenticationProvider**
* 对用户身份进行加密操作：可以直接使用BCryptPasswordEncoder，或者实现自定义加密
* 实现抽象类的2个抽象方法：获取用户信息和验证密码




要提供一个完整的鉴权服务，我们至少需要完成以下两个任务：
1. 在预置的用户信息库中查找出当前需要鉴权的用户记录；
2. 验证Authentication中的鉴权凭证和用户名与预置查询出来的是否一致。

在实际开发中，我们的系统能支持多种鉴权实现，可能是对比预置用户密码是否一致，可能是对比一个令牌的值是否一致，也可以简单的认为
AuthenticationProvider实现的是如何进行身份验证的服务，通常我们也会称为“认证机制”。Spring Security自身已经提供了多种认证机制，
看看下面罗列的类名大概也就知道对应什么认证机制了：

* OAuth2LoginAuthenticationProvider
* CasAuthenticationProvider
* JwtAuthenticationProvider
* GoogleAccountsAuthenticationProvider


这些AuthenticationProvider中都有针对自身不同的认证机制的业务逻辑实现，上述的第二个任务，验证Authentication的鉴权凭证。而这些
`AuthenticationProvider`还都需要从预置的用户信息库中获取预置的用户信息才能完成比对工作。Spring Security中将获取用户预置的信息的服务，
提到了中：`	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;`

而预置的用户信息被封装在了UserDetails的类中,UserDetails中主要包含的基本与输入的Authentication差不多，
包含了用户名、密码、权限列表等基础的用户信息属性。


**UsernamePasswordAuthenticationFilter**
UsernamePasswordAuthenticationFilter是AbstractAuthenticationProcessingFilter针对使用用户名和密码进行身份验证而定制化的一个过滤器。
UsernamePasswordAuthenticationFilter继承扩展了AbstractAuthenticationProcessingFilter，相对与AbstractAuthenticationProcessingFilter
而言主要有以下几个改动：
* 属性中增加了username和password字段；
* 强制的只对POST请求应用；
* 重写了attemptAuthentication身份验证入口方法
而重写attemptAuthentication的方法主要是完了构建一个UsernamePasswordAuthenticationToken对象并且将其传递给AuthenticationManager进行身份验证。
```
UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
this.setDetails(request, authRequest);
return this.getAuthenticationManager().authenticate(authRequest);
```

**UsernamePasswordAuthenticationToken**













