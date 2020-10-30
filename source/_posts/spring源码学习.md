---
title: SpringBoot 源码学习 一
date: 2020-10-16 18:02:12
tags:
---

### 正文开始

#### SpringBoot如何通过main方法启动整个容器的？

启动入口：
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringBootLearnApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootLearnApplication.class, args);
    }

}
```
在`SpringApplication`在执行静态方法`run`的时候，首先初始化`SpringApplication`实例，见代码：
```java
class SpringApplication{
    
    private ResourceLoader resourceLoader;
    
    
    public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
        this.resourceLoader = resourceLoader; // 1 
        Assert.notNull(primarySources, "PrimarySources must not be null");
        this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources)); // 2
        this.webApplicationType = WebApplicationType.deduceFromClasspath(); // 3 
        setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class)); // 4
        setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class)); // 5
        this.mainApplicationClass = deduceMainApplicationClass(); //6
    }
}

```

可以看到，在初始化 `SpringApplication` 的时候，会有几个过程：
1. 给 resourceLoader 赋值（当前为null），resourceLoader是一个资源加载器，可以加载文件系统或者classpath下的一些资源，返回`Resource`
2. 指定 primarySources，主要指定当前SpringBoot main方法所在的类
3. 判断当前应用的类型，主要判断思路是check当前classpath中是否有相应的类，应用类型有 `NONE`， `SERVLET`， `REACTIVE` 三种
4. 4和5都用到了 `getSpringFactoriesInstances`方法，该方法主要是通过`SpringFactoriesLoader` 加载classpath下`META-INF/spring.factories`中的一些类， 该步骤主要是用来加载`ApplicationContextInitializer`的实现类
5. 同4，用来实例化一些listeners, `ApplicationListener`的实现，用来处理 spring的各种`ApplicationEvent`事件
6. 从当前的堆栈信息中找到main方法所在的类，设置为springboot的main class

至此，SpringApplication就初始化完成了。

下面继续看`SpringApplication.run`的实现：
继续上代码：
```java

class SpringApplication{
    
    public ConfigurableApplicationContext run(String... args) {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        ConfigurableApplicationContext context = null;
        Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
        configureHeadlessProperty();
        SpringApplicationRunListeners listeners = getRunListeners(args);
        listeners.starting();
        try {
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
            configureIgnoreBeanInfo(environment);
            Banner printedBanner = printBanner(environment);
            context = createApplicationContext();
            exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
    				new Class[] { ConfigurableApplicationContext.class }, context);
            prepareContext(context, environment, listeners, applicationArguments, printedBanner);
            refreshContext(context);
            afterRefresh(context, applicationArguments);
            stopWatch.stop();
            if (this.logStartupInfo) {
                new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
            }
            listeners.started(context);
            callRunners(context, applicationArguments);
        }
        catch (Throwable ex) {
            handleRunFailure(context, ex, exceptionReporters, listeners);
            throw new IllegalStateException(ex);
        }
    
        try {
            listeners.running(context);
        }
        catch (Throwable ex) {
            handleRunFailure(context, ex, exceptionReporters, null);
            throw new IllegalStateException(ex);
        }
        return context;
    }
}
```
