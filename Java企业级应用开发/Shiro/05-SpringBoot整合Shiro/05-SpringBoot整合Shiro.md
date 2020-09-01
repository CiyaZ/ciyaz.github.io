# SpringBoot整合Shiro

实际上，SpringBoot中整合Shiro和传统Spring工程是一样的，只不过是把XML配置换成了JavaConfig，这里记录一下作为备用。

## Maven依赖

首先创建一个SpringBoot工程，然后引入Shiro的依赖，这里我们使用的SpringBoot版本是`2.3.3`，Shiro版本是`1.6.0`。

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>${shiro.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-web</artifactId>
    <version>${shiro.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>${shiro.version}</version>
</dependency>
```

除此之外，一般还需要`commons-codec`等和认证相关的依赖，这里就不多介绍了。

## JavaConfig

ShiroConfig.java
```java
package com.ciyaz.demo.config;

import java.util.HashMap;
import java.util.Map;

import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.ciyaz.demo.realm.SystemRealm;

@Configuration
public class ShiroConfig {

	@Bean
	public SystemRealm systemRealm() {
		return new SystemRealm();
	}

	@Bean
	public SecurityManager securityManager() {
		DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
		securityManager.setRealm(systemRealm());
		return securityManager;
	}

	@Bean
	public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
		ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
		shiroFilterFactoryBean.setSecurityManager(securityManager);
		Map<String, String> filterMap = new HashMap<String, String>();
		filterMap.put("/admin/**", "authc");
		shiroFilterFactoryBean.setLoginUrl("/auth/login");
		shiroFilterFactoryBean.setFilterChainDefinitionMap(filterMap);
		return shiroFilterFactoryBean;
	}

	@Bean
	public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
		AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
		authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
		return authorizationAttributeSourceAdvisor;
	}
}
```

在JavaConfig中，我们配置了自定义Realm、SecurityManager、过滤器、注解扫描。

这样配置好后，我们根据具体工程中所选的各种其它组件，编写好Realm，以及相关认证、鉴权逻辑，Shiro就可以正常使用了。
