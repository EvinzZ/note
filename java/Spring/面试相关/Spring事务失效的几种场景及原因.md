# Spring事务失效的几种场景及原因

**抛出检查异常导致事务不能回滚**

原因：Spring默认只会回滚非检查异常

解法：配置rollbackFor属性



**业务方法内自己try-catch异常导致事务不能正确回滚**

原因：事务通知只有捉到了目标抛出的异常，才能进行后续的回滚处理，如果目标自己处理掉异常，事务通知无法知悉

解法1:异常原样抛出

解法2:手动设置TranslationStatus.setRollbackOnly()



**aop切面顺序导致事务不能正确回滚**

原因：事务切面优先级最低，但如果自定义的切面优先级和他一样，则还是自定义切面在内层，这是若自定义切面没有正确抛出异常

解法：同情况2



**非public方法导致的事务失效**

原因：Spring为方法创建代理、添加事务通知、前提条件都是该方法是public的

解法：改为public方法



**父子容器导致的事务失效**

原因：子容器扫描范围过大，把未加事务配置的service扫描进来

解法1:各扫描各的，不要图简便

解法2:不要用父子容器，所有bean放在同一容器



**调用本类方法导致传播行为失效**

原因：本类方法调用不经过代理，因此无法增强

解法1:依赖注入自己（代理）

解法2:通过AopCOntext拿到代理对象，来调用

解法3:通过CTW、LTW实现功能增强



**`@Transactional`没有保证原子行为**

原因：事务的原子性仅涵盖insert、update、delete、select ... for update语句，select方法并不阻塞



**`@Transactional`方法导致的`synchronized`失效**

原因：synchronized保证的仅是目标方法的原子性，环绕目标方法的还有commit等操作，它们并未处于sync块内

解法1:synchronize范围应扩大至代理方法调用

解法2：使用select ... for update替换select