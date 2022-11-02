# Spring MVC的九大组件

**1.HandlerMapping**

根据request找到相应的处理器。因为Handler有两种形式，一种是基于类的Handler，另一种是基于Method的Handler；

**2.HandlerAdapter**

调用Handler的适配器。如果把Handler当作工具的话，那么HandlerAdapter就相当于干活的工人

**3.HandlerExceptionResolver**

对异常的处理

**4.ViewResolver**

用来将String类型的视图名和Locale解析为View类型的视图。

**5.RequestToViewNameTranslator**

有的Handler处理完后没有设置返回类型，比如是void方法，这时就需要从request中获取viewName。

**6.LocaleResolver**

从request中解析出Locale。Locale表示一个区域，比如Zh-cn，对不同的区域的用户，显示不同的结果，这就是i18n（SpringMVC中有具体的拦截器LocaleChangeInterceptor）

**7.ThemeResolver**

主题解析，这种类似于我们手机更换主题，不同的UI，css等

**8.MultipartResolver**

处理上传请求，将普通的request封装成MultipartHttpServletRequest

**9.FlashMapManager**

用于管理FlashMap，FlashMap用于在redirect重定向中传递参数