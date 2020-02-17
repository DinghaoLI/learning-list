# 1. Spring基础

## 1.1 简介

Spring是一种开源轻量级框架，是为了解决企业应用程序开发复杂性而创建的，Spring致力于解决JavaEE的各层解决方案，而不仅仅于某一层的方案。

2003年2月Spring框架正式称为一道开源项目，Spring致力于J2EE应用的各种解决方案，而不仅仅专注于某一层解决方案。可以说Spring是企业应用开发的“一站式”选择， Spring贯穿于表现层、业务层、持久层，然而Spring并不想取代那些已经有的框架，而是以高度的开放性，与这些已有的框架进行整合。

### Spring的目标：

- 1. 让现有的技术更容易使用，
- 2. 促进良好的编程习惯。

Spring是一个全面的解决方案，它坚持一个原则：不从新造轮子。已经有较好解决方案的领域，Spring绝不重复性实现，比如：对象持久化和OR映射，Spring只对现有的JDBC，Hibernate等技术提供支持，使之更容易使用，而不做重复的实现。Spring框架有很多特性，这些特性由7个定义良好的模块构成。

### Spring体系结构

- 1. Spring Core：即，Spring核心，它是框架最基础的部分，提供IOC和依赖注入特性
- 2. Spring Context：即，Spring上下文容器，它是BeanFactory功能加强的一个子接口
- 3. Spring Web：它提供Web应用开发的支持
- 4. Spring MVC：它针对Web应用中MVC思想的实现
- 5. Spring DAO：提供对JDBC抽象层，简化了JDBC编码，同时，编码更具有健壮性。
- 6. Spring ORM：它支持用于流行的ORM框架的整合，比如：Spring + Hibernate、Spring + iBatis、Spring + JDO的整合等等。
- 7. Spring AOP：AOP即，面向切片编程，它提供了与AOP联盟兼容的编程实现

### Spring常用组件

![](./img/spring_component.png)

## 1.2 Spring初体验

![](./img/spring_test_1.png)
![](./img/spring_test_2.png)

## 1.3 @ComponentScan扫描规则

### 配置类例子以及扫描规则解释：

```java
package com.enjoy.cap2.config;

// import...

@Configuration

// ######### 1
@ComponentScan(value="com.enjoy.cap2")
// 扫描com.enjoy.cap2下所有的组件/Bean，包括Configuration等、Cap2MainConfig自己

// ######### 2
@ComponentScan(value="com.enjoy.cap2",includeFilters={
		@Filter(type=FilterType.ANNOTATION,classes={Controller.class}),
		@Filter(type=FilterType.ASSIGNABLE_TYPE,classes={OrderDao.class}),
}, useDefaultFilters = false)
//excludeFilters = Filter[] 指定扫描的时候按照什么规则排除那些组件，exclude的时候需要使用useDefaultFilters = true
//includeFilters = Filter[] 指定扫描的时候只需要包含哪些组件
//useDefaultFilters = false 默认是true，扫描所有组件，使用includeFilters要改成false。useDefaultFilters会把所有@Component全部扫描。

//－－－－扫描规则如下
//FilterType.ANNOTATION：按照注解
//FilterType.ASSIGNABLE_TYPE：按照给定的类型（类的名字）；比如按BookService类型
//FilterType.ASPECTJ：使用ASPECTJ表达式
//FilterType.REGEX：使用正则指定
//FilterType.CUSTOM：使用自定义规则，自已写类，实现TypeFilter接口

// ######## 3
@ComponentScan(value="com.enjoy.cap2", includeFilters={
		@Filter(type=FilterType.CUSTOM, classes={JamesTypeFilter.class})
}, useDefaultFilters=false)
// 扫描com.enjoy.cap2下所有的组件/Bean，使用自定义的过滤器
// useDefaultFilters默认是true,扫描所有组件，要改成false,使用自定义扫描范围

public class Cap2MainConfig {
	// 给容器中注册一-个bean,类型为返回值的类型，id默认为方法名person01
	@Bean
	public Person person01(){
		return new Person("james",20);
	}
}
```

### 自定义扫描类：

```java
package com.enjoy.cap2.config;

// import...

public class JamesTypeFilter implements TypeFilter{
	private ClassMetadata classMetadata;

	/*
	 * MetadataReader: 读取到的当前正在扫描的类的信息
	 * MetadataReaderFactory: 可以获取到其他任何类信息的
	 */
	@Override
	public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)
			throws IOException {
		// 获取当前类注解的信息
		AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
		// 获取当前正在扫描的类的类信息
		classMetadata = metadataReader.getClassMetadata();
		// 获取当前类资源(类的路径)
		Resource resource = metadataReader.getResource();
		
		String className = classMetadata.getClassName();
		System.out.println("----->"+className);
		if(className.contains("Controller")){
			//当类名包含"Controller"，则匹配成功
			return true;
		}
		return false;
	}
}
```

### @ComponentScans：可放入多个@ComponentScan

[What is the difference between @ComponentScans and @ComponentScan?](https://stackoverflow.com/questions/52421944/what-is-the-difference-between-componentscans-and-componentscan)

```java
@Configuration
@ComponentScans({ 
    @ComponentScan(basePackages = "com.baeldung.annotations"), 
    @ComponentScan(basePackageClasses = VehicleFactoryConfig.class)
})
class VehicleFactoryConfig {}
```

## 1.4 @Scope扫描规则

### IOC容器的单实例和多实例：

```java
package com.enjoy.cap3.config;

// import...

@Configuration
public class Cap3MainConfig {
	// 给容器中注册一个bean，类型为返回值的类型，默认是单实例
	/*
	 * prototype：多实例: IOC容器 启动的时候, IOC容器启动并不会去调用方法创建对象，
	 * 而是每次获取的时候才会调用方法创建对象
	 *
	 * singleton：单实例(默认)：IOC容器启动的时候会调用方法创建对象并放到IOC容器中，
	 * 以后每次获取的就是直接从容器（实例存储在一个大的Map中）中拿（Map.get）的同一个bean
	 *
	 * request：主要针对web应用，递交一次请求创建一个实例
	 *
	 * session：同一个session创建一个实例
	 */
	@Scope("prototype")
	@Bean
	public Person person(){
		return new Person("james",20);
	}
}
```

### IOC容器的单实例和多实例的测试：

```java
// import...

public class Cap3Test {
	@Test
	public void test01(){
		AnnotationConfigApplicationContext app = new AnnotationConfigApplicationContext(Cap3MainConfig.class);

		Object bean1 = app.getBean("person");
		Object bean2 = app.getBean("person");
		System.out.println(bean1 == bean2);
		// Person为prototype时，(bean1 == bean2) => false
		// Person为singleton时，(bean1 == bean2) => true

	}
}

```

## 1.5 @Lazy懒加载/延迟加载

### @Lazy的使用

```java
package com.enjoy.cap4.config;

// import ...

@Configuration
public class Cap4MainConfig {
	// 给容器中注册一个bean,类型为返回值的类型，默认是单实例
	/*
	 * 懒加载主要针对单实例bean，默认在容器启动的时候创建对象
	 * 使用懒加载时，容器启动时候不创建对象，仅当第一次使用（获取）bean的时候才创建被初始化
	 */
	@Lazy
	@Bean
	public Person person(){
		// 在Bean创建对象创建前打印
		System.out.println("给容器中添加person.......");
		return new Person("james",20);
	}
}
```

### 测试

```java
// import ...

public class Cap4Test {
	@Test
	public void test01(){
		AnnotationConfigApplicationContext app = new AnnotationConfigApplicationContext(Cap4MainConfig.class);
		
		System.out.println("IOC容器创建完成........");
		app.getBean("person");
		// 如果Person加@Lazy，输出顺序为：
		// IOC容器创建完成........
		// 给容器中添加person.......
		//
		// 如果Person不加@Lazy，输出顺序为:
		// 给容器中添加person.......
		// IOC容器创建完成........
	}
}
```

## 1.6 @Conditional条件注册bean

将IOC容器注册bean时，当操作系统为WINDOWS时,注册Lison实例；当操作系统为LINUX时，注册James实例，此时要用得@Conditional注解进行定制化条件选择注册bean。


```java
package com.enjoy.cap5.config;

// import ...

@Configuration
public class Cap5MainConfig {
	@Bean("person")
	public Person person(){
		System.out.println("给容器中添加person.......");
		return new Person("person",20);
	}
	
	@Conditional(WinCondition.class)
	@Bean("lison")
	public Person lison(){
		System.out.println("给容器中添加lison.......");
		return new Person("Lison",58);
	}

	@Conditional(LinCondition.class)
	@Bean("james")//bean在容器中的ID为james, IOC容器MAP,  map.put("id",value)
	public Person james(){
		System.out.println("给容器中添加james.......");
		return new Person("james",20);
	}
}
```

### Linux和Windows的条件类

#### Tips：

- FactoryBean：注册机制，可以把Java实例Bean通过FactoryBean注入到容器中
- BeanFactory：从IOC容器获取实例化后的bean

```java
package com.enjoy.cap5.config;

// import ...

public class LinCondition implements Condition{
	/*
	*ConditionContext: 判断条件可以使用的上下文(环境)
	*AnnotatedTypeMetadata: 注解的信息
	*
	*/
	@Override
	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
		// 是否为LINUX系统？
		// 能获取到IOC容器正在使用的beanFactory
		ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
		// 获取当前环境变量(包括我们操作系统是WIN还是LINUX??)
		Environment environment = context.getEnvironment();
		String os_name = environment.getProperty("os.name");
		if(os_name.contains("linux")){
			return true;
		}
		return false;
	}
}
```

### 测试

```java
// import ...

public class Cap5Test {
	// 如果操作系统是WINDOWS系统，给I0C容器注册lison,bean
	// 如果操作系統是LINUX系統，IOC容器注册james的bean
	// 新建WinCondition类，用来指定Lison的实例bean
	// 新建LinuxCondition.java类，用来指定James的实例bean
	// 如何测试Linux: run as configuration -> VM Argment -> -Dos.name=linux
    // 条件注入
	@Test
	public void test01(){
		// 声明IOC容器
		AnnotationConfigApplicationContext app = new AnnotationConfigApplicationContext(Cap5MainConfig.class);
		System.out.println("IOC容器创建完成........");

		// 拿到容器中所有Person定义，看看容器中有多少bean
		String[] beanNamesForType = app.getBeanNamesForType(Person.class);
		for(String name : beanNamesForType) {
			System.out.println(name);
		}
		// 测试：-Dos.name=linux
		// output:
		// 给容器中添加person.......
		// 给容器中添加james.......
		// IOC容器创建完成........
		// person
		// james

		// 把所有对象和ID放到Map里
		Map<String, Person> personBean = app.getBeansOfType(Person.class);
		System.out.println(personBean);
		// output:
		// {person=Person [name=person, age=20], james=Person [name=james, age=20]}
	}
}
```

## 1.7 @Import注册bean



```java
package com.enjoy.cap6.config;

// import ...

@Configuration
@Import(value = { Dog.class,Cat.class, JamesImportSelector.class,JamesImportBeanDefinitionRegistrar.class })
public class Cap6MainConfig {
	/*
	 * 给容器中注册组件的方式
	 * 1,@Bean: [导入第三方的类或包的组件],比如Person为第三方的类, 需要在我们的IOC容器中使用
	 * 2,包扫描+组件的标注注解(@ComponentScan:  @Controller, @Service  @Reponsitory  @Componet),一般是针对我们自己写的类
	 * 3,@Import:[快速给容器导入一个组件] 注意:@Bean有点简单
	 *      a,@Import(要导入到容器中的组件):容器会自动注册这个组件,bean 的 id为全类名
	 *      b,ImportSelector:是一个接口,返回需要导入到容器的组件的全类名数组
	 *      c,ImportBeanDefinitionRegistrar:可以手动添加组件到IOC容器, 所有Bean的注册可以使用BeanDifinitionRegistry
	 *          写JamesImportBeanDefinitionRegistrar实现ImportBeanDefinitionRegistrar接口即可
	 * 4,使用Spring提供的FactoryBean(工厂bean)进行注册
	 *     
	 *   
	 */
	//容器启动时初始化person的bean实例
	@Bean("person")
	public Person person(){
		return new Person("james",20);
	}
	@Bean
	public JamesFactoryBean jamesFactoryBean(){
		return new JamesFactoryBean();
	}
}

```

### import ImportSelector例子

```java
package com.enjoy.cap6.config;

// import ...

public class JamesImportSelector implements ImportSelector{
	@Override
	public String[] selectImports(AnnotationMetadata importingClassMetadata){
		// 返回全类名的bean
		return new String[]{"com.enjoy.cap6.bean.Fish","com.enjoy.cap6.bean.Tiger"};
//		return new String[]{};

	}
}
```

### import ImportBeanDefinitionRegistrar例子

```java
package com.enjoy.cap6.config;

// import ...

public class JamesImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

	/*
	* AnnotationMetadata:当前类的注解信息
	* BeanDefinitionRegistry:BeanDefinition注册类
	*    把所有需要添加到容器中的bean加入;
	*    @Scope
	*/
	@Override
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
		boolean bean1 = registry.containsBeanDefinition("com.enjoy.cap6.bean.Dog");
		boolean bean2 = registry.containsBeanDefinition("com.enjoy.cap6.bean.Cat");
		// 如果Dog和Cat同时存在于我们IOC容器中，那么创建Pig类，并加入到IOC容器
		if(bean1 && bean2){
			// 对于我们要注册的bean, 给bean进行封装,
			RootBeanDefinition beanDefinition = new RootBeanDefinition(Pig.class);
			registry.registerBeanDefinition("pig", beanDefinition);
		}
	}

}

```

### import FactoryBean例子

#### Tips：

- FactoryBean：注册机制，可以把Java实例Bean通过FactoryBean注入到容器中
- BeanFactory：从IOC容器获取实例化后的bean

```java
package com.enjoy.cap6.config;

// import ...

// 创建一个Spring定义的工厂bean
public class JamesFactoryBean implements FactoryBean<Monkey>{
	// 返回一个Monkey对象
	@Override
	public Monkey getObject() throws Exception {
		return new Monkey();
	}

	@Override
	public Class<?> getObjectType() {
		return Monkey.class;
	}

	// 是否为单例？
	// true：这个bean是单实例，在容器只会存一份
	// false：每次获取都会创建一个新的bean
	// 怎么创建？就是调用getObject
	@Override
	public boolean isSingleton() {
		return true;
	}
}
```

**将JamesFactoryBean包装成Bean**：

```java
// import ...

@Configuration
@Import(value = { Dog.class, Cat.class, JamesImportSelector.class,
		JamesImportBeanDefinitionRegistrar.class })
public class Cap6MainConfig {
	// ...
	@Bean
	public JamesFactoryBean jamesFactoryBean(){
		return new JamesFactoryBean();
	}
}

```


#### 测试

```java
// import ...

public class Cap6Test {
	@Test
	public void test01(){
		AnnotationConfigApplicationContext app = new AnnotationConfigApplicationContext(Cap6MainConfig.class);
		System.out.println("IOC容器创建完成........");
		
		Object bean1 = app.getBean("&jamesFactoryBean"); 
		// 获取到的bean为jamesFactoryBean对象，源码中有判断"&"的操作，如果有“&”就返回bean本身，
		// 如果没有“&”就调用FactoryBean.getObject()返回bean
		// 由此可知，对于一般的bean，IOC容器会为其自动生成一个FactoryBean
		
		Object bean2 = app.getBean("jamesFactoryBean"); 
		// 实际是获取getObject创建的对象，即Monkey对象，并不是jamesFactoryBean对象

		System.out.println("bean1的类型="+bean1.getClass());
		// output: bean1的类型=class com.enjoy.cap6.config.JamesFactoryBean
		System.out.println("bean2的类型="+bean2.getClass());
		// output: bean2的类型=class com.enjoy.cap6.bean.Monkey
		System.out.println(bean1 == bean2);
		// false
		
		String[] beanDefinitionNames = app.getBeanDefinitionNames();
		for(String name:beanDefinitionNames){
			System.out.println(name);
		}
	}
}

```


## 1.8 Bean的生命周期

### 1.8.1 指定初始化init-method方法和销毁destory-method方法（单实例）

bean的生命周期：指bean创建、初始化、销毁的过程，bean的生命周期是由容器进行管理的。我们可以自定义bean的初始化和销毁方法：容器在bean进行到当前生命周期的时候, 来调用自定义的初始化和销毁方法。

```java
public class Bike {
	public Bike(){
		System.out.println("Bike constructor..............");
	}
	public void init(){
		System.out.println("Bike .....init.....");
	}
	public void destory(){
		System.out.println("Bike.....destory");
	}
}
```

#### 给Bike加上初始化和销毁方法：

```java
// import ...

@ComponentScan("com.enjoy.cap7.bean")
@Configuration
public class Cap7MainConfigOfLifeCycle {
	@Bean("person")
	public Person person(){
		System.out.println("给容器中添加person.......");
		return new Person("person",20);
	}
	@Bean(initMethod="init", destroyMethod="destory")
	public Bike bike(){
		return new Bike();
	}
}
```

#### 测试：

```java
// import ...

public class Cap7Test {
	@Test
	public void test01(){
		AnnotationConfigApplicationContext app = new AnnotationConfigApplicationContext(Cap7MainConfigOfLifeCycle.class);
		
		System.out.println("IOC容器创建完成........");

		// 什么时候被销毁呢?单实例模式下，当关闭容器的时候app.close();
		// 当声明bean @scope("prototype")多实例bean时，只有在获取bean的时候才初始化
		// anno.getBean("bike"); => 此时才会调用初始化
		// 针对多实例，IOC容器不负责销毁的，是由自己来控制，容器关闭不会调用销毁方法
		app.close();
		// 容器的销毁：
		// 	public void destroySingletons() {
		//      ...
		//		this.containedBeanMap.clear();
		//		this.dependentBeanMap.clear();
		//		this.dependenciesForBeanMap.clear();
	}
}

// output:
// 
// 给容器中添加person.......
// Bike constructor..............
// Bike .....init.....
// IOC容器创建完成........
// Bike.....destory
```

### 1.8.2 Bean实现InitializingBean和DisposableBean接口

- InitializingBean（定义初始化逻辑,可点进去看此类）：afterPropertiesSet()方法：**当beanFactory创建好对象，且把bean所有属性设置好之后，会调这个方法，相当于初始化方法**。

- DisposableBean（定义销毁逻辑,可点进去看此类）：destory()方法，当bean销毁时，**会把单实例bean进行销毁**。


```java
// import ...

@Component
public class Train implements InitializingBean, DisposableBean {

	public Train(){
		System.out.println("Train......constructor............");
	}

	// 在容器创建时调用：当bean属性赋值和初始化完成时调用
	// Invoked by a BeanFactory after it has set all bean properties supplied
	// (and satisfied BeanFactoryAware and ApplicationContextAware).
	@Override
	public void afterPropertiesSet() throws Exception {
		System.out.println("Train.......afterPropertiesSet()...");

	}

	// 在容器关闭时调用：当bean销毁时，调用此方法
	// Invoked by a BeanFactory on destruction of a singleton.
	@Override
	public void destroy() throws Exception {
		System.out.println("Train......destory......");
		//logger.error
	}
}
```

#### 测试：

```java
// import ...

public class Cap7Test {
	@Test
	public void test01(){
		AnnotationConfigApplicationContext app = new AnnotationConfigApplicationContext(Cap7MainConfigOfLifeCycle.class);
		
		System.out.println("IOC容器创建完成........");

		// 什么时候被销毁呢?单实例模式下，当关闭容器的时候app.close();
		// 当声明bean @scope("prototype")多实例bean时，只有在获取bean的时候才初始化
		// anno.getBean("bike"); => 此时才会调用初始化
		// 针对多实例，IOC容器不负责销毁的，是由自己来控制，容器关闭不会调用销毁方法
		app.close();
	}
}

// output:
// 
// Train......constructor............
// Train.......afterPropertiesSet()...
// IOC容器创建完成........
// Train......destory......
```

### 1.8.3 使用JSR250规则定义的（java规范）两个注解来实现

- @PostConstruct: 在Bean创建完成,且属于赋值完成后进行初始化,属于JDK规范的注解
- @PreDestroy: 在bean将被移除之前进行通知, 在容器销毁之前进行清理工作

```java
// import ...

@Component
public class Jeep {
	public Jeep(){
		System.out.println("Jeep.....constructor........");
	}

	// 对象创建并赋值之后调用
	@PostConstruct
	public void init(){
		System.out.println("Jeep.....@PostConstruct........");
	}
	
	// 容器移除对象之前回调通知，销毁bean
	@PreDestroy
	public void destory(){
		System.out.println("Jeep.....@PreDestroy......");
	}
}
```

#### 测试

```java
// import ...

public class Cap7Test {
	@Test
	public void test01(){
		AnnotationConfigApplicationContext app = new AnnotationConfigApplicationContext(Cap7MainConfigOfLifeCycle.class);
		
		System.out.println("IOC容器创建完成........");

		// 什么时候被销毁呢?单实例模式下，当关闭容器的时候app.close();
		// 当声明bean @scope("prototype")多实例bean时，只有在获取bean的时候才初始化
		// anno.getBean("bike"); => 此时才会调用初始化
		// 针对多实例，IOC容器不负责销毁的，是由自己来控制，容器关闭不会调用销毁方法
		app.close();
	}
}

// output:
//
// Jeep.....constructor........
// Jeep.....@PostConstruct........
// IOC容器创建完成........
// Jeep.....@PreDestroy......

```
































