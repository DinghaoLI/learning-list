## 1. 什么是声明式事务? 

以方法为单位，进行事务控制；抛出异常，事务回滚。

最小的执行单位为方法。决定执行成败是通过是否抛出异常来判断的，抛出异常即执行失败。

### 样例

```java
// ############## DAO

@Repository
public class OrderDao {
	@Autowired
	private JdbcTemplate jdbcTemplate;
	//操作数据的方法
	public void insert(){
		String sql = "insert into `order` (ordertime, ordermoney, orderstatus) values(?,?,?)";
		jdbcTemplate.update(sql,new Date(),20,0);
	}
}


// ############## Service服务层

package com.enjoy.cap11.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.enjoy.cap11.dao.OrderDao;

@Service
public class OrderService {
    @Autowired
	private OrderDao orderDao;
    @Transactional
    public void addOrder(){
    	orderDao.insert();
    	System.out.println("操作完成.........");
    	
    	// 会导致Transaction抛出异常，orderDao.insert()回滚
    	int a = 1/0;
    }
}

// ############## 配置层

/*
 * 
 * InfrastructureAdvisorAutoProxyCreator
 * 1,注册  2，利用后置处理器机制在创建以后，包装对象，返回一个代理对象（增强），代理对象执行方法时，利用拦截器键进行调用 
 * 
 *  AnnotationTransactionAttributeSource：事务增强器要用事务注解的信息，使用这个类解析事务注解
 *  TransactionInterceptor：保存了事务属性信息，事务管理器  MethodInterceptor
 *  当执行目标方法时：
 *     执行拦截器链
 *     事务拦截器：
 *     1.先获取事务相关属性
 *     2.获取PlatformTransactionManager事务管理器，直接到容器中获取platformTransactionManager bean实例
 *  执行目标方法：
 *  
 *     如果异常：completeTransactionAfterThrowing，利用事务管理回滚操作
 *     如果正常：利用 事务管理器，提交事务。
 *  事务管理器：
 *  
 */



@Configuration
@ComponentScan("com.enjoy.cap11")
@EnableTransactionManagement  //开启事务管理功能，对@Transactional起作用  EnableXXXX
public class Cap11MainConfig {
	//创建数据源
	@Bean
	public DataSource dataSource() throws PropertyVetoException{
		//这个c3p0封装了JDBC, dataSource接口的实现
		ComboPooledDataSource dataSource = new ComboPooledDataSource();
		dataSource.setUser("root");
		dataSource.setPassword("root");
		dataSource.setDriverClass("com.mysql.jdbc.Driver");
		dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
		return dataSource;
	}
	
	//jdbcTemplate能简化增删改查的操作
	@Bean
	public JdbcTemplate jdbcTemplate() throws PropertyVetoException{
		return new JdbcTemplate(dataSource());
	}
	//注册事务管理器
	@Bean
	public PlatformTransactionManager platformTransactionManager() throws PropertyVetoException{
		return new DataSourceTransactionManager(dataSource());
	}
}


// ############## 测试

public class Cap11Test {
	@Test
	public void test01(){
		AnnotationConfigApplicationContext app = new AnnotationConfigApplicationContext(Cap11MainConfig.class);
		
		OrderService bean = app.getBean(OrderService.class);
		bean.addOrder();
		app.close();		
	}
}
```


### 源码分析


- 1. @EnableTransactionManagement注解开启事务管理器
- 2. 注册AutoProxyRegistrar和ProxyTransactionManagementConfiguration
- 3. 核心组件InfrastructureAdvisorAutoProxyCreator注册


```java
@EnableTransactionManagement源码分析（与AOP的创建拦截流程一致）：
 1）、@EnableTransactionManagement
 			利用TransactionManagementConfigurationSelector给容器中会导入组件
 			导入两个组件
 			AutoProxyRegistrar
 			ProxyTransactionManagementConfiguration
 2）、AutoProxyRegistrar：
 			给容器中注册一个 InfrastructureAdvisorAutoProxyCreator 组件；基本的动态代理创建器
 			InfrastructureAdvisorAutoProxyCreator：？
 			利用后置处理器机制在对象创建以后，包装对象，返回一个代理对象（增强器），代理对象执行方法利用拦截器链进行调用；
 
 3）、ProxyTransactionManagementConfiguration 做了什么？
 			1、给容器中注册事务增强器；
 				1）、事务增强器要用事务注解的信息，AnnotationTransactionAttributeSource解析事务注解
 				2）、事务拦截器：
 					TransactionInterceptor；保存了事务属性信息，事务管理器；
 					他是一个 MethodInterceptor；
 					在目标方法执行的时候；
 						执行拦截器链；
 						事务拦截器：
 							1）、先获取事务相关的属性
 							2）、再获取PlatformTransactionManager，如果事先没有添加指定任何transactionmanger
 								最终会从容器中按照类型获取一个PlatformTransactionManager；
 							3）、执行目标方法
 								如果异常，获取到事务管理器，利用事务管理回滚操作；
 								如果正常，利用事务管理器，提交事务		
``` 



























