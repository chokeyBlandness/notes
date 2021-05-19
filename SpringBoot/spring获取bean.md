# spring获取bean的方式

### 1、在初始化时保存ApplicationContext对象

这种方式适用于采用Spring框架的独立应用程序，需要程序通过配置文件手工初始化Spring的情况。

~~~java
ApplicationContext ac = new FileSystemXmlApplicationContext("applicationContext.xml"); 
ac.getBean("beanId");
~~~

### 2、通过spring提供的工具类获取ApplicationContext对象

这种方式适合于采用Spring框架的B/S系统，通过ServletContext对象获取ApplicationContext对象，然后在通过它获取需要的类实例。上面两个工具方式的区别是，前者在获取失败时抛出异常，后者返回null。

~~~java
ApplicationContext ac1 = WebApplicationContextUtils
.getRequiredWebApplicationContext(ServletContext sc); 
ApplicationContext ac2 = WebApplicationContextUtils
.getWebApplicationContext(ServletContext sc); 
ac1.getBean("beanId"); 
ac2.getBean("beanId");
~~~

### 3、继承抽象类ApplicationObjectSupport

抽象类ApplicationObjectSupport提供getApplicationContext()方法，可以方便的获取到ApplicationContext。
Spring初始化时，会通过该抽象类的setApplicationContext(ApplicationContext context)方法将ApplicationContext 对象注入。

### 4、实现接口ApplicationContextAware

实现该接口的setApplicationContext(ApplicationContext context)方法，并保存ApplicationContext 对象。Spring初始化时，会通过该方法将ApplicationContext对象注入。需要将此类配置为bean，否则无法获取到ApplicationContext对象。

~~~java
public class SpringContextUtil implements ApplicationContextAware { 
 // Spring应用上下文环境 
 private static ApplicationContext applicationContext; 
 /** 
 * 实现ApplicationContextAware接口的回调方法，设置上下文环境 
 * 
 * @param applicationContext 
 */ 
 public void setApplicationContext(ApplicationContext applicationContext) { 
 SpringContextUtil.applicationContext = applicationContext; 
 } 
 /** 
 * @return ApplicationContext 
 */ 
 public static ApplicationContext getApplicationContext() { 
 return applicationContext; 
 } 
 /** 
 * 获取对象 
 * 
 * @param name 
 * @return Object
 * @throws BeansException 
 */ 
 public static Object getBean(String name) throws BeansException { 
 return applicationContext.getBean(name); 
 } 
}
~~~

### 5、通过Spring提供的ContextLoader

提供一种不依赖于servlet,不需要注入的方式。但是需要注意一点，在服务器启动时，Spring容器初始化时，不能通过以下方法获取Spring 容器，细节可以查看spring源码org.springframework.web.context.ContextLoader。

~~~java
WebApplicationContext wac = ContextLoader.getCurrentWebApplicationContext();
wac.getBean(beanID);
~~~

