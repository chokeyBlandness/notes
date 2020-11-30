# junit4和junit5差异

### 包可见性

在JUnit 4里我们的测试方法必须定义为public的访问级别，如果没有定义成public,虽然编译的时候不会提示异常，但是在运行时会提示 “java.lang.Exception: Method testIsBlank() should be public” 如下错误信息。
而在JUnit 5里，我们不再需要将测试类与测试方法定义为public了，默认的包可见的访问级别就可以了。

### 常用注解

| JUnit4       | Junit5      | 说明                                                         |
| ------------ | ----------- | ------------------------------------------------------------ |
| @Test        | @Test       | 指明被注解的方法是一个测试方法，注意JUnit 5的@Test在jupiter-api里 |
| @BeforeClass | @BeforeAll  | 被注解的静态方法会在当前类的所有@Test方法执行前执行一次      |
| @Before      | @BeforeEach | 被注解的方法会在当前类的每个@Test方法执行前执行一次          |
| @AfterClass  | @AfterAll   | 被注解的静态方法会在当前类的所有@Test方法执行后执行一次      |
| @After       | @AfterEach  | 被注解的方法会在当前类的每个@Test方法执行后执行一次          |
| @Ignore      | @Disabled   | 被注解的方法不会被执行，但是在测试报告里会记录为已执行       |

