# Spring的扩展组件原理及IOC源码讲解

## 1. BeanFactory的两个重要后置处理器

### 1.1 扩展原理 － BeanFactoryPostProcessor

BeanFactoryPostProcessor：beanFactory的后置处理器；

作用如下： 

- 1. 在BeanFactory标准初始化之后调用，来定制和修改BeanFactory的内容；
- 2. 所有的bean定义已经保存加载到beanFactory，但是bean的实例还未创建

注意：之前也讲过BeanPostProcessor，它是bean后置处理器，bean创建对象初始化前后进行拦截工作的

#### 样例

```java

// ############### 实现类

@Component
public class JamesBeanFactoryPostProcessor implements BeanFactoryPostProcessor{

	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
		System.out.println("JamesBeanFactoryPostProcessor.....调到了BeanFactoryPostProcessor.postProcessBeanFactory()");
		//所有bean的定义，已经加载到beanFactory, 但是bean实例还没创建
		int count = beanFactory.getBeanDefinitionCount();
		String[] beanDefName = beanFactory.getBeanDefinitionNames();
		System.out.println("当前BeanFactory中有"+count+"个Bean");
		System.out.println(Arrays.asList(beanDefName));

	}

}



// ############### 配置类

@Configuration
@ComponentScan("com.enjoy.cap12.processor")
public class Cap12MainConfig {
	@Bean
	public Moon getMoon(){
		return new Moon();
	}
}


// ############### 测试类

public class Cap12Test {
	@Test
	public void test01(){
		AnnotationConfigApplicationContext app = new AnnotationConfigApplicationContext(Cap12MainConfig.class);
		
		app.close();
	
		
	}
}

// output:

// JamesBeanFactoryPostProcessor.....调到了BeanFactoryPostProcessor.postProcessBeanFactory()
// 当前BeanFactory中有9个Bean
// [org.springframework.context.annotation.internalConfigurationAnnotationProcessor, org.springframework.context.annotation.internalAutowiredAnnotationProcessor, org.springframework.context.annotation.internalRequiredAnnotationProcessor, org.springframework.context.annotation.internalCommonAnnotationProcessor, org.springframework.context.event.internalEventListenerProcessor, org.springframework.context.event.internalEventListenerFactory, cap12MainConfig, jamesBeanFactoryPostProcessor, getMoon]
// Moon constructor........
```

不难发现，在“Moon constructor”创建之前，当前9个bean已被加载到beanFactory中了。

跟踪debug栈,不难发现以下步骤如下：

- 1. ioc容器创建对象
- 2. invokeBeanFactoryPostProcessors(beanFactory); 如何找到所有的BeanFactoryPostProcessor并执行他们的方法；
  	- a. 直接在BeanFactory中找到所有类型是BeanFactoryPostProcessor的组件，并执行他们的方法
	- b. 在初始化创建其他组件前面执行



### 1.2 扩展原理 － BeanDefinitionRegistryPostProcessor

postProcessBeanDefinitionRegistry()；在所有bean定义信息将要被加载，bean实例还未创建的；

操作步骤：新建JamesBeanDefinitionRegistryPostProcessor

注意：可使用BeanDefinitionBuilder构建器来创建bean定义信息

新增：

```java

@Component
public class JamesBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor{

	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
		System.out.println("JamesBeanDefinitionProcessor..postProcessBeanFactory(),Bean的数量"+beanFactory.getBeanDefinitionCount());
	}

	public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
		System.out.println("JamesBeanDefinition.postProcessBeanDefinitionRegistry()...bean的数量"+registry.getBeanDefinitionCount());
		//RootBeanDefinition rbd = new RootBeanDefinition(Moon.class);
		AbstractBeanDefinition rbd = BeanDefinitionBuilder.rootBeanDefinition(Moon.class).getBeanDefinition();
		registry.registerBeanDefinition("hello", rbd);
	}
}

// output
// JamesBeanDefinition.postProcessBeanDefinitionRegistry()...bean的数量10 => 先执行bean定义相关处理器
// JamesBeanDefinitionProcessor..postProcessBeanFactory(),Bean的数量11 => 再执行BeanFactoryPostProcessor
// JamesBeanFactoryPostProcessor.....调到了BeanFactoryPostProcessor.postProcessBeanFactory()
// 当前BeanFactory中有11个Bean
// [org.springframework.context.annotation.internalConfigurationAnnotationProcessor, org.springframework.context.annotation.internalAutowiredAnnotationProcessor, org.springframework.context.annotation.internalRequiredAnnotationProcessor, org.springframework.context.annotation.internalCommonAnnotationProcessor, org.springframework.context.event.internalEventListenerProcessor, org.springframework.context.event.internalEventListenerFactory, cap12MainConfig, jamesBeanDefinitionRegistryPostProcessor, jamesBeanFactoryPostProcessor, getMoon, hello]
// Moon constructor........
// Moon constructor........
```

#### 源码分析

从refresh()-->invokeBeanFactoryPostProcessors()-->PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(), 跟进去即发现代码逻辑如下：

```java
class PostProcessorRegistrationDelegate {

	public static void invokeBeanFactoryPostProcessors(
			ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {

		// Invoke BeanDefinitionRegistryPostProcessors first, if any.
		Set<String> processedBeans = new HashSet<>();

		if (beanFactory instanceof BeanDefinitionRegistry) { // 在此类里，优先处理Bean定义的后置处理器，即postProcessBeanDefinitionRegistry
			BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
			List<BeanFactoryPostProcessor> regularPostProcessors = new LinkedList<>();
			List<BeanDefinitionRegistryPostProcessor> registryProcessors = new LinkedList<>();

			for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
				if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
					BeanDefinitionRegistryPostProcessor registryProcessor =
							(BeanDefinitionRegistryPostProcessor) postProcessor;
					registryProcessor.postProcessBeanDefinitionRegistry(registry);
					registryProcessors.add(registryProcessor);
				}
				else {
					regularPostProcessors.add(postProcessor);
				}
			}

```

所以，在任何情况下都会优先执行BeanDefinitionRegistryPostProcessor的处理器，而BeanFactoryPostProcessor的处理器在它后面执行











## 2. IOC容器处理流程

**详细见[springIOC核心组件分析.pdf](./springIOC核心组件分析.pdf)**

```
Spring容器的refresh()【创建刷新】;
1、prepareRefresh()刷新前的预处理;
	1）、initPropertySources()初始化一些属性设置;子类自定义个性化的属性设置方法；
	2）、getEnvironment().validateRequiredProperties();检验属性的合法等
	3）、earlyApplicationEvents= new LinkedHashSet<ApplicationEvent>();保存容器中的一些早期的事件；


2、obtainFreshBeanFactory();获取BeanFactory；
	1）、refreshBeanFactory();刷新【创建】BeanFactory；
			110行：创建了一个this.beanFactory = new DefaultListableBeanFactory();
			设置id；
	2）、getBeanFactory();返回刚才GenericApplicationContext创建的BeanFactory对象；
	3）、将创建的BeanFactory【DefaultListableBeanFactory】返回；


3、prepareBeanFactory(beanFactory);BeanFactory的预准备工作（以上创建了beanFactory,现在对BeanFactory对象进行一些设置属性）；
	1）、设置BeanFactory的类加载器、支持表达式解析器...
	2）、添加部分BeanPostProcessor【ApplicationContextAwareProcessor】
	3）、设置忽略的自动装配的接口EnvironmentAware、EmbeddedValueResolverAware、xxx；
	4）、注册可以解析的自动装配；我们能直接在任何组件中自动注入：
			BeanFactory、ResourceLoader、ApplicationEventPublisher、ApplicationContext
	5）、添加BeanPostProcessor【ApplicationListenerDetector】
	6）、添加编译时的AspectJ；
	7）、给BeanFactory中注册一些能用的组件；
		environment【ConfigurableEnvironment】、
		systemProperties【Map<String, Object>】、
		systemEnvironment【Map<String, Object>】


4、postProcessBeanFactory(beanFactory)：// Allows post-processing of the bean factory in context subclasses. 子类通过重写这个方法来在Beanfactory创建并预准备完成以后做进一步的设置。


5、invokeBeanFactoryPostProcessors(beanFactory) 	
	执行BeanFactoryPostProcessor的方beanFactory标准初始化之后执行...

	两个接口: BeanFactoryPostProcessor, BeanDefinitionRegistryPostProcessor

	BeanFactoryPostProcessor的方法:
		先执行BeanDefinit ionRegistryPostProcessor
		1、获取实现了PriorityOrdered的优先处理
		2、获取bean


6、registerBeanPostProcessors(beanFactory);	// Register bean processors that intercept bean creation.
	InstaintiationAwareBeanPostProcessor
	SmartInstaintiationAwareBeanPostProcessor--->beanPostProcessor
	拦截我们的bean的创建:后置处理器里进行增强，动态代理。。。


7、initMessageSource()标签国际化资源，初始化MessageSource组件(国际化功能:消息解析，消息绑定)
	messageSource：判断容器中是否有ID为messageSource，类型是MessageSource组件，如果没有的话,自己创建- -个newDelegatingMessageSource();
	并注册到BeanFactory，beanFactory.registerSingleton(MESSAGE_ SOURCE_ BEAN_NAME, this.messageSource) ;其实就是把bean放到一个个singletonObject Map


8、initApplicationEventMulticaster() ;初始化事件派发器
	获取beanFactory
	判断当前beanFactory有没有一-个bean名为applicationEventMulticaster;
	通过SimpleApplicationEventMulticaster来创建bean=applicationEventMulticaster


9、onRefresh()：留给子容器(子类) :子类可以重写这个方法，在容器刷新的时候可以自定义逻辑


10、registerListeners();给容器中将所有项目里面的ApplicationListener注册进来
	从我们的容器中拿到所有的ApplicationListener
	getApplicationEventMulticaster().multicastEvent(earlyEvent); => 事件派发


11、finishBeanFactoryInitialization(beanFactory)；给容器中将所有项目的单实例bean (非懒加载)初始化
	
	beanFactory. preInstantiateSingletons() ;单实例bean(非懒加载)初始化
	
	getMergedLocalBeanDefinition(beanName) ;获取bean的定义信息，依次进行创建和初始化
	
	doGetBean(name, nu1l, nu1l, false);
	
	Object sharedInstance = getSingleton(beanName) ;先获取我们MAP缓存中保存的实例bean ,如果这个bean第二次来拿，就直接从缓存MAP中拿到，其实就是从singletonObject这个Map
	
	markBeanAsCreated(beanName);标记当前bean已经被创建， this.alreadyCreated.add(beanName);进行标记
	
	mbd.getDepends0n()；beanx.xml，获取当前bean依赖的其它bean,如果存在的话，使用getBean从容器拿出来
	
	Object bean = resolveBeforeInstantiation(beanName, mbdToUse);：让我们BeanPostProcessor尝试返回-个代理对象

	Object beanInstance = doCreateBean(beanName, mbdToUse, args);: 创建bean实例

	populateBean(beanName, mbd, instanceWrapper); bean属性赋值，

	if (bp instanceofInstantiationAwareBeanPostProcessor):后置处理器的处理
		postProcessAfterInstantiation调用后置处理方法对bean进行定制处理
		applyPropertyValues (beanName, mbd, bw, pvs) ;设值

	initializeBean: bean前后处理器执行地方，AOP动态代理增强的入口


12、finishRefresh(); // Last step: publish corresponding event.
```


### 主要设计模式


BeanFactory -> getBean() 单例模式
FactoryBean -> 工厂模式
Advisor -> 适配器模式
CGLIB / JDK（实现invocationHandler）-> 动态代理 


查漏补缺：《Spring源码深度解析》















































