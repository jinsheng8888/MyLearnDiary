#### 1. 开始

```
@SpringBootApplication
public class MyApplication {
 
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

#### 2.  初始化

- **推断应用类型**
- **加载初始化构造器ApplicationContextInitializer**
- **创建应用监听器**
- **设置应用main()方法所在的类**

```

    public SpringApplication(ResourceLoader resourceLoader, Class... primarySources) {
        this.sources = new LinkedHashSet();
        //默认的横幅模式console
        this.bannerMode = Mode.CONSOLE;
        //是否打印启动日志默认为true
        this.logStartupInfo = true;
        //添加命令行属性默认为true
        this.addCommandLineProperties = true;
        this.addConversionService = true;
        //无显示器模式，默认为true
        this.headless = true;
        //默认注册一个关闭应用的钩子函数，可以实现springboot优雅关闭
        this.registerShutdownHook = true;
        this.additionalProfiles = new HashSet();
        this.isCustomEnvironment = false;
        this.resourceLoader = resourceLoader;
        Assert.notNull(primarySources, "PrimarySources must not be null");
        this.primarySources = new LinkedHashSet(Arrays.asList(primarySources));
        **推断应用类型**
        this.webApplicationType = WebApplicationType.deduceFromClasspath();
            //**加载初始化构造器ApplicationContextInitializer**this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
             //**创建应用监听器**
        this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
        //**设置应用main()方法所在的类**
        this.mainApplicationClass = this.deduceMainApplicationClass();
    }
```



#### 3. 初始化完成后，开始执行

**Headless模式设置**

***\*加载SpringApplicationRunListeners监听器\****

**封装ApplicationArguments对象**

**配置环境模块**

**根据环境信息配置要忽略的bean信息**

**Banner配置SpringBoot彩蛋**

**创建ApplicationContext应用上下文**

**加载SpringBootExceptionReporter**

**ApplicationContext基本属性配置**

**更新应用上下文**

**查找是否注册有CommandLineRunner/ApplicationRunner**

```
  try {
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            ConfigurableEnvironment environment = this.prepareEnvironment(listeners, applicationArguments);
            this.configureIgnoreBeanInfo(environment);
            Banner printedBanner = this.printBanner(environment);
            context = this.createApplicationContext();
            exceptionReporters = this.getSpringFactoriesInstances(SpringBootExceptionReporter.class, new Class[]{ConfigurableApplicationContext.class}, context);
            this.prepareContext(context, environment, listeners, applicationArguments, printedBanner);
            this.refreshContext(context);
            this.afterRefresh(context, applicationArguments);
            stopWatch.stop();
            if (this.logStartupInfo) {
                (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), stopWatch);
            }

            listeners.started(context);
            this.callRunners(context, applicationArguments);
        } catch (Throwable var10) {
            this.handleRunFailure(context, var10, exceptionReporters, listeners);
            throw new IllegalStateException(var10);
        }
```



#### 4. 创建bean, this.refreshContext(context);

```
public void refresh() throws BeansException, IllegalStateException {//刷新spring容器
    synchronized(this.startupShutdownMonitor) {//线程安全
        this.prepareRefresh();//刷新前预处理（初始化容器需要的一些配置参数信息）
        ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();//获取BeanFactory（DefaultListableBeanFactory对象）
        this.prepareBeanFactory(beanFactory);//预处理DefaultListableBeanFactory（设置一些必要的对象属性信息）
        try {
            this.postProcessBeanFactory(beanFactory);//处理beanFactory对象
            this.invokeBeanFactoryPostProcessors(beanFactory);//处理一些自动配置类的信息并生成bean 比方说一些自动配置类，Configuration注解bean注解import注解componentScan注解等信息（这里调用栈太深了，懒得贴代码，有兴趣自己看）
            this.registerBeanPostProcessors(beanFactory);//添加BeanPostProcessors对象的实现类，并在bean实例化显示之前调用
            this.initMessageSource();//初始化国际化资源信息（有配置messageSourcebean，则获取该bean，否则创建DelegatingMessageSource对象）
            this.initApplicationEventMulticaster();//初始化应用相关的监听器
            this.onRefresh();//这里主要就是启动内置的web容器比方说Tomcat或jetty
            this.registerListeners();//发布及注册监听器
            this.finishBeanFactoryInitialization(beanFactory);//自动注入完成beanFactory的初始化创建单例的bean的实例（调用前置后置的处理方法，调用Aware接口的子类，调用afterPropertySet方法和自定义的初始化方法）
            this.finishRefresh();//完成刷新容器
        } catch (BeansException var9) {
            if (this.logger.isWarnEnabled()) {
                this.logger.warn("Exception encountered during context initialization - cancelling refresh attempt: " + var9);
            }
 
            this.destroyBeans();
            this.cancelRefresh(var9);
            throw var9;
        } finally {
            this.resetCommonCaches();
        }
 
    }
}
```

