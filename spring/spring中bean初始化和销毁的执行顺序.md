# spring中bean的生命周期

bean生命周期

![](..\pictures\bean的生命周期.png)

通过代码直接测试bean初始化和销毁的执行顺序。spring版本为5.2.5.RELEASE。

```java
public class TestBean implements InitializingBean, DisposableBean {

    public TestBean() {
        System.out.println("a");
    }

    public void initMethod() {
        System.out.println("b");
    }

    @PostConstruct
    public void postConstruct() {
        System.out.println("c");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("d");
    }

    public void destroyMethod() {
        System.out.println("e");
    }

    @PreDestroy
    public void preDestroy() {
        System.out.println("f");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("g");
    }
}
```

通过注解直接创建bean

```java
 @Bean(initMethod = "initMethod", destroyMethod = "destroyMethod")
    public TestBean testBean() {
        return new TestBean();
    }
```

### 结果

a、c、d、b、f、g、e。

一开始是类的初始化；

然后是`@PostConstruct`和`@PreDestroy`注解的方法；

接着是`InitializingBean`和`Disposable`接口的实现方法；

最后是定义bean是指定的`initMethod`和`destroyMethod`参数指定的方法。
