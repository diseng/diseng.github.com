---
layout: post
title : SpringMVC4 Hibernate4零XML整合 
category : program
tags : [program,java]
---
{% include JB/setup %}

自从JDK5推出Annotation以来，越来越多的应用和框架也将原先基于XML的配置方式逐渐的转而支持基于Annotation的配置方式。我本人是非常讨厌XML配置的，其一是XML文件内容显得非常得繁琐，其二是在IDE中使用XML，常常因为DTD或者Schema引起一些莫名其妙的问题。Servlet 3.0的推出，使得零XML配置成为可能。下面简单记录一下零XML整合SpringMVC4和Hibernate4的过程。

通常的java web项目都会有一个web.xml配置文件，而Servlet 3.0规范中可以通过Annotation来配置Servlet、Filter和Listener，其对应的接口是ServletContainerInitializer，其中有一个方法叫onStartup。Spring从3.1.x系列开始支持Servlet 3.0，其对ServletContainerInitializer接口的封装叫作WebApplicationInitializer。下面是使用WebApplicationInitializer来取代web.xml的一个例子。

```java
public class DefaultWebApplicationInitializer implements WebApplicationInitializer {

    /**
     * Configure the given {@link ServletContext} with any servlets, filters, listeners
     * context-params and attributes necessary for initializing this web application. See
     * examples {@linkplain org.springframework.web.WebApplicationInitializer above}.
     *
     * @param servletContext the {@code ServletContext} to initialize
     * @throws javax.servlet.ServletException if any call against the given {@code ServletContext}
     *                          throws a {@code ServletException}
     */
    @Override
    public void onStartup(javax.servlet.ServletContext servletContext) throws ServletException {
        AnnotationConfigWebApplicationContext rootContext = new AnnotationConfigWebApplicationContext();
        rootContext.register(DefaultAppConfig.class);
        servletContext.addListener(new ContextLoaderListener(rootContext));
        ServletRegistration.Dynamic dispatcher = servletContext.addServlet("dispatcher", new DispatcherServlet(rootContext));
        dispatcher.setLoadOnStartup(1);
        dispatcher.addMapping("/");
    }
}
```

其中DefaultAppConfig.class为Spring配置类，可以在其中定义一些常用的Bean，如SessionFactory,HibernateTransactionManager等，类中的ComponentScan和setPackagesToScan值依项目而定，其内容如下：

```java
@Configuration
@ComponentScan(basePackages = "com.github.diseng")
@Import(DataSourceConfig.class)
@EnableTransactionManagement
@EnableWebMvc
public class DefaultAppConfig {

    @Autowired
    private DataSourceConfig dataSourceConfig;

    @Bean
    public static PropertySourcesPlaceholderConfigurer placehodlerConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }

    @Bean
    public LocalSessionFactoryBean sessionFactory() throws PropertyVetoException {
        LocalSessionFactoryBean sessionFactoryBean = new LocalSessionFactoryBean();
        sessionFactoryBean.setDataSource(dataSourceConfig.dataSource());
        Properties hibernateProperties = new Properties();
        hibernateProperties.setProperty("hibernate.dialect", "org.hibernate.dialect.MySQL5Dialect");
        hibernateProperties.setProperty("hibernate.hbm2ddl.auto","update");
        hibernateProperties.setProperty("hibernate.show_sql","true");
        hibernateProperties.setProperty("hibernate.format_sql","true");
        sessionFactoryBean.setHibernateProperties(hibernateProperties);
        sessionFactoryBean.setPackagesToScan("com.github.diseng.model");
        return sessionFactoryBean;
    }

    @Bean
    public HibernateTransactionManager txManager() throws PropertyVetoException {
        HibernateTransactionManager txManager = new HibernateTransactionManager();
        txManager.setSessionFactory(sessionFactory().getObject());
        return txManager;
    }

    @Bean
    public UrlBasedViewResolver setupViewResolver() {
        UrlBasedViewResolver resolver = new UrlBasedViewResolver();
        resolver.setPrefix("/WEB-INF/pages/");
        resolver.setSuffix(".jsp");
        resolver.setViewClass(JstlView.class);
        return resolver;
    }
}
```

在DefaultAppConfig.class中使用@Import引入了数据源配置类DataSourceConfig.class，这里使用了jdbc.properties来单独配置数据源所用到的值，其内容如下：

```java
@Configuration
@PropertySource("classpath:jdbc.properties")
public class DataSourceConfig {
    @Value("${jdbc.driverClass}") String driverClass;
    @Value("${jdbc.url}") String url;
    @Value("${jdbc.user}") String user;
    @Value("${jdbc.password}") String password;

    @Bean
    public DataSource dataSource() throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setDriverClass(driverClass);
        dataSource.setJdbcUrl(url);
        dataSource.setUser(user);
        dataSource.setPassword(password);
        return dataSource;
    }
}
```

至此，SpringMVC4和Hibernate4的零XML配置就完成了，当然针对不一样的功能可能还需要进行相应的修改。[这里](https://github.com/diseng/SpringMVC4-Hibernate4-ZeroXML-Template)是以上代码的一个完整demo。




