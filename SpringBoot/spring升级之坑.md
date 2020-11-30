# spring升级之坑

#### 问题：

spring2.X升级至3.X后，如使用ViewResolver辅助视图的渲染，会出现`No bean class specified on bean definition`的异常错误。

```
org.springframework.web.util.NestedServletException: Request processing failed; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name '500View': Instantiation of bean failed; nested exception is java.lang.IllegalStateException: No bean class specified on bean definition

```

#### 解决方法：

在views.properties配置文件中将class字段用小括号括起来。

```properties
#500View.class=org.springframework.web.servlet.view.freemarker.FreeMarkerView
500View.(class)=org.springframework.web.servlet.view.freemarker.FreeMarkerView
```

https://blog.csdn.net/iteye_3691/article/details/81896910

