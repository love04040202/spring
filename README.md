# spring
http://blog.csdn.net/qh_java/article/details/51811533  
`
二、使用AOP的方式实现事务的配置

1、这种事务使用也很多，下面我们来看看简单的例子

2、事务配置实例：

（1）、配置文件

<!-- 定义事务管理器 -->
	<bean id="transactionManager"
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource" />
	</bean>
	<!-- 下面使用aop切面的方式来实现 -->
	<tx:advice id="TestAdvice" transaction-manager="transactionManager">
		<!--配置事务传播性，隔离级别以及超时回滚等问题 -->
		<tx:attributes>
			<tx:method name="save*" propagation="REQUIRED" />
			<tx:method name="del*" propagation="REQUIRED" />
			<tx:method name="update*" propagation="REQUIRED" />
			<tx:method name="add*" propagation="REQUIRED" />
			<tx:method name="*" rollback-for="Exception" />
		</tx:attributes>
	</tx:advice>
	<aop:config>
		<!--配置事务切点 -->
		<aop:pointcut id="services"
			expression="execution(* com.website.service.*.*(..))" />
		<aop:advisor pointcut-ref="services" advice-ref="TestAdvice" />
	</aop:config>
上面我们看到了，简单的配置了事务，其中tx:attributes中设置了事务的传播性，隔离级别以及那种问题能进行回滚超时等这些问题，也就是你自己按照业务需求定制一个事务来满足你的业务需求。
注意： 这里注意一下，在tx:method中配置了rollback_for 中配置的Exception 这个是运行时的异常才会回滚不然其他异常是不会回滚的！

@Service("userService")
public class UserService {
	@Autowired
	private UserDao userDao;
	public void saveUser(Map<String, String> map) throws Exception {
		 userDao.saveUser(map);
		 throw new RuntimeException();
		 // throw new Exception ();
		 
	}
}

这里我们看到了，如果抛出的是一个运行时异常则会回滚而如果抛出的不是运行时异常则会不回滚的。
需要注意的地方:

（1）、在spring 配置文件中引入：xmlns:aop="http://www.springframework.org/schema/aop"

 http://www.springframework.org/schema/aop 
                  http://www.springframework.org/schema/aop/spring-aop.xsd
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	 xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	              http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
				  http://www.springframework.org/schema/context
				  http://www.springframework.org/schema/context/spring-context-3.2.xsd
				  http://www.springframework.org/schema/aop 
                  http://www.springframework.org/schema/aop/spring-aop.xsd		
				 http://www.springframework.org/schema/tx 
				 http://www.springframework.org/schema/tx/spring-tx-3.2.xsd">
（2） advice（建议）的命名：由于每个模块都会有自己的Advice，所以在命名上需要作出规范，初步的构想就是模块名+Advice（只是一种命名规范）。

（3） tx:attribute标签所配置的是作为事务的方法的命名类型。

         如<tx:method name="save*" propagation="REQUIRED"/>

        其中*为通配符，即代表以save为开头的所有方法，即表示符合此命名规则的方法作为一个事务。

        propagation="REQUIRED"代表支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。

（4） aop:pointcut标签配置参与事务的类，由于是在Service中进行数据库业务操作，配的应该是包含那些作为事务的方法的Service类。

       首先应该特别注意的是id的命名，同样由于每个模块都有自己事务切面，所以我觉得初步的命名规则因为 all+模块名+ServiceMethod。而且每个模块之间不同之处还在于以下一句：

       expression="execution(* com.test.testAda.test.model.service.*.*(..))"

       其中第一个*代表返回值，第二*代表service下子包，第三个*代表方法名，“（..）”代表方法参数。

（5） aop:advisor标签就是把上面我们所配置的事务管理两部分属性整合起来作为整个事务管理。

（6）注意标红的地方



ok 到这里两种配置方式简单的demo 都有了，下面我们来看看tx:method 中还有那些属性可以配置
下面来看看aop 这种方式来配置的时候我们还能配置那些属性：


<tx:advice id="advice" transaction-manager="txManager">
  <tx:attributes>
    <!-- tx:method的属性:
          * name 是必须的,表示与事务属性关联的方法名(业务方法名),对切入点进行细化。通配符（*）可以用来指定一批关联到相同的事务属性的方法。
                    如：'get*'、'handle*'、'on*Event'等等.
          * propagation  不是必须的 ，默认值是REQUIRED 
                            表示事务传播行为, 包括REQUIRED,SUPPORTS,MANDATORY,REQUIRES_NEW,NOT_SUPPORTED,NEVER,NESTED
          * isolation    不是必须的 默认值DEFAULT 
                            表示事务隔离级别(数据库的隔离级别) 
          * timeout      不是必须的 默认值-1(永不超时)
                            表示事务超时的时间（以秒为单位） 
          
          * read-only    不是必须的 默认值false不是只读的 
                            表示事务是否只读？ 
          
          * rollback-for 不是必须的   
                            表示将被触发进行回滚的 Exception(s)；以逗号分开。
                            如：'com.foo.MyBusinessException,ServletException' 
          
          * no-rollback-for 不是必须的  
                              表示不被触发进行回滚的 Exception(s)；以逗号分开。
                              如：'com.foo.MyBusinessException,ServletException'
                              
                              
        任何 RuntimeException 将触发事务回滚，但是任何 checked Exception 将不触发事务回滚                      
     -->
     <tx:method name="save*" propagation="REQUIRED" isolation="DEFAULT" read-only="false"/>
     <tx:method name="update*" propagation="REQUIRED" isolation="DEFAULT" read-only="false"/>
     <tx:method name="delete*" propagation="REQUIRED" isolation="DEFAULT" read-only="false"/>
     <!-- 其他的方法之只读的 -->
     <tx:method name="*" read-only="true"/>
  </tx:attributes>
</tx:advice>

OK 两种配置方式我们也都简单学习了，两种配置方式有哪些属性要配置我们也了解了，但是，这里只是说了有这两种常用的方法，而没有具体说事务以及事务的配置带来的数据的安全性以及性能的影响，其实事务不是那么简单，具体深入学习希望以后有总结！


下面给大家列出spring事务的几种传播特性：

1. PROPAGATION_REQUIRED: 如果存在一个事务，则支持当前事务。如果没有事务则开启
2. PROPAGATION_SUPPORTS: 如果存在一个事务，支持当前事务。如果没有事务，则非事务的执行
3. PROPAGATION_MANDATORY: 如果已经存在一个事务，支持当前事务。如果没有一个活动的事务，则抛出异常。
4. PROPAGATION_REQUIRES_NEW: 总是开启一个新的事务。如果一个事务已经存在，则将这个存在的事务挂起。
5. PROPAGATION_NOT_SUPPORTED: 总是非事务地执行，并挂起任何存在的事务。
6. PROPAGATION_NEVER: 总是非事务地执行，如果存在一个活动事务，则抛出异常
7. PROPAGATION_NESTED：如果一个活动的事务存在，则运行在一个嵌套的事务中. 如果没有活动事务, 
      则按TransactionDefinition.PROPAGATION_REQUIRED 属性执行
Spring事务的隔离级别
1. ISOLATION_DEFAULT： 这是一个PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别,另外四个与JDBC的隔离级别相对应
2. ISOLATION_READ_UNCOMMITTED： 这是事务最低的隔离级别，它充许令外一个事务可以看到这个事务未提交的数据,这种隔离级别会产生脏读，不可重复读和幻像读。
3. ISOLATION_READ_COMMITTED： 保证一个事务修改的数据提交后才能被另外一个事务读取。另外一个事务不能读取该事务未提交的数据
4. ISOLATION_REPEATABLE_READ： 这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻像读,它除了保证一个事务不能读取另一个事务未提交的数据外，还保证了避免下面的情况产生(不可重复读)。
5. ISOLATION_SERIALIZABLE 这是花费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行,除了防止脏读，不可重复读外，还避免了幻像读。 

`
