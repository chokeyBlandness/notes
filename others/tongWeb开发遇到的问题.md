# TongWeb开发时遇到的问题

### 1、NoSuchMethodError:ConfigurationState异常

**问题描述**：在idea中使用TongWeb插件新建tongweb server运行时，出现`java.lang.NoSuchMethodError:javax.validation.spi.ConfigurationState.getValueExtractors()Ljava/util/Set`异常。

**原因**：TongWeb目录lib下带了低版本的validation-api.jar，不存在此方法。

**解决方法**：把TongWeb目录lib下的validation-api.jar删除，替换成高版本的jar包。

### 2、一个容器中部署多个应用异常

```shell
Caused by: org.springframework.jmx.export.UnableToRegisterMBeanException: Unable to register MBean [org.springframework.cloud.context.environment.EnvironmentManager@3ebe27fc] with key 'environmentManager'; nested exception is javax.management.InstanceAlreadyExistsException: org.springframework.cloud.context.environment:name=environmentManager,type=EnvironmentManager
```

**原因**：spring.jmx默认开启。

**解决方法1**：配置`spring.jmx.enabled=false`。

**解决方法2**：在application中各自配置`spring.jmx.default-domain=project1`和`spring.jmx.default-domain=project2`，以确保domain不一样。