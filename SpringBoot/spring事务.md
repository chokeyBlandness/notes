# Spring事务

### 事务传播类型

Spring在`TransactionDefinition`接口中规定了7中类型的事务传播行为，propagation开头。对应`Transactional`注解的propagation属性。

1、**required**。如果没有事务，就新建一个事务，如果已经存在就加入该事务。默认选项。

2、**supports**。该方法在某个事务范围内被调用，则方法成为该事务的一部分。如果方法在该事务范围外被调用，该方法就在没有事务的环境下执行。

3、**mandatory**。该方法只能在一个已经存在的事务中执行，业务方法不能发起自己的事务。如果在没有事务的环境下被调用，抛出异常。

4、**requires_new**。不管是否存在事务，业务方法总会为自己发起一个新的事务。如果方法已经运行在一个事务中，则原有事务会被挂起， 新的事务会被创建，直到方法执行结束，新事务才算结束，原先的事务才会恢复执行。需要使用**jtaTransactionManager**作为事务管理器（独立事务用这个）。

5、**not_supported**。声明方法不需要事务。如果方法没有关联到一个事务，容器不会为它开启事务。如果方法在一个事务中被调用，该事务会被挂起，在方法调用结束后，原先的事务便会恢复执行。需要使用**jtaTransactionManager**作为事务管理器。

6、**nerver**。以非事务方式运行，如果存在事务则抛出异常。

7、**nested**。如果一个活动的事务存在，则运行在一个嵌套的事务中。如果没有活动事务，则按REQUIRED属性执行。它使用了一个单独的事务，这个事务拥有多个可以回滚的保存点。内部事务的回滚不会对外部事务造成影响。它只对**DataSourceTransactionManager**事务管理器起效。

### 事务隔离级别

在`TransactionDefinition`的事务管理器中定义了5个事务隔离级别，isolation开头。

1、**default**。默认使用数据库的隔离级别。

2、**read_uncommitted**。允许其他事务可以看到当前事务未提交的数据，会出现脏读、不可重复读和幻象读。

3、**read_committed**。保证当前事务修改的数据提交后才可被其他事务读取，不会出现脏读，会出现不可重复读和幻象读  一般用这个即可。

4、**repeatable_read**。允许幻像读，但不允许不可重复读和脏读。

5、**serializable**。事务被处理为顺序执行，耗费最大的资源。幻像读、不可重复读和脏读都不允许。

### 内置事务管理器

1、**DataSourceTransactionManager**。位于`org.springframework.jdbc.datasource`包中，数据源事务管理器，提供对单个`javax.sql.DataSource`事务管理，用于Spring JDBC抽象框架或MyBatis框架的事务管理。

2、**JpaTransactionManager**。位于`org.springframework.orm.jpa`包中，提供对单个`javax.persistence.EntityManagerFactory`事务支持，用于集成JPA实现框架时的事务管理。

3、**JtaTransactionManager**。位于`org.springframework.transaction.jta`包中，提供对分布式事务管理的支持，并将事务管理委托给Java EE应用服务器事务管理器。

4、**WebSphereUowTransactionManager**。位于`org.springframework.transaction.jta`包中，Spring提供的对WebSphere 6.0+应用服务器事务管理器的适配器，此适配器用于对应用服务器提供的高级事务的支持。

5、**WebLogicJtaTransactionManager**。位于`org.springframework.transaction.jta`包中，Spring提供的对WebLogic 8.1+应用服务器事务管理器的适配器，此适配器用于对应用服务器提供的高级事务的支持。