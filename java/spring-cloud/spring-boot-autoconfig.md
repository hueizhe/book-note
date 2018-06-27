## A quick peek into auto-configuration

Auto-configuration is one of the most important features of Spring Boot. 

In this section, we will take a quick peek behind the scenes to understand how Spring Boot auto-configuration works.

Most of the Spring Boot auto-configuration magic comes from `spring-boot-autoconfigure-{version}.jar`. When we start any Spring Boot applications, a number of beans get auto-configured. How does this happen?

The following screenshot shows an extract from `spring.factories` from `spring-boot-autoconfigure-{version}.jar` 

We have filtered out some of the configuration in the interest of space:
~~~properties
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer

# Auto Configuration Import Listeners
org.springframework.boot.autoconfigure.AutoConfigurationImportListener=\
org.springframework.boot.autoconfigure.condition.ConditionEvaluationReportAutoConfigurationImportListener

# Auto Configuration Import Filters
org.springframework.boot.autoconfigure.AutoConfigurationImportFilter=\
org.springframework.boot.autoconfigure.condition.OnClassCondition

# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
~~~

The preceding list of auto-configuration classes is run whenever a Spring Boot application is launched. Let's take a quick look at one of them:

`org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration.`

Here's a small snippet:
~~~java
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurerAdapter.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter(DispatcherServletAutoConfiguration.class)
public class WebMvcAutoConfiguration { }
~~~

Some of the important points to note are as follows:

- `@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurerAdapter.class }) `: 
    This auto-configuration is enabled if any of the mentioned classes are in the classpath. When we add a web starter project, we bring in dependencies with all these classes. Hence, this auto-configuration will be enabled.

- `@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)`: 
    This auto-configuration is enabled only if the application does not explicitly declare a bean of the WebMvcConfigurationSupport.class class.
    
- `@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)`: This specifies the precedence of this specific auto-configuration.

Let's look at another small snippet showing one of the methods from the same class:
~~~java
    @Bean
    @ConditionalOnBean(ViewResolver.class)
    @ConditionalOnMissingBean(name = "viewResolver", value = ContentNegotiatingViewResolver.class)
    public ContentNegotiatingViewResolver  viewResolver(BeanFactory beanFactory) {
      ContentNegotiatingViewResolver resolver = new   ContentNegotiatingViewResolver();
      resolver.setContentNegotiationManager (beanFactory.getBean(ContentNegotiationManager.class));
      resolver.setOrder(Ordered.HIGHEST_PRECEDENCE);
      return resolver;
     }
~~~
View resolvers are one of the beans configured by WebMvcAutoConfiguration class. The preceding snippet ensures that if a view resolver is not provided by 

the application, then Spring Boot auto-configures a default view resolver. Here are a few important points to note:

- `@ConditionalOnBean(ViewResolver.class)`: Create this bean if ViewResolver.class is on the classpath
- `@ConditionalOnMissingBean(name = "viewResolver", value = ContentNegotiatingViewResolver.class)`: Create this bean if there are no explicitly declared beans of the name viewResolver and of type ContentNegotiatingViewResolver.class
- The rest of the method is configured in the view resolver

To summarize, all the auto-configuration logic is executed at the start of a Spring Boot application. If a specific class (from a specific dependency or starter project) is available on the classpath, then the auto configuration classes are executed. These auto-configuration classes look at what beans are already configured. Based on the existing beans, they enable the creation of the default beans.