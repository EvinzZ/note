# JDK1.8的新特性有哪些

## 1.接口的默认方法

java8允许我们给接口添加一个非抽象的方法实现，只需要使用default关键字即可，这个特征又叫做扩展方法。

案例：

```java
interface Formual {
    double calculate(int a);
    default double sqrt(int a) {
        return Math.sqrt(a);
    }                  
}
```

Formula接口在拥有calculate方法之外同时还定义了sqrt方法，实现了Formula接口的子类只需要实现一个calculate方法，默认方法sqrt将在子类上可以直接使用。

代码如下：

```java
Formula formula = new Formula(){
    @Override
    public double calculate(int a) {
        return sqrt(a*100);
    }
}
formula.calculate(100); // 100.0
formula.sqrt(16); // 4.0
```

文中的formula被实现为一个匿名类的实例，该代码非常容易理解，6行代码实现了计算sqrt(a * 100)。

## 2.lambda表达式

老版本java排列字符串

代码：

```java
List<String> names = Arrays.asList("peterF", "anna", "mike", "xenia");
Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return b.compareTo(a);
    }
});
```

只需要给静态方法Collections.sort传入一个list对象以及一个比较器来按指定顺序排列。通常做法都是创建一个匿名的比较器对象然后将其传递给sort方法。



java8方式

代码：

```java
Collections.sort(names, (String a, String b) -> {
    return b.compareTo(a);
});
```

```java
Collections.sort(names, (String a, String b) -> b.compareTo(a));
```

```java
Collections.sort(names, (a, b) -> b.compareTo(a));
```

## 3.函数式接口

## 4.方法与构造函数引用

## 5.lambda作用域

在lambda表达式中访问外层作用域和老版本的匿名对象中的方式很相似，可以直接访问标记了final的外层局部变量，或者实例的字段以及静态变量。

## 6.访问局部变量

## 7.访问对象字段与静态变量