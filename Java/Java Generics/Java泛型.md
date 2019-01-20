[TOC]

# Generics泛型简介

Java泛型方法和泛型类使程序员可以用一个方法声明一组相关的方法，或者用一个类声明一组相关的类型。泛型还提供编译时类型安全性，允许程序员在编译时捕获无效类型，目前IDE已经能够在编写代码时给出错误提示。  

Java中的泛型使用 `<参数类型>`去表示。  

# 泛型类

泛型类在集合框架中很常见，如，`ArrayList<String>` 和 `HashMap<String, Person>` 。  

泛型类的定义和普通类的定义很相似，只是在类名后面跟上了类型参数，并且类型参数在整个类中有效，当类型参数被明确为具体的类型后，该类中所有操作的类型就固定了。    

```java
public class MyGenericsClass<T> {
    private T t;

    public T get(){
        return t;
    }

    public void set(T t){
        this.t = t;
    }
    /*
    [output]
    String is : R.I.P
    Integer is : 42
    */
    public static void main(String[] args) {
        MyGenericsClass<String> str = new MyGenericsClass<>();
        MyGenericsClass<Integer> num = new MyGenericsClass<>();
        str.set("R.I.P");
        num.set(42);
        System.out.println("String is : " + str.get());
        System.out.println("Integer is : " + num.get());
    }
}
```

与泛型方法一样，泛型类的类型参数部分可以具有一个或多个由逗号分隔的类型参数。这些类被称为参数化类或参数化类型，因为类接受一个或多个参数。  

```java
public class MyGenericsClass<T, R> {
    private T t;
    private R r;

    public T getT(){
        return t;
    }

    public void setT(T t){
        this.t = t;
    }

    public R getR(){
        return r;
    }

    public void setR(R r){
        this.r = r;
    }

    public R test(){
        System.out.println(this.t);
        return this.r;
    }

    public static void main(String[] args) {
        MyGenericsClass<String, Integer> mg = new MyGenericsClass<>();
        mg.setT("cloudsen");
        mg.setR(mg.getT().length());
        System.out.println(mg.test());
    }
}
```



# 泛型方法

泛型类会固定所有用到参数类型的方法的操作类型，**泛型方法可以让类中不同的方法操作不同的类型**。  

泛型方法的定义如下，就是在返回类型的前面加上了 `<参数类型>` ：  

```java
public <T> void test(T t) {
    //bulabula...
}

public static <T> void test2(T t) {
    //bulabula...
}

public abstract <T> void test(T t);
```

注意，**泛型方法的参数类型可以和泛型类的参数类型同名**：  

```java
public class MyClass<T> {
    // 这里的T就是类的T
    public void test(T t) {
        System.out.println("print: " + t);
    }
    
    // 这里的T不是类的T，而是根据你调用方法时，传递的类型动态改变
    public <T> void test2(T t) {
        System.out.println("print: " + t);
    }
}
```

注意，**静态方法只能使用泛型方法，静态方法无法访问类的类型参数。**因为类的类型参数是在实例化的时候指定的，静态方法不依靠实例对象。  

```java
public class MyGenericsClass<T> {
    public void printClass(T t) {
        System.out.println(t);
    }

    public <T> void printNormal(T t) {
        System.out.println(t);
    }

    public static <T> void printStatic(T s) {
        System.out.println(s);
    }

    /*
    [output]
    true
    print normal
    66666
    print static
    false
    */
    public static void main(String[] args) {
        MyGenericsClass<Boolean> myClass = new MyGenericsClass<>();
        myClass.printClass(Boolean.TRUE);
        myClass.printNormal("print normal");
        myClass.printNormal(66666);
        MyGenericsClass.printStatic("print static");
        MyGenericsClass.printStatic(Boolean.FALSE);
    }
}
```



# 泛型接口

## 定义

泛型接口的定义如下：  

```java
interface MyInterface<T, R> {
    void myMethod(T t);
    
    default void myDefaultMethod(R r);
    
    static <T> void testStatic(T t) {
        System.out.println(t);
    }
}
```

## 实现接口的同时指定类型参数

如果在实现接口的同时指定类型参数，那么就直接定死了接口的泛型类型，实现类的实例对象只能通过该接口处理指定的类型。
```java
interface MyInterface<T> {
    void test(T t);

    default void testDefault(T t) {
		System.out.println(t);
    }
    
	// 泛型方法，这里的T是在调用时指定
    static <T> void testStatic(T t) {
        System.out.println(t);
    }
}

class MyClass implements MyInterface<String> {
    @Override
    public void test(String s) {
        System.out.println(s);
    }
}

/*
[output]
haha
hehe
heihei
66666
*/
public static void main(String[] args) {
    MyClass myClass = new MyClass();
    myClass.test("haha");
    myClass.testDefault("hehe");
    MyInterface.testStatic("heihei");
    MyInterface.testStatic(66666);
}
```

## 实现接口的同时不指定类型参数

如果实现接口的同时不指定类型参数，那么该接口的实现类也必须是泛型类。生成实现类对象时再去指定泛型类型，则可以通过该接口处理不同类型的数据。  

```java
interface Test<T> {
    void test(T t);

    default void testDefault(T t) {
        System.out.println(t);
    }

    static <T> void testStatic(T t) {
        System.out.println(t);
    }
}

class MyClass<T> implements Test<T> {
    @Override
    public void test(T s) {
        System.out.println(s);
    }
}

/*
haha
hehe
6666
9999
heihei
42
*/
public static void main(String[] args) {
    MyClass<String> myClass = new MyClass<>();
    MyClass<Integer> myClass2 = new MyClass<>();
    myClass.test("haha");
    myClass.testDefault("hehe");
    myClass2.test(6666);
    myClass2.testDefault(9999);
    Test.testStatic("heihei");
    Test.testStatic(42);
}
```



# 限制类型参数

