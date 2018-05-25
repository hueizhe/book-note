## [Spring事务配置解惑](http://ifeve.com/spring%E4%BA%8B%E5%8A%A1%E9%85%8D%E7%BD%AE%E8%A7%A3%E6%83%91/)

### 一、项目中spring+mybaits xml配置解析

一般我们会在datasource.xml中进行如下配置，但是其中每个配置项原理和用途是什么，并不是那么清楚，如果不清楚的话，在使用时候就很有可能会遇到坑，所以下面对这些配置项进行一一解说
~~~xml

<!--（1）配置数据源 -->
 <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
        <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver" />
        <property name="url" value="${db_url}" />
        <property name="username" value="$db_user}" />
        <property name="password" value="${db_passwd}" />
        <property name="maxWait" value="${db_maxWait}" />
        <property name="maxActive" value="28" /> 
        <property name="initialSize" value="2" />
        <property name="minIdle" value="0" />
        <property name="timeBetweenEvictionRunsMillis" value="db_time" />
    </bean>

<!--（2）创建sqlSessionFactory-->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="mapperLocations" value="classpath*:com/**/mapper/*Mapper*.xml" /> 
        <property name="dataSource" ref="dataSource" />
        <property name="typeAliasesPackage" value="com.test.***.dal" />
</bean>

<!--（3）配置扫描器，扫描指定路径的mapper生成数据库操作代理类-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="annotationClass" value="javax.annotation.Resource"></property>
        <property name="basePackage" value="com.test.***.dal.***.mapper" />
        <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>

<!--（4）配置事务管理器-->
<bean id="transactionManager"
    class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
<property name="dataSource" ref="dataSource" />
</bean>

<!--（5）声明使用注解式事务-->
<tx:annotation-driven transaction-manager="transactionManager" />
<!-- 6）注册各种beanfactory处理器-->
<context:annotation-config />    

<!--（7）该配置创建了一个TransactionInterceptor的bean，作为事务切面的执行方法-->
 <tx:advice id="defaultTxAdvice">
    <tx:attributes>
        <tx:method name="*" rollback-for="Exception" />
    </tx:attributes>
</tx:advice>

<!--（8）该配置创建了一个DefaultBeanFactoryPointcutAdvisor的bean，该bean是一个advisor,里面包含了pointcut和advice.前者说明切面加在哪里，后者是执行逻辑。此处可以配多个advisor -->
<aop:config>
    <aop:pointcut id="myCut" expression="(execution(* *..*BoImpl.*(..))) "/>
    <aop:advisor pointcut-ref="myCut" advice-ref="defaultTxAdvice" />
</aop:config>
~~~
### 1.1 数据源配置
(1）是数据源配置，这个没啥好说的。

### 1.2 配置SqlSessionFactory

(2) 作用是根据配置创建一个SqlSessionFactory，看下SqlSessionFactoryBean的代码知道它实现了FactoryBean和InitializingBean类，由于实现了InitializingBean，所以自然它的afterPropertiesSet方法，由于实现了FactoryBean类，所以自然会有getObject方法。下面看下时序图：
![](http://upload-images.jianshu.io/upload_images/5879294-82daa1ef0d7d2a43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
从时序图可知，SqlSessionFactoryBean类主要是通过属性配置创建SqlSessionFactory实例，具体是解析配置中所有的mapper文件放到configuration,然后作为构造函数参数实例化一个DefaultSqlSessionFactory作为SqlSessionFactory。

### 1.3 配置扫描器，扫描指定路径的mapper生成数据库操作代理类

MapperScannerConfigurer 实现了 BeanDefinitionRegistryPostProcessor, InitializingBean, ApplicationContextAware, BeanNameAware接口，所以会重写一下方法：
~~~java
1.3.1
//在bean注册到ioc后创建实例前修改bean定义和新增bean注册，这个是在context的refresh方法调用
void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException;
1.3.2
//在bean注册到ioc后创建实例前修改bean定义或者属性值
void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
1.3.3
//set属性设置后调用
void afterPropertiesSet() throws Exception;
1.3.4
//获取IOC容器上下文，在context的prepareBeanFactory中调用
void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
1.3.5
//获取bean在ioc容器中名字，在context的prepareBeanFactory中调用
void setBeanName(String name);
~~~

先上个扫描mapper生成代理类并注册到ioc时序图：
![](http://upload-images.jianshu.io/upload_images/5879294-11c59e044003898d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

首先MapperScannerConfigurer实现的afterPropertiesSet方法用来确保属性basePackage不为空

~~~java
public void afterPropertiesSet() throws Exception {
    notNull(this.basePackage, "Property 'basePackage' is required");
  }
~~~
postProcessBeanFactory里面啥都没做，setBeanName获取了bean的名字，setApplicationContext里面获取了ioc上下文。下面看重要的方法postProcessBeanDefinitionRegistry，由于mybais是运行时候才通过解析mapper文件生成代理类注入到ioc，所以postProcessBeanDefinitionRegistry正好可以干这个事情。

