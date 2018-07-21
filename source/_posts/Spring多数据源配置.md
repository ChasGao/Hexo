---
title: Spring多数据源配置
date: 2018-7-14 18:10:45
tags:
	- Spring多数据源配置
categories: 
	- Spring	
---

继承Spring的AbstractRoutingDataSource来实现多数据源配置

1. 数据源配置
```xml
<!-- 数据库连接池配置 -->
 <bean id="dataSourceTask" name="dataSourceTask" class="com.mchange.v2.c3p0.ComboPooledDataSource">  
	<!-- 指定连接数据库的驱动-->  
	<property name="driverClass" value="${db.task.jdbc.driverClassName}"/>  
	<!-- 指定连接数据库的URL-->  
	<property name="jdbcUrl" value="${db.task.jdbc.url}"/>  
	<!-- 指定连接数据库的用户名-->  
	<property name="user" value="${db.task.jdbc.user}"/>  
	<!-- 指定连接数据库的密码-->  
	<property name="password" value="${db.task.jdbc.password}"/>  
	<!-- 指定连接池中保留的最大连接数. Default:15-->  
	<property name="maxPoolSize" value="${db.task.jdbc.maxPoolSize}"/>  
	<!-- 指定连接池中保留的最小连接数-->  
	<property name="minPoolSize" value="${db.task.jdbc.minPoolSize}"/>  
	<!-- 指定连接池的初始化连接数  取值应在minPoolSize 与 maxPoolSize 之间.Default:3-->  
	<property name="initialPoolSize" value="${db.task.jdbc.initialPoolSize}"/>  
	<!-- 最大空闲时间,60秒内未使用则连接被丢弃。若为0则永不丢弃。 Default:0-->  
	<property name="maxIdleTime" value="${db.task.jdbc.maxIdleTime}"/>  
	<!-- 当连接池中的连接耗尽的时候c3p0一次同时获取的连接数. Default:3-->  
	<property name="acquireIncrement" value="${db.task.jdbc.acquireIncrement}"/>  
	<!-- JDBC的标准,用以控制数据源内加载的PreparedStatements数量。但由于预缓存的statements属于单个connection
		而不是整个连接池所以设置这个参数需要考虑到多方面的因数.如果maxStatements与maxStatementsPerConnection均为0,则缓存被关闭。Default:0-->  
	<property name="maxStatements" value="${db.task.jdbc.maxStatements}"/>  
	<!-- 每60秒检查所有连接池中的空闲连接.Default:0 -->  
	<property name="idleConnectionTestPeriod" value="${db.task.jdbc.idleConnectionTestPeriod}"/>
	<!--连接池获取新连接的时间，超时后将 抛出SQLException,如设为0则无限期等待。单位毫秒。Default: 0 -->
	<property name="checkoutTimeout" value="${db.task.jdbc.checkoutTimeout}"/>  
</bean>
 
 <bean id="dataSourceEnv" name="dataSourceEnv" class="com.mchange.v2.c3p0.ComboPooledDataSource">  
	<!-- 指定连接数据库的驱动-->  
	<property name="driverClass" value="${db.lcmyw.jdbc.driverClassName}"/>  
	<!-- 指定连接数据库的URL-->  
	<property name="jdbcUrl" value="${db.lcmyw.jdbc.url}"/>  
	<!-- 指定连接数据库的用户名-->  
	<property name="user" value="${db.lcmyw.jdbc.user}"/>  
	<!-- 指定连接数据库的密码-->  
	<property name="password" value="${db.lcmyw.jdbc.password}"/>  
	<!-- 指定连接池中保留的最大连接数. Default:15-->  
	<property name="maxPoolSize" value="${db.lcmyw.jdbc.maxPoolSize}"/>  
	<!-- 指定连接池中保留的最小连接数-->  
	<property name="minPoolSize" value="${db.lcmyw.jdbc.minPoolSize}"/>  
	<!-- 指定连接池的初始化连接数  取值应在minPoolSize 与 maxPoolSize 之间.Default:3-->  
	<property name="initialPoolSize" value="${db.lcmyw.jdbc.initialPoolSize}"/>  
	<!-- 最大空闲时间,60秒内未使用则连接被丢弃。若为0则永不丢弃。 Default:0-->  
	<property name="maxIdleTime" value="${db.lcmyw.jdbc.maxIdleTime}"/>  
	<!-- 当连接池中的连接耗尽的时候c3p0一次同时获取的连接数. Default:3-->  
	<property name="acquireIncrement" value="${db.lcmyw.jdbc.acquireIncrement}"/>  
	<!-- JDBC的标准,用以控制数据源内加载的PreparedStatements数量。但由于预缓存的statements属于单个connection
		而不是整个连接池所以设置这个参数需要考虑到多方面的因数.如果maxStatements与maxStatementsPerConnection均为0,则缓存被关闭。Default:0-->  
	<property name="maxStatements" value="${db.lcmyw.jdbc.maxStatements}"/>  
	<!-- 每60秒检查所有连接池中的空闲连接.Default:0 -->  
	<property name="idleConnectionTestPeriod" value="${db.lcmyw.jdbc.idleConnectionTestPeriod}"/>
	<!--连接池获取新连接的时间，超时后将 抛出SQLException,如设为0则无限期等待。单位毫秒。Default: 0 -->
	<property name="checkoutTimeout" value="${db.lcmyw.jdbc.checkoutTimeout}"/>  
</bean>
```

2. 定义一个类继承AbstractRoutingDataSource实现determineCurrentLookupKey方法，来实现数据库的动态切换

```java
package com.johnfnash.learn.pro.dashboardserver.core;
 
import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;
 
/**
 * 
* ClassName: DynamicDataSource <br/> 
* Function: 扩展Spring的AbstractRoutingDataSource抽象类，实现动态数据源. <br/> 
* date: 2017年3月15日 下午10:23:52 <br/> 
* 
* @author JohnFNash 
* @version  
* @since JDK 1.6
 */
public class DynamicDataSource extends AbstractRoutingDataSource {
 
	@Override
	protected Object determineCurrentLookupKey() {
		// 从自定义的位置获取数据源标识
		return DynamicDataSourceHolder.getDatasource();
	}

}
```

3. 定义工具类，用于动态切换数据源

```java
package com.johnfnash.learn.pro.dashboardserver.core;
 
/**
 * 
* ClassName: DatabaseContextHolder <br/> 
* Function: 当前线程中使用的数据源标识 <br/> 
* date: 2017年3月15日 下午10:10:17 <br/> 
* 
* @author JohnFNash 
* @version  
* @since JDK 1.6
 */
public class DynamicDataSourceHolder {
	/** 数据源标识保存在线程变量中，避免多线程操作数据源时互相干扰 */
	private static final ThreadLocal<String> contextHolder = new ThreadLocal<String>();
	
	/**
	 * 设置数据源名称
	 * @param dastsource 数据源名称
	 */
	public static void setDatasource(String datasource) {
		contextHolder.set(datasource);
	}
	
	/**
	 * 获取数据源名称
	 * @return 数据源名称
	 */
	public static String getDatasource() {
		return contextHolder.get();
	}
	
	/**
	 * 清除标志
	 */
	public static void clearDatasource() {
		contextHolder.remove();
	}
	
}
```

4. 在数据库配置文件中添加如下配置

```xml
<!-- 动态数据源配置 -->
<bean id="dataSource" class="com.johnfnash.learn.pro.dashboardserver.core.DynamicDataSource">
	<property name="targetDataSources">
		<map key-type="java.lang.String">
			<entry key="dataSourceTask" value-ref="dataSourceTask"></entry>
			<entry key="dataSourceEnv" value-ref="dataSourceEnv"></entry>
		</map>
	</property>
	<property name="defaultTargetDataSource" ref="dataSourceEnv"></property>
</bean>
 
<!-- spring和MyBatis完美整合，不需要mybatis的配置映射文件 -->
 <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
	<property name="dataSource" ref="dataSource" />
	<!-- 自动扫描mapping.xml文件 -->
	<property name="mapperLocations" value="classpath:/mapper_*/*.xml"/>
 </bean>
 
 <!-- DAO接口所在包名，Spring会自动查找其下的类 -->
 <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
	<property name="basePackage" value="com.johnfnash.learn.pro.dashboardserver.dao" />
	<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
 </bean>
 
 <!-- 配置事物管理器 -->
 <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource" />
 </bean>
 
 <!-- 拦截器方式配置事物 -->
 <tx:advice id="transactionAdvice" transaction-manager="transactionManager">
	<tx:attributes>
		<tx:method name="add*" propagation="REQUIRED" />
		<tx:method name="insert*" propagation="REQUIRED" />
		<tx:method name="update*" propagation="REQUIRED" />
		<tx:method name="delete*" propagation="REQUIRED" />

		<tx:method name="get*" propagation="SUPPORTS" read-only="true" />
		<tx:method name="find*" propagation="SUPPORTS" read-only="true" />
		<tx:method name="select*" propagation="SUPPORTS" read-only="true" />
	</tx:attributes>     	
 </tx:advice>
 
 <!-- spring aop事物管理 -->
 <aop:config>
       <aop:pointcut id="transactionPointcut" 
        expression="execution(* com.johnfnash.learn.pro.dashboardserver.service..*Impl.*(..))" />
    <aop:advisor pointcut-ref="transactionPointcut" advice-ref="transactionAdvice" order="2"/>    
 </aop:config>
```

到目前为止，动态数据源的配置以及OK，不过还不能实现动态切换，下面将介绍
5. 数据源动态切换

1） 方式一： 定义切面，设置切面在数据库事物开启前作用，切换数据源
```java
package com.johnfnash.learn.pro.dashboardserver.core;
 
/**
 * 
* ClassName: DynamicDataSourceInterceptor <br/> 
* Function: 数据源动态切换拦截器. <br/> 
* date: 2017年3月15日 下午10:28:17 <br/> 
* 
* @author JohnFNash 
* @version  
* @since JDK 1.6
 */
public class DynamicDataSourceInterceptor {
    /** 切换到数据源1 */
    public void setDS1() {
    	DynamicDataSourceHolder.setDataSource("dataSourceTask");
    }
    
    
 
    /** 切换到数据源2 */
 
    public void setDS2() { 
      DynamicDataSourceHolder.setDataSource("dataSourceEnv"); 
    }
}
```
配置切面，进行数据源动态切换
```xml

<!-- 数据源动态切换实体 -->
<bean id="dataSourceInterceptor" class="com.johnfnash.learn.pro.dashboardserver.core.DynamicDataSourceInterceptor"/>
 
<!-- 数据源动态切换切面配置 -->
<aop:config>
<aop:aspect id="dataSourceAspect" ref="dataSourceInterceptor" order="1">
	<!-- 拦截所有service实现类的方法 -->
	<aop:pointcut id="dataSourcePointcutTask" 
		expression="execution(* com.johnfnash.learn.pro.dashboardserver.service.task.*Impl.*(..))"/>
	<aop:before pointcut-ref="dataSourcePointcutTask" method="setDS1" />
	<aop:pointcut id="dataSourcePointcutEnv" 
		expression="execution(* com.johnfnash.learn.pro.dashboardserver.service.env.*Impl.*(..))"/>
	<aop:before pointcut-ref="dataSourcePointcutEnv" method="setDS2" />
</aop:aspect>
</aop:config>
```

注意，需要将数据源动态切换的切面的执行顺序设置为1，将数据库事物管理的执行顺序设置为2，在数据库

事物开启前进行数据源切换

至此，方式一 多个数据源配置以及数据源动态切换功能已经实现，完整配置如下
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:context="http://www.springframework.org/schema/context"  
    xmlns:tx="http://www.springframework.org/schema/tx"  
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="  
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd  
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
     <!-- 数据库连接池配置 -->
     <bean id="dataSourceTask" name="dataSourceTask" class="com.mchange.v2.c3p0.ComboPooledDataSource">  
        <!-- 指定连接数据库的驱动-->  
        <property name="driverClass" value="${db.task.jdbc.driverClassName}"/>  
        <!-- 指定连接数据库的URL-->  
        <property name="jdbcUrl" value="${db.task.jdbc.url}"/>  
        <!-- 指定连接数据库的用户名-->  
        <property name="user" value="${db.task.jdbc.user}"/>  
        <!-- 指定连接数据库的密码-->  
        <property name="password" value="${db.task.jdbc.password}"/>  
        <!-- 指定连接池中保留的最大连接数. Default:15-->  
        <property name="maxPoolSize" value="${db.task.jdbc.maxPoolSize}"/>  
        <!-- 指定连接池中保留的最小连接数-->  
        <property name="minPoolSize" value="${db.task.jdbc.minPoolSize}"/>  
        <!-- 指定连接池的初始化连接数  取值应在minPoolSize 与 maxPoolSize 之间.Default:3-->  
        <property name="initialPoolSize" value="${db.task.jdbc.initialPoolSize}"/>  
        <!-- 最大空闲时间,60秒内未使用则连接被丢弃。若为0则永不丢弃。 Default:0-->  
        <property name="maxIdleTime" value="${db.task.jdbc.maxIdleTime}"/>  
        <!-- 当连接池中的连接耗尽的时候c3p0一次同时获取的连接数. Default:3-->  
        <property name="acquireIncrement" value="${db.task.jdbc.acquireIncrement}"/>  
        <!-- JDBC的标准,用以控制数据源内加载的PreparedStatements数量。但由于预缓存的statements属于单个connection
        	而不是整个连接池所以设置这个参数需要考虑到多方面的因数.如果maxStatements与maxStatementsPerConnection均为0,则缓存被关闭。Default:0-->  
        <property name="maxStatements" value="${db.task.jdbc.maxStatements}"/>  
        <!-- 每60秒检查所有连接池中的空闲连接.Default:0 -->  
        <property name="idleConnectionTestPeriod" value="${db.task.jdbc.idleConnectionTestPeriod}"/>
        <!--连接池获取新连接的时间，超时后将 抛出SQLException,如设为0则无限期等待。单位毫秒。Default: 0 -->
		<property name="checkoutTimeout" value="${db.task.jdbc.checkoutTimeout}"/>  
    </bean>
     
     <bean id="dataSourceEnv" name="dataSourceEnv" class="com.mchange.v2.c3p0.ComboPooledDataSource">  
        <!-- 指定连接数据库的驱动-->  
        <property name="driverClass" value="${db.lcmyw.jdbc.driverClassName}"/>  
        <!-- 指定连接数据库的URL-->  
        <property name="jdbcUrl" value="${db.lcmyw.jdbc.url}"/>  
        <!-- 指定连接数据库的用户名-->  
        <property name="user" value="${db.lcmyw.jdbc.user}"/>  
        <!-- 指定连接数据库的密码-->  
        <property name="password" value="${db.lcmyw.jdbc.password}"/>  
        <!-- 指定连接池中保留的最大连接数. Default:15-->  
        <property name="maxPoolSize" value="${db.lcmyw.jdbc.maxPoolSize}"/>  
        <!-- 指定连接池中保留的最小连接数-->  
        <property name="minPoolSize" value="${db.lcmyw.jdbc.minPoolSize}"/>  
        <!-- 指定连接池的初始化连接数  取值应在minPoolSize 与 maxPoolSize 之间.Default:3-->  
        <property name="initialPoolSize" value="${db.lcmyw.jdbc.initialPoolSize}"/>  
        <!-- 最大空闲时间,60秒内未使用则连接被丢弃。若为0则永不丢弃。 Default:0-->  
        <property name="maxIdleTime" value="${db.lcmyw.jdbc.maxIdleTime}"/>  
        <!-- 当连接池中的连接耗尽的时候c3p0一次同时获取的连接数. Default:3-->  
        <property name="acquireIncrement" value="${db.lcmyw.jdbc.acquireIncrement}"/>  
        <!-- JDBC的标准,用以控制数据源内加载的PreparedStatements数量。但由于预缓存的statements属于单个connection
        	而不是整个连接池所以设置这个参数需要考虑到多方面的因数.如果maxStatements与maxStatementsPerConnection均为0,则缓存被关闭。Default:0-->  
        <property name="maxStatements" value="${db.lcmyw.jdbc.maxStatements}"/>  
        <!-- 每60秒检查所有连接池中的空闲连接.Default:0 -->  
        <property name="idleConnectionTestPeriod" value="${db.lcmyw.jdbc.idleConnectionTestPeriod}"/>
        <!--连接池获取新连接的时间，超时后将 抛出SQLException,如设为0则无限期等待。单位毫秒。Default: 0 -->
		<property name="checkoutTimeout" value="${db.lcmyw.jdbc.checkoutTimeout}"/>  
    </bean>
    
    <bean id="dataSource" class="com.johnfnash.learn.pro.dashboardserver.core.DynamicDataSource">
    	<property name="targetDataSources">
    		<map key-type="java.lang.String">
    			<entry key="dataSourceTask" value-ref="dataSourceTask"></entry>
    			<entry key="dataSourceEnv" value-ref="dataSourceEnv"></entry>
    		</map>
    	</property>
    	<property name="defaultTargetDataSource" ref="dataSourceEnv"></property>
    </bean>
    
    <!-- spring和MyBatis完美整合，不需要mybatis的配置映射文件 -->
     <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
     	<property name="dataSource" ref="dataSource" />
     	<!-- 自动扫描mapping.xml文件 -->
     	<property name="mapperLocations" value="classpath:/mapper_*/*.xml"/>
     </bean>
     
     <!-- DAO接口所在包名，Spring会自动查找其下的类 -->
     <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
     	<property name="basePackage" value="com.johnfnash.learn.pro.dashboardserver.dao" />
     	<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
     </bean>
     
     <!-- 配置事物管理器 -->
     <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
     	<property name="dataSource" ref="dataSource" />
     </bean>
     
     <!-- 拦截器方式配置事物 -->
     <tx:advice id="transactionAdvice" transaction-manager="transactionManager">
		<tx:attributes>
			<tx:method name="add*" propagation="REQUIRED" />
			<tx:method name="insert*" propagation="REQUIRED" />
			<tx:method name="update*" propagation="REQUIRED" />
			<tx:method name="delete*" propagation="REQUIRED" />
			
			<tx:method name="get*" propagation="SUPPORTS" read-only="true" />
			<tx:method name="find*" propagation="SUPPORTS" read-only="true" />
			<tx:method name="select*" propagation="SUPPORTS" read-only="true" />
		</tx:attributes>     	
     </tx:advice>
     
     <!-- spring aop事物管理 -->
     <aop:config>
			<aop:pointcut id="transactionPointcut" 
     			expression="execution(* com.johnfnash.learn.pro.dashboardserver.service..*Impl.*(..))" />
     		<aop:advisor pointcut-ref="transactionPointcut" advice-ref="transactionAdvice" order="2"/>
     		<!-- <aop:advisor pointcut-ref="transactionPointcut" advice-ref="dataSourceInterceptor" order="1"/> -->     	
     </aop:config>
     
     <!-- 数据源动态切换切面配置 -->
     <aop:config>
	 <aop:aspect id="dataSourceAspect" ref="dataSourceInterceptor" order="1">
            <aop:pointcut id="dataSourcePointcutTask" 
            	expression="execution(* com.johnfnash.learn.pro.dashboardserver.service.task.*Impl.*(..))"/>
            <aop:before pointcut-ref="dataSourcePointcutTask" method="setDS1" />
            <aop:pointcut id="dataSourcePointcutEnv" 
            	expression="execution(* com.johnfnash.learn.pro.dashboardserver.service.env.*Impl.*(..))"/>
            <aop:before pointcut-ref="dataSourcePointcutEnv" method="setDS2" />
		</aop:aspect>
	 </aop:config>
     
     <!-- 数据源动态切换实体 -->
     <bean id="dataSourceInterceptor" class="com.johnfnash.learn.pro.dashboardserver.core.DynamicDataSourceInterceptor"/>    
</beans>
```
2）方式二：定义注解，通过注解的值来获取当前数据源，并进行切换

定义注解，作用于类和方法

```java
package com.johnfnash.learn.pro.dashboardserver.core;
 
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
 
@Target({ ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface DataSource {
 
	String value();
}
```

定义拦截器，解析注解切换数据源
```java
package com.johnfnash.learn.pro.dashboardserver.core;
 
import java.lang.reflect.Method;
 
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.reflect.MethodSignature;
 
/**
 * 
* ClassName: DynamicDataSourceInterceptor <br/> 
* Function: 数据源动态切换拦截器. <br/> 
* date: 2017年3月15日 下午10:28:17 <br/> 
* 
* @author JohnFNash 
* @version  
* @since JDK 1.6
 */
public class DynamicDataSourceInterceptor {
		
    /**
     * 拦截目标方法，获取由@DataSource指定的数据源标识，设置到线程存储中以便切换数据源
     * 
     * @param point
     * @throws Exception
     */
    public void intercept(JoinPoint point) throws Exception {
        Class<?> target = point.getTarget().getClass();
        MethodSignature signature = (MethodSignature) point.getSignature();
        resolveDataSource(target, signature.getMethod());
    }
 
    /**
     * 提取目标对象方法注解和类型注解中的数据源标识
     * 
     * @param clazz
     * @param method
     */
    private void resolveDataSource(Class<?> clazz, Method method) {
        try {
            Class<?>[] types = method.getParameterTypes();
            // 默认使用类型注解
            if (clazz.isAnnotationPresent(DataSource.class)) {
                DataSource source = clazz.getAnnotation(DataSource.class);
                DynamicDataSourceHolder.setDataSource(source.value());
            }
            // 方法注解可以覆盖类型注解
            Method m = clazz.getMethod(method.getName(), types);
            if (m != null && m.isAnnotationPresent(DataSource.class)) {
                DataSource source = m.getAnnotation(DataSource.class);
                DynamicDataSourceHolder.setDataSource(source.value());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

在配置文件中配置切面，实现数据源动态切换
```xml
<!-- 数据源动态切换切面配置 -->
<aop:config>
<aop:aspect id="dataSourceAspect" ref="dataSourceInterceptor" order="1">
	<!-- 拦截所有service实现类的方法 -->
	<aop:pointcut id="dataSourcePointcut" 
		expression="execution(* com.johnfnash.learn.pro.dashboardserver.service..*Impl.*(..))"/>
	<aop:before pointcut-ref="dataSourcePointcut" method="intercept" />	
</aop:aspect>
</aop:config>
 
<!-- 数据源动态切换实体 -->
<bean id="dataSourceInterceptor" class="com.johnfnash.learn.pro.dashboardserver.core.DynamicDataSourceInterceptor"/>
```

注意：设置执行顺序为1，并使用 aop:before 在数据库事物开启前进行数据源切换
至此，方式二 多个数据源配置以及数据源动态切换功能已经实现，完整配置如下
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:context="http://www.springframework.org/schema/context"  
    xmlns:tx="http://www.springframework.org/schema/tx"  
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="  
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd  
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
     <!-- 数据库连接池配置 -->
     <bean id="dataSourceTask" name="dataSourceTask" class="com.mchange.v2.c3p0.ComboPooledDataSource">  
        <!-- 指定连接数据库的驱动-->  
        <property name="driverClass" value="${db.task.jdbc.driverClassName}"/>  
        <!-- 指定连接数据库的URL-->  
        <property name="jdbcUrl" value="${db.task.jdbc.url}"/>  
        <!-- 指定连接数据库的用户名-->  
        <property name="user" value="${db.task.jdbc.user}"/>  
        <!-- 指定连接数据库的密码-->  
        <property name="password" value="${db.task.jdbc.password}"/>  
        <!-- 指定连接池中保留的最大连接数. Default:15-->  
        <property name="maxPoolSize" value="${db.task.jdbc.maxPoolSize}"/>  
        <!-- 指定连接池中保留的最小连接数-->  
        <property name="minPoolSize" value="${db.task.jdbc.minPoolSize}"/>  
        <!-- 指定连接池的初始化连接数  取值应在minPoolSize 与 maxPoolSize 之间.Default:3-->  
        <property name="initialPoolSize" value="${db.task.jdbc.initialPoolSize}"/>  
        <!-- 最大空闲时间,60秒内未使用则连接被丢弃。若为0则永不丢弃。 Default:0-->  
        <property name="maxIdleTime" value="${db.task.jdbc.maxIdleTime}"/>  
        <!-- 当连接池中的连接耗尽的时候c3p0一次同时获取的连接数. Default:3-->  
        <property name="acquireIncrement" value="${db.task.jdbc.acquireIncrement}"/>  
        <!-- JDBC的标准,用以控制数据源内加载的PreparedStatements数量。但由于预缓存的statements属于单个connection
        	而不是整个连接池所以设置这个参数需要考虑到多方面的因数.如果maxStatements与maxStatementsPerConnection均为0,则缓存被关闭。Default:0-->  
        <property name="maxStatements" value="${db.task.jdbc.maxStatements}"/>  
        <!-- 每60秒检查所有连接池中的空闲连接.Default:0 -->  
        <property name="idleConnectionTestPeriod" value="${db.task.jdbc.idleConnectionTestPeriod}"/>
        <!--连接池获取新连接的时间，超时后将 抛出SQLException,如设为0则无限期等待。单位毫秒。Default: 0 -->
		<property name="checkoutTimeout" value="${db.task.jdbc.checkoutTimeout}"/>  
    </bean>
     
     <bean id="dataSourceEnv" name="dataSourceEnv" class="com.mchange.v2.c3p0.ComboPooledDataSource">  
        <!-- 指定连接数据库的驱动-->  
        <property name="driverClass" value="${db.lcmyw.jdbc.driverClassName}"/>  
        <!-- 指定连接数据库的URL-->  
        <property name="jdbcUrl" value="${db.lcmyw.jdbc.url}"/>  
        <!-- 指定连接数据库的用户名-->  
        <property name="user" value="${db.lcmyw.jdbc.user}"/>  
        <!-- 指定连接数据库的密码-->  
        <property name="password" value="${db.lcmyw.jdbc.password}"/>  
        <!-- 指定连接池中保留的最大连接数. Default:15-->  
        <property name="maxPoolSize" value="${db.lcmyw.jdbc.maxPoolSize}"/>  
        <!-- 指定连接池中保留的最小连接数-->  
        <property name="minPoolSize" value="${db.lcmyw.jdbc.minPoolSize}"/>  
        <!-- 指定连接池的初始化连接数  取值应在minPoolSize 与 maxPoolSize 之间.Default:3-->  
        <property name="initialPoolSize" value="${db.lcmyw.jdbc.initialPoolSize}"/>  
        <!-- 最大空闲时间,60秒内未使用则连接被丢弃。若为0则永不丢弃。 Default:0-->  
        <property name="maxIdleTime" value="${db.lcmyw.jdbc.maxIdleTime}"/>  
        <!-- 当连接池中的连接耗尽的时候c3p0一次同时获取的连接数. Default:3-->  
        <property name="acquireIncrement" value="${db.lcmyw.jdbc.acquireIncrement}"/>  
        <!-- JDBC的标准,用以控制数据源内加载的PreparedStatements数量。但由于预缓存的statements属于单个connection
        	而不是整个连接池所以设置这个参数需要考虑到多方面的因数.如果maxStatements与maxStatementsPerConnection均为0,则缓存被关闭。Default:0-->  
        <property name="maxStatements" value="${db.lcmyw.jdbc.maxStatements}"/>  
        <!-- 每60秒检查所有连接池中的空闲连接.Default:0 -->  
        <property name="idleConnectionTestPeriod" value="${db.lcmyw.jdbc.idleConnectionTestPeriod}"/>
        <!--连接池获取新连接的时间，超时后将 抛出SQLException,如设为0则无限期等待。单位毫秒。Default: 0 -->
		<property name="checkoutTimeout" value="${db.lcmyw.jdbc.checkoutTimeout}"/>  
    </bean>
    
    <bean id="dataSource" class="com.johnfnash.learn.pro.dashboardserver.core.DynamicDataSource">
    	<property name="targetDataSources">
    		<map key-type="java.lang.String">
    			<entry key="dataSourceTask" value-ref="dataSourceTask"></entry>
    			<entry key="dataSourceEnv" value-ref="dataSourceEnv"></entry>
    		</map>
    	</property>
    	<property name="defaultTargetDataSource" ref="dataSourceEnv"></property>
    </bean>
    
    <!-- spring和MyBatis完美整合，不需要mybatis的配置映射文件 -->
     <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
     	<property name="dataSource" ref="dataSource" />
     	<!-- 自动扫描mapping.xml文件 -->
     	<property name="mapperLocations" value="classpath:/mapper_*/*.xml"/>
     </bean>
     
     <!-- DAO接口所在包名，Spring会自动查找其下的类 -->
     <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
     	<property name="basePackage" value="com.johnfnash.learn.pro.dashboardserver.dao" />
     	<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
     </bean>
     
     <!-- 配置事物管理器 -->
     <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
     	<property name="dataSource" ref="dataSource" />
     </bean>
     
     <!-- 拦截器方式配置事物 -->
     <tx:advice id="transactionAdvice" transaction-manager="transactionManager">
		<tx:attributes>
			<tx:method name="add*" propagation="REQUIRED" />
			<tx:method name="insert*" propagation="REQUIRED" />
			<tx:method name="update*" propagation="REQUIRED" />
			<tx:method name="delete*" propagation="REQUIRED" />
			
			<tx:method name="get*" propagation="SUPPORTS" read-only="true" />
			<tx:method name="find*" propagation="SUPPORTS" read-only="true" />
			<tx:method name="select*" propagation="SUPPORTS" read-only="true" />
		</tx:attributes>     	
     </tx:advice>
     
     <!-- spring aop事物管理 -->
     <aop:config>
	<aop:pointcut id="transactionPointcut" 
     			expression="execution(* com.johnfnash.learn.pro.dashboardserver.service..*Impl.*(..))" />
        <!-- 设置order的值为2，使得数据库事物开启在数据源切换之后，否则数据源切换不会达到效果  -->
       <aop:advisor pointcut-ref="transactionPointcut" advice-ref="transactionAdvice" order="2"/>
     </aop:config>
     
     <!-- 数据源动态切换切面配置 -->
     <aop:config>
	    <aop:aspect id="dataSourceAspect" ref="dataSourceInterceptor" order="1">
			<!-- 拦截所有service实现类的方法 -->
            <aop:pointcut id="dataSourcePointcut" 
            	     expression="execution(* com.johnfnash.learn.pro.dashboardserver.service..*Impl.*(..))"/>
                <aop:before pointcut-ref="dataSourcePointcut" method="intercept" />           
	    </aop:aspect>
      </aop:config>
     
     <!-- 数据源动态切换实体 -->
     <bean id="dataSourceInterceptor" class="com.johnfnash.learn.pro.dashboardserver.core.DynamicDataSourceInterceptor"/>
     
</beans>
```

@DataSource注解使用示例:
```java
package com.johnfnash.learn.pro.dashboardserver.service.env;
 
import java.util.List;
import java.util.Map;
 
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
 
import com.johnfnash.learn.pro.dashboardserver.core.DataSource;
import com.johnfnash.learn.pro.dashboardserver.dao.env.TaskInfoMapper;
import com.johnfnash.learn.pro.dashboardserver.domain.TaskInfo;
 
/**
 * 
* ClassName: FactTskServiceImpl <br/> 
* Function: 任务信息业务逻辑操作实现类 <br/> 
* date: 2017年3月8日 下午10:09:46 <br/> 
* 
* @author JohnFNash 
* @version  
* @since JDK 1.6
 */
@Service("taskInfoService")
@DataSource("dataSourceEnv")
public class TaskInfoServiceImpl implements TaskInfoService {
 
	@Autowired
	private TaskInfoMapper taskInfoMapper;
 
	@Override
	public int insertTasks(List<TaskInfo> tasks) {
		return taskInfoMapper.insertTasks(tasks);
	}
 
	@Override
	public int updateTaskControlStatus(Map<String, Object> params) {
		return taskInfoMapper.updateTaskControlStatus(params);
	}
 
	@Override
	public int updateTaskControlTimestamp(Map<String, Object> params) {
		return taskInfoMapper.updateTaskControlTimestamp(params);
	}
 
	@Override
	public Map<String, Object> getTaskControlInfo(Map<String, Object> params) {
		return taskInfoMapper.getTaskControlInfo(params);
	}
 
}
```

上面 @DataSource 注解是定义在类上的，也可以定义在方法上，方法上定义的注解优先级高于定义在类上的注解
方式一与方式二对比：

方式一，需要为每个数据源service层单独建一个包，一个Service只能使用一个数据源

方式二，不需要为每个数据源service层单独建一个包，一个Service可以使用多个数据源


