# Spring bean的生命周期

## 1.概述

```java
protected <T> T doGetBean(String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly)
```

- 阶段1:处理名称，检查缓存
- 阶段2:检查父工厂
- 阶段3:检查DependsOn
- 阶段4:按Scope创建bean
  - 创建`singleton`
  - 创建`prototype`
  - 创建其他`scopre`
- 阶段5:创建bean
  - 创建bean实例
  - 依赖注入
  - 初始化
  - 登记可销毁bean
- 阶段6:类型转换
- 阶段7:销毁bean

### 1.1.处理名称，检查缓存

- 先把别名解析为实际名称，在进行后续处理
- 若要FactoryBean本身，需要使用&名称获取
- singletonObjects是一级缓存，放单例成品对象
- singletonFactories是三级缓存，放单例工厂
- earlySingletonObjects是二级缓存，放单例工厂的产品，可称为提前单例对象

### 1.2.处理父子容器

- 父子容器的bean名称可以重复
- 优先找子容器的bean，找到了直接返回，找不到继续到父容器找

### 1.3.dependsOn

dependsOn用在非显式依赖的bean的创建顺序控制

### 1.4.按Scope创建bean

- scope理解为从xxx范围内找这个bean更加贴切
- singleton scope表示从单例池范围内获取bean，如果没有，则创建并放入单例池
- prototype scope，它从不缓存bean，每次都创建新的
- request scope表示从request对象范围内获取bean，如果没有，则创建并放入request