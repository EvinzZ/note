# spring事物

## spring事物的七种传播机制

事物传播行为是为了解决业务层方法之间互相调用的事物问题。

### 1. PROPAGATION_REQUIRED (注解 @Transactional 默认使用的传播机制)

若当前存在事物，则加入该事物，若不存在事务，则新建一个事务。

```java
class C1 {
    @Transactional(propagation = PROPAGATION.REQUIRED)
    function A() {
        C2.B();            
    }    
}

class C2 {
    @Transactional(propagation = PROPAGATION.REQUIRED)
    function B() {
        do something;           
    }    
}
```

- spring在调用methodA的时候没有事务，因此创建一个新的事务，而methodA中又调用了methodB，而此时已经存在一个事务了，所以methodB会加入到这个事务中
   对于第二个方法，由于methodB此时没有事务，因此spring会开启一个新的事务。spring会确保方法中的所有调用都得到一个相同的连接。
- A, B 可以操作同一条纪录, 因为处于同一个事务中.

## 2. PROPAGATION_REQUIRED_NEW

**若当前不存在事务, 则新建一个事务. 若当前存在事务, 则将当前事务挂起, 新建一个事务**. 即不管外部方法有无事务, PROPAGETION_REQUIRED_NEW 修饰的内部方法都会开启一个新事务, 且开启的事务相互独立, 互不干扰. 外部事务抛出异常回滚不会影响内部事务正常提交.

```java
class C1 {
    @Transactional(propagation = PROPAGATION.REQUIRED)
    function A() {
        do somethingA();
       C2.B();   
       do somethingB();                
    }    
}

class C2 {
    @Transactional(propagation = PROPAGATION.REQUIRED_NEW)
    function B() {
        do something;           
    }    
}
```

- 这两段代码在一起。此时methodA会创建一个新的事务tr1，而methodA在执行到methodB()方法时，调用methodB方法，此时methodB方法又会创建一个新的事务tr2，这两个事务是没有关联相互独立的，如果methodA中在执行完methodB方法之后的doSomeThingB()方法执行失败了，那么methodA方法的数据会执行回滚不会提交，但是其中的methodB的数据仍然会提交到数据库中，这表明tr1和tr2之间是独立的。也就是说，只要methodB成功执行了，那么methodB的数据就会被加入到数据库，不管其后面的代码是否发生异常。如果使PROPAGATION_REQUIRES_NEW，需要使用JtaTransactionManager作为事务管理器。
- A,B 不可操作同一条纪录, 因为处于不同事务, 会产生死锁.

## 3. PROPAGATION_NESTED (嵌套事务)

**如果当前存在事务, 则创建一个事务作为当前事务的嵌套事务来运行; 如果当前没有事务, 则该取值等价于 propagation_required**

```java
class C1 {
    @Transactional(propagation = PROPAGATION.REQUIRED)
    function A() {
        do somethingA();
        C2.B();   
        do somethingB();         
    }    
}

class C2 {
    @Transactional(propagation = PROPAGATION.NESTED)
    function B() {
        do something;           
    }    
}
```

这里如果单独调用methodB方法，则按照PROPAGATION_REQUIRED属性执行。

如果调用methodA方法，那么在执行methodA方法内调用methodB方法的时候，会先调用setSavePoint方法，保存当前的状态到savepoint中，如果methodB方法调用失败，则恢复到之前记录的那个状态，然后继续执行后续代码，此时事务都还没有提交，如果其后面的doSomeThingB()方法调用失败，则回滚包括methodB在内的所有操作。也就是所：这是一个嵌套事务，嵌套事务的内层事务依赖于外层事务，如果外层事务失败时，会回滚内层事务，而内层事务操作失败的时候，并不会引起外层事务的回滚，外层事务会回复到内层事务执行之前的状态，然后继续执行后面的方法。

## 4. PROPAGATION_SUPPORTS

**如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。**

## 5. PROPAGATION_NOT_SUPPORTS

**以非事务方式运行，如果当前存在事务，则把当前事务挂起。**

## 6. PROPAGATION_MANDATORY (很少使用)

**如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。**（mandatory：强制性）

## 7. PROPAGATION_NEVER

**以非事务方式运行，如果当前存在事务，则抛出异常。**

注：
 PROPAGATION_NESTED 与 PROPAGATION_REQUIRES_NEW 的区别：这两个事物的最大区别是 PROPAGATION_NESTED 是一个嵌套事务，内层事务依赖于外层事务，内层事务成功执行完成之后，数据其实并没有提交到数据库，而需要外层事务全部成功完成之后，内外层数据才会提交到数据库，若内层事务虽然成功执行，但是外层事务在其之后发生异常，则内层事务和外层事务全部回滚；
 而 PROPAGATION_REQUIRES_NEW 是完全两个不同的事务，相互之间是独立的，一旦内层事务提交了，外层事务发生异常也并不能让内层事务回滚。

# 二. spring 事务的四种隔离级别

## 1. 事务四大特性 (ACID)

**原子性**   -->   操作要么全部成功, 要么全部回滚.
 **一致性**   -->   事务执行前后处于一致性状态.
 **隔离性**   -->   事务直接互不干扰.
 **持久性**   -->   事务一旦提交, 数据的改变是永久性的. 即使数据库发生故障, 数据也不会丢失.

## 2. 事务隔离级别相关问题

**脏读**
 **脏读指事务A读到了事务B还没有提交的数据**。 A事务对一条记录进行修改，尚未提交，B事务已经看到了A的修改结果。若A发生回滚，B读到的数据就是错误的，这就是脏读。

**不可重复读**
 **在一个事务里面读取了两次某个数据，读出来的数据不一致。**A事务对一条记录进行修改，尚未提交，B事务第一次查询该记录，看到的是修改之后的结果，此时A发生回滚，B事务又一次查询该记录，看到的是回滚后的结果。同一个事务内，B两次查询结果不一致，这就是不可重复读。

**幻读**
 **所谓幻读，就是指在一个事务里面的操作中发现了未被操作的数据**。比如学生信息，事务A开启事务-->修改所有学生当天签到状况为false，此时切换到事务B，事务B开启事务-->事务B插入了一条学生数据，此时切换回事务A，事务A提交的时候发现了一条自己没有修改过的数据，这就是幻读，就好像发生了幻觉一样。幻读出现的前提是并发的事务中有事务发生了插入、删除操作。

## 3. 事务的隔离级别

和事务传播行为这块一样，为了方便使用，Spring 也相应地定义了一个枚举类：Isolation



```csharp
public enum Isolation {
    
    // 使用后端数据库默认的隔离级别 (MySQL --> REPEATABLE_READ, Oracle --> READ_COMMITTED)
    DEFAULT(TransactionDefinition.ISOLATION_DEFAULT),
    
    // 读未提交
    READ_UNCOMMITTED(TransactionDefinition.ISOLATION_READ_UNCOMMITTED),
    
    // 读提交
    READ_COMMITTED(TransactionDefinition.ISOLATION_READ_COMMITTED),
    
    // 可重复读 ---> innodb默认隔离级别
    REPEATABLE_READ(TransactionDefinition.ISOLATION_REPEATABLE_READ),
 
    // 序列化
    SERIALIZABLE(TransactionDefinition.ISOLATION_SERIALIZABLE);

    private final int value;

    Isolation(int value) {
        this.value = value;
    }

    public int value() {
        return this.value;
    }

}
```

**ISOLATION_DEFAULT**
 使用后端数据库默认的隔离级别，MySQL 默认采用的 REPEATABLE_READ 隔离级别, Oracle 默认采用的 READ_COMMITTED 隔离级别

**ISOLATION_READ_UNCOMMITTED**   -->  读未提交
 最低的隔离级别，使用这个隔离级别很少，因为它允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读

**ISOLATION_READ_COMMITTED**  --> 读已提交
 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生

**ISOLATION_REPEATABLE_READ**  --> 可重复读
 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。

**ISOLATION_SERIALIZABLE**  --> 序列化
 最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

