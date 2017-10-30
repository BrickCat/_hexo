---
title: 解决springboot-shiro-session设置过期时间无效的问题
date: 2017-10-30 14:19:19
tags: [springboot,shiro,常见问题]
categories: springboot framework
---

### 问题描述

在`springboot`整合`shiro`的时候session总是过期，往往是刚刚登录没一两分钟就又要重新登陆。网上查了很多资料
说是可以设置session的过期时间`server.session.timeout`，无论你设置成多少都无效，后来才知道此session和shiro的session根本就是两回事。

### 解决办法
下边我们就来看看在springboot中如何设置shiro的session过期时间。废话不多说直接上代码吧！

```java
    /**
	 * 添加session管理器
	 * @return
	 */
	@Bean(name="sessionManager")
	public DefaultWebSessionManager defaultWebSessionManager() {
		System.out.println("ShiroConfiguration.defaultWebSessionManager()");
		DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
		sessionManager.setCacheManager(cacheManager());
		sessionManager.setSessionValidationInterval(3600000*12);
		sessionManager.setGlobalSessionTimeout(3600000*12);
		sessionManager.setDeleteInvalidSessions(true);
		sessionManager.setSessionValidationSchedulerEnabled(true);
		Cookie cookie = new SimpleCookie(ShiroHttpSession.DEFAULT_SESSION_ID_NAME);
		cookie.setName("ITBC");
		cookie.setHttpOnly(true);
		sessionManager.setSessionIdCookie(cookie);
		return sessionManager;
	}
	/**
     * 用户授权信息Cache
     */
    @Bean(name = "shiroCacheManager")
    @ConditionalOnMissingBean
    public CacheManager cacheManager() {
        return new MemoryConstrainedCacheManager();
    }
    
    @Bean(name = "securityManager")
    @ConditionalOnMissingBean
    public DefaultSecurityManager securityManager() {
        DefaultSecurityManager sm = new DefaultWebSecurityManager();
        //添加缓存
        sm.setCacheManager(cacheManager());
        //添加session
    	sm.setSessionManager(defaultWebSessionManager());
        return sm;
    }
```
### 后记
其实在springboot的实际开发中博主遇到了很多坑，网上的解决办法也是零零碎碎的，总感觉不是自己想要的。有时间整理整理总结一下，发出来大家一起学习。