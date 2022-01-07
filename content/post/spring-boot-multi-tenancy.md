---
title: "Multi-tenancy using Spring Boot and Hibernate"
date: 2021-11-13
description: "Implement Multi-tenancy using Spring boot"
categories:
  - "Development"
tags: 
    - "Development"
    - "Spring Boot"
    - "Java"
#math: false
lead: "Implement Multi-tenancy using Spring boot"
comments: true
mathjax: true
toc: false
authorbox: true
sidebar: "right" 
#menu: main
---
In modern Sass applications, a single application can be used by multiple companies(tenant). There are three ways with which one can divide an application by,

1. Single schema and DB instance
2. Separate Schema for each company but single DB instance
3. Different DB instance for each Company

In this article, I will be explaining about the second method , i.e Separate Schema for each tenant using Spring boot.

1. First we need to write the TenantContext class. Here we set the current tenant(schema) in the currently running Thread. The class is as follows,

```java
public class TenantContext {

	private static ThreadLocal<String> currentTenant = new InheritableThreadLocal<>();

	public static void setCurrentTenant(String tenant) {
		currentTenant.set(tenant);
	}

	public static String getCurrentTenant() {
		return currentTenant.get();
	}

	public static void clear() {
		currentTenant.remove();
	}

}
```

2. Now we need to implement the 'CurrentTenantIdentifierResolver'. This is used to set/resolve the current tenant. This class is as follows,

```java
import org.hibernate.context.spi.CurrentTenantIdentifierResolver;
import org.springframework.stereotype.Component;

import com.test.www.test.tenant.TenantContext;
import com.test.www.test.util.OotaConstants;

@Component
public class CurrentTenantIdentifierResolverImpl implements CurrentTenantIdentifierResolver {

	 private String defaultTenant ="DEFAULT_TENANT";

	    @Override
	    public String resolveCurrentTenantIdentifier() {
	        String t =  TenantContext.getCurrentTenant();
	        if(t!=null){
	            return t;
	        } else {
	            return defaultTenant;
	        }
	    }

	    @Override
	    public boolean validateExistingCurrentSessions() {
	        return true;
	    }
	}
```

3. The next step is to define 'MultiTenantConnectionProvider' class. Here we define how we go about getting the connection to Database. This class is as follow,

```java
import java.sql.Connection;
import java.sql.SQLException;

import javax.sql.DataSource;

import org.hibernate.engine.jdbc.connections.spi.MultiTenantConnectionProvider;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class MultiTenantConnectionProviderImpl implements MultiTenantConnectionProvider {

	/**
	 * 
	 */
	private static final long serialVersionUID = 1L;

	@Autowired
	private DataSource dataSource;
	private static Logger logger = LoggerFactory.getLogger(MultiTenantConnectionProviderImpl.class);

	public MultiTenantConnectionProviderImpl(DataSource dataSource) {
		this.dataSource = dataSource;
	}

	@Override
	public Connection getAnyConnection() throws SQLException {
		return dataSource.getConnection();
	}

	@Override
	public void releaseAnyConnection(Connection connection) throws SQLException {
		connection.close();
	}

	@Override
	public Connection getConnection(String tenantIdentifier) throws SQLException {
		logger.info("Get connection for tenant {}", tenantIdentifier);
		final Connection connection = getAnyConnection();
		connection.setSchema(tenantIdentifier);
		return connection;
	}

	@Override
	public void releaseConnection(String tenantIdentifier, Connection connection) throws SQLException {
		logger.info("Release connection for tenant {}", tenantIdentifier);
		connection.setSchema("DEFAULT_TENANT");
		releaseAnyConnection(connection);

	}

	@Override
	public boolean supportsAggressiveRelease() {
		return false;
	}

	@Override
	@SuppressWarnings("rawtypes")
	public boolean isUnwrappableAs(Class unwrapType) {
		return false;
	}

	@Override
	public <T> T unwrap(Class<T> unwrapType) {
		return null;
	}

}
```

4. Finally we need to set the hibernate configuration. We need to inform hibernate that we are using multi-tenancy and which kind of multi-tenancy at that. Code for the same is as below,

```java
import java.util.HashMap;
import java.util.Map;

import javax.sql.DataSource;

import org.hibernate.MultiTenancyStrategy;
import org.hibernate.cfg.Environment;
import org.hibernate.context.spi.CurrentTenantIdentifierResolver;
import org.hibernate.engine.jdbc.connections.spi.MultiTenantConnectionProvider;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.orm.jpa.JpaProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.orm.jpa.JpaVendorAdapter;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;

@Configuration
public class HibernateConfig {

  @Autowired
  private JpaProperties jpaProperties;

  @Bean
  public JpaVendorAdapter jpaVendorAdapter() {
    return new HibernateJpaVendorAdapter();
  }

  @Bean
  public LocalContainerEntityManagerFactoryBean entityManagerFactory(DataSource dataSource,
      MultiTenantConnectionProvider multiTenantConnectionProviderImpl,
      CurrentTenantIdentifierResolver currentTenantIdentifierResolverImpl) {
    Map<String, Object> properties = new HashMap<>();
    properties.putAll(jpaProperties.getProperties());
    properties.put(Environment.MULTI_TENANT, MultiTenancyStrategy.SCHEMA);
    properties.put(Environment.MULTI_TENANT_CONNECTION_PROVIDER, multiTenantConnectionProviderImpl);
    properties.put(Environment.MULTI_TENANT_IDENTIFIER_RESOLVER, currentTenantIdentifierResolverImpl);

    LocalContainerEntityManagerFactoryBean em = new LocalContainerEntityManagerFactoryBean();
    em.setDataSource(dataSource);
    em.setPackagesToScan("com.test.www.test");
    em.setJpaVendorAdapter(jpaVendorAdapter());
    em.setJpaPropertyMap(properties);
    return em;
  }
}
```

As shown in above code, we need to set below properties for multi-tenancy to work,

```java
properties.put(Environment.MULTI_TENANT, MultiTenancyStrategy.SCHEMA);
properties.put(Environment.MULTI_TENANT_CONNECTION_PROVIDER, multiTenantConnectionProviderImpl);
properties.put(Environment.MULTI_TENANT_IDENTIFIER_RESOLVER, currentTenantIdentifierResolverImpl);
```

5. Now let's test this multi-tenancy in a simple REST application

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import com.test.www.test.pojos.User;
import com.test.www.test.services.UserService;

@RestController
public class UserController {
	
	@Autowired
	private UserService userRegistrationService;
	
	@GetMapping("/user/{userName}")
	public User getUserDetailsByUserName(@PathVariable(name = "userName",required=true) String userName) {
		return userRegistrationService.getUserByUserName(userName);
	}
}
```

```java
package com.test.www.test.services.impl;

import java.util.Date;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.test.www.test.entities.UserDetails;
import com.test.www.test.repository.UserDetailsRepository;
import com.test.www.test.services.UserService;

@Service
public class UserServiceImpl implements UserService{
	@Autowired
	private UserDetailsRepository userDetailsRepository;
	
	@Override
	public UserDetails getUserByUserName(String username) {
		UserDetails userDetails = userDetailsRepository.getUserByUserName(username);
		return userDetails;
	}
}
```

The repository class 'UserDetailsRepository' is as shown below,

```java
package com.test.www.test.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import com.test.www.test.entities.UserDetails;

@Repository
public interface UserDetailsRepository extends JpaRepository<UserDetails,Integer>{
	@Query(value="SELECT u FROM UserDetails u where userName =:user_name")
	UserDetails getUserByUserName(@Param("user_name")String userName);
}
```

So that is how we go about implementing multi-tenancy in Java using Spring Boot and Hibernate.