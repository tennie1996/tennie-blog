---
title: Java学习笔记（十三）：Junit单元测试、反射、注解
date: 2020-1-10 15:17
urlname: 2020011001
categories: Java
tags:
  - Java
author: foochane
toc: true
mathjax: true
top: false
top_img: /images/banner/0.jpg
cover: /images/cover/1.jpg
---


<!-- 
>[foochane](https://foochane.cn/) ：[https://foochane.cn/article/2020011001.html](https://foochane.cn/article/2020011001.html) 

[toc]
-->


## 1. Junit单元测试



### 1.1 测试分类

1. 黑盒测试：不需要写代码，给输入值，看程序是否能够输出期望的值。
2. 白盒测试：需要写代码的。关注程序具体的执行流程。



### 1.2 Junit使用

Junit测试属于白盒测试。

使用步骤如下：



1. 定义一个测试类(测试用例)

   如：

   -  测试类名：被测试的类名Test		CalculatorTest
   -  包名：xxx.xxx.xx.test		cn.xxxx.test

2. 定义测试方法：可以独立运行

   - 方法名：test测试的方法名		testAdd()  

   - 返回值：void

   - 参数列表：空参

3. 给方法加@Test

4. 导入junit依赖环境



### 1.3 判定结果

一般我们会使用断言操作来处理结果 : `Assert.assertEquals(期望的结果,运算的结果)`

- 红色：失败 
- 绿色：成功



### 1.4 补充

* `@Before`: 修饰的方法会在测试方法之前被自动执行
* `@After`: 修饰的方法会在测试方法执行之后自动被执行



### 1.5 代码示例

先写被测试的方法：

```java 
/**
 * 计算器类
 */
public class Calculator {


    /**
     * 加法
     * @param a
     * @param b
     * @return
     */
    public int add (int a , int b){
        //int i = 3/0;

        return a - b;
    }

    /**
     * 减法
     * @param a
     * @param b
     * @return
     */
    public int sub (int a , int b){
        return a - b;
    }

}
```



添加测试类

```java 
public class CalculatorTest {
    /**
     * 初始化方法：
     *  用于资源申请，所有测试方法在执行之前都会先执行该方法
     */
    @Before
    public void init(){
        System.out.println("init...");
    }

    /**
     * 释放资源方法：
     *  在所有测试方法执行完后，都会自动执行该方法
     */
    @After
    public void close(){
        System.out.println("close...");
    }


    /**
     * 测试add方法
     */
    @Test
    public void testAdd(){
       // System.out.println("我被执行了");
        //1.创建计算器对象
        System.out.println("testAdd...");
        Calculator c  = new Calculator();
        //2.调用add方法
        int result = c.add(1, 2);
        //System.out.println(result);

        //3.断言  我断言这个结果是3
        Assert.assertEquals(3,result);

    }

    @Test
    public void testSub(){
        //1.创建计算器对象
        Calculator c  = new Calculator();
        int result = c.sub(1, 2);
        System.out.println("testSub....");
        Assert.assertEquals(-1,result);
    }
}
```



## 2 反射：框架设计的灵魂

### 2.1 框架

框架：半成品软件。可以在框架的基础上进行软件开发，简化编码



### 2.2 反射的概念

* 反射：将类的各个组成部分封装为其他对象，这就是反射机制
* 好处：
    1. 可以在程序运行过程中，操作这些对象。
    2. 可以解耦，提高程序的可扩展性。

### 2.3 获取Class对象的方式

1. `Class.forName("全类名")`：将字节码文件加载进内存，返回Class对象

   多用于配置文件，将类名定义在配置文件中。读取文件，加载类

2. `类名.class`：通过类名的属性`class`获取

   多用于参数的传递

3. `对象.getClass()`：`getClass()`方法在Object类中定义着。

   多用于对象的获取字节码的方式

代码演示：

创建Person类：

```java 
package cn.chane.domain;

public class Person {
    private String name;
    private int age;

    public String a;
    protected String b;
    String c;
    private String d;


    public Person() {
    }

    public Person(String name, int age) {

        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", a='" + a + '\'' +
                ", b='" + b + '\'' +
                ", c='" + c + '\'' +
                ", d='" + d + '\'' +
                '}';
    }


    public void eat(){
        System.out.println("eat...");
    }

    public void eat(String food){
        System.out.println("eat..."+food);
    }
}
```



获取Class对象：

```java 
package cn.chane.reflect;

import cn.chane.domain.Person;
import cn.chane.domain.Student;

public class ReflectDemo1 {


    /**
        获取Class对象的方式：
            1. Class.forName("全类名")：将字节码文件加载进内存，返回Class对象
            2. 类名.class：通过类名的属性class获取
            3. 对象.getClass()：getClass()方法在Object类中定义着。

     */
    public static void main(String[] args) throws Exception {

        //1.Class.forName("全类名")
        Class cls1 = Class.forName("cn.chane.domain.Person");
        System.out.println(cls1);
        //2.类名.class
        Class cls2 = Person.class;
        System.out.println(cls2);
        //3.对象.getClass()
        Person p = new Person();
        Class cls3 = p.getClass();
        System.out.println(cls3);

        //== 比较三个对象
        System.out.println(cls1 == cls2);//true
        System.out.println(cls1 == cls3);//true


        Class c = Student.class;
        System.out.println(c == cls1); //false


    }
}
```



> 同一个字节码文件`(*.class)`在一次程序运行过程中，只会被加载一次，不论通过哪一种方式获取的Class对象都是同一个。



### 2.4 Class对象功能

1. 获取成员变量们
	* `Field[] getFields()` ：获取所有public修饰的成员变量
	* `Field getField(String name)`   ：获取指定名称的 public修饰的成员变量
	* `Field[] getDeclaredFields()`  ：获取所有的成员变量，不考虑修饰符
	* `Field getDeclaredField(String name)`  ：
	
2. 获取构造方法们
	* `Constructor<?>[] getConstructors()  `
	* `Constructor<T> getConstructor(类<?>... parameterTypes)`  
	* `Constructor<T> getDeclaredConstructor(类<?>... parameterTypes)`  
	* `Constructor<?>[] getDeclaredConstructors()`  
3. 获取成员方法们：
	* `Method[] getMethods()`  
	* `Method getMethod(String name, 类<?>... parameterTypes)`  
	* `Method[] getDeclaredMethods()`  
	* `Method getDeclaredMethod(String name, 类<?>... parameterTypes)`  

4. 获取全类名	
	* `String getName()`  

### 2.5 Field：成员变量

具体操作

1. 设置值
	* `void set(Object obj, Object value)  `
2. 获取值
	* `get(Object obj) `

3. 忽略访问权限修饰符的安全检查
	* `setAccessible(true)`:暴力反射



### 2.6 Constructor:构造方法

创建对象：

* `T newInstance(Object... initargs)  `
* 如果使用空参数构造方法创建对象，操作可以简化：Class对象的newInstance方法



### 2.7 Method：方法对象

* 执行方法：`Object invoke(Object obj, Object... args)  `


* 获取方法名称：`String getName:获取方法名`



## 3 注解

### 3.1 概念



* 概念：说明程序的。给计算机看的
* 注释：用文字描述程序的。给程序员看的

* 定义：注解（Annotation），也叫元数据。一种代码级别的说明。它是JDK1.5及以后版本引入的一个特性，与类、接口、枚举是在同一个层次。它可以声明在包、类、字段、方法、局部变量、方法参数等的前面，用来对这些元素进行说明，注释。
* 概念描述：
    * JDK1.5之后的新特性
    * 说明程序的
    * 使用注解：@注解名称

​	

### 3.2 注解的作用

* 作用分类：
    ①编写文档：通过代码里标识的注解生成文档【生成文档doc文档】
    ②代码分析：通过代码里标识的注解对代码进行分析【使用反射】
    ③编译检查：通过代码里标识的注解让编译器能够实现基本的编译检查【Override】



### 3.4 JDK中预定义的一些注解



* `@Override`	：检测被该注解标注的方法是否是继承自父类(接口)的
* `@Deprecated`：该注解标注的内容，表示已过时
* `@SuppressWarnings`：压制警告
* 一般传递参数all : `@SuppressWarnings("all")`



### 3.5 自定义注解

#### 格式

```java
public @interface 注解名称{
	属性列表;
}
```
#### 本质

注解本质上就是一个接口，该接口默认继承Annotation接口

#### 要求

1. 属性的返回值类型有下列取值
    * 基本数据类型
    * String
    * 枚举
    * 注解
    * 以上类型的数组


2. 定义了属性，在使用时需要给属性赋值
    1. 如果定义属性时，使用default关键字给属性默认初始化值，则使用注解时，可以不进行属性的赋值。
    2. 如果只有一个属性需要赋值，并且属性的名称是value，则value可以省略，直接定义值即可。
    3. 数组赋值时，值使用{}包裹。如果数组中只有一个值，则{}可以省略

#### 元注解

用于描述注解的注解

@Target：描述注解能够作用的位置
* `ElementType`取值：
    * `TYPE`：可以作用于类上
    * `METHOD`：可以作用于方法上
    * `FIELD`：可以作用于成员变量上
* `@Retention`：描述注解被保留的阶段
* `@Retention(RetentionPolicy.RUNTIME)`：当前被描述的注解，会保留到class字节码文件中，并被JVM读取到
* `@Documented`：描述注解是否被抽取到api文档中
* `@Inherited`：描述注解是否被子类继承

### 3.6 在程序使用（解析）注解

获取注解中定义的属性值

1. 获取注解定义的位置的对象  （`Class`，`Method`,`Field`）
2. 获取指定的注解
3. 调用注解中的抽象方法获取配置的属性值
