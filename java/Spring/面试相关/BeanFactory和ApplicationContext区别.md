# BeanFactory和ApplicationContext区别

相同：

- Spring提供了两种不同的IOC容器，一个是BeanFactory，另外一个是ApplicationContext，他们都是java interface，ApplicationContext继承于BeanFactory（ApplicationContext集成ListableBeanFactory）
- 他们都可以用来配置XML属性，也支持属性的自动注入
- 而ListableBeanFactory集成BeanFactory，BeanFactory和ApplicationContext都提供了一种方式，使用getBean("bean name")获取bean

不同：

- 当调用getBean()方法时，BeanFactory仅实例化bean，而ApplicationContext在启动容器的时候实例化单例bean，不会等待调用getBean()方法时再实例化。
- BeanFactory不支持国际化，但ApplicationContext提供了对它的支持。
- BeanFactory与ApplicationContext之间的另一个区别是能够将事件发布到注册为监听器的bean
- BeanFactory的一个核心实现是XMLBeanFactory而ApplicationContext的一个核心是ClassPathXmlApplicationContext，Web容器的环境我们使用WebApplicationContext并且增加了getServletContext方法。
- 如果使用自动注入并使用BeanFactory，则需要使用API注册AutoWiredBeanPostProcessor，如果使用ApplicationContext，则可以使用XML进行配置。
- 如果使用自动注入并使用BeanFactory，则需要使用APi注册AutoWiredBeanPostProcessor，如果使用ApplicationContext，则可以使用XML进行配置。
- BeanFactory提供基本的IOC和DI功能，而ApplicationContext提供高级功能，BeanFactory可用于测试和非生产使用，但ApplicationContext是功能更丰富的容器实现，应该优于BeanFactory

