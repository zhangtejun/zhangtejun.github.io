---
layout: post
title:  "注解Annotation"
date:   2017-06-22 10:53:02
author: zhangtejun
categories: java
---
##### 注解

在JDK5.0中，我们可以自定义这些标签，并通过Java语言的反射机制中获取类中标注的注解，完成特定的功能。  
注解是代码的附属信息，它遵循一个基本原则：注解不能直接干扰程序代码的运行，无论增加或删除注解，代码都能够正常运行。
Java语言解释器会忽略这些注解，而由第三方工具负责对注解进行处理。第三方工具可以利用代码中的注解间接控制程序代码的运行，
它们通过Java反射机制读取注解的信息，并根据这些信息更改目标程序的逻辑，而这正是Spring AOP对@AspectJ提供支持所采取的方法。 

##### 一个简单的注解类  
```java
/**
 * Created by zhangtejun on 2017/2/21.
 * 
 * 1. Java5.0定义了4个meta-annotation(元注解)
 * 	  @Target
 * 	  @Retention
 * 	  @Documented
 *    @Inherited
 * 
 * 2. 元注解(注解的注解)
 * 	@Target 标识这个注解的作用域.
 *          ElementType.METHOD : 这个注解的作用域的列表，METHOD是方法声明，除此之外，还有：
 *          ElementType.CONSTRUCTOR（构造方法声明）
 *          ElementType.FIELD（字段声明）
 *          ElementType.ANNOTATION_TYPE(annotation)
 *          ElementType.LOCAL_VARIABLE（局部变量声明）
 *          ElementType.PACKAGE（包声明）
 *          ElementType.PARAMETER（参数声明）
 *          ElementType.TYPE（类接口）
 * 3. 生命周期
 * @Retention 声明注解的保留期限,即它的生命周期。
 * 		RUNTIME 就是在运行时存在，可以通过反射读取。除此之外，还有:
 * 		SOURCE（只在源码显示，编译时丢弃）
 * 		CLASS（编译时记录到class中，运行时忽略）
 * 4. 标识
 * @Inherited 是一个标识性的元注解，它允许子注解继承它。阐述了某个被标注的类型是被继承的。
 * 如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类。
 * 5. 
 * @Documented  生成javadoc时会包含注解。
 *
 *
 * 所有的注解类都隐式继承于java.lang.annotation.Annotation，但注解不允许显式继承于其他的接口
 */
@Target({ElementType.METHOD,ElementType.CONSTRUCTOR,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface MyAnnotation {
	
	String desc() default "annotation";//成员以无参无异常的方式声明，可有默认值.
    String author();
    public enum FontColor{ BULE,RED,GREEN};//枚举
    FontColor fontColor() default FontColor.GREEN;
    Class myclass() default String.class;//class
    Description ds() default @Description(desc = "22",author = "ere");//注解
}
```
##### AnnotatedElement接口
```java
//java.lang.reflect包下
public interface AnnotatedElement
```
表示目前正在此 VM 中运行的程序的一个已注释元素。该接口允许反射性地读取注释。由此接口中的方法返回的所有注释都是不可变并且可序列化的。调用方可以修改已赋值数组枚举成员的访问器返回的数组；这不会对其他调用方返回的数组产生任何影响。

如果此接口中的方法返回的注释（直接或间接地）包含一个已赋值的 Class 成员，该成员引用了一个在此 VM 中不可访问的类，则试图通过在返回的注释上调用相关的类返回的方法来读取该类，将导致一个 TypeNotPresentException。

类似地，如果注释中的枚举常量不再以枚举类型存在，那么试图读取一个已赋值的枚举成员将导致一个 EnumConstantNotPresentException。

**所有已知实现类**
`AccessibleObject`, `Class`, `Constructor`, `Field`, `Method`, `Package`

* 方法1：`<T extends Annotation> T getAnnotation(Class<T> annotationClass)`: 返回该元素上存在的、指定类型的注解，如果该类型注解不存在，则返回null。
* 方法2：`Annotation[] getAnnotations()`:返回该元素上存在的所有注解。
* 方法3：`boolean is AnnotationPresent(Class<?extends Annotation> annotationClass)`:判断该元素上是否包含指定类型的注解，存在则返回true，否则返回false.
* 方法4：`Annotation[] getDeclaredAnnotations()`：返回直接存在于此元素上的所有注释。与此接口中的其他方法不同，该方法将忽略继承的注释。（如果没有注释直接存在于此元素上，则返回长度为零的一个数组。）该方法的调用者可以随意修改返回的数组；这不会对其他调用者返回的数组产生任何影响。

![导图来自网络]({{ site.annotation | prepend: site.baseurl }})

##### 
```java
@AnnotationA(value = "aa",lValues = {1,2,3})
public class AnnotationDemo extends AnnotationSub {
    public static void main(String[] args) throws ClassNotFoundException {
        Class<?> clazz = AnnotationDemo.class;
        Annotation[] annotations =clazz.getAnnotations();//返回此元素上存在的所有注解，包括从父类继承的
        System.out.println("an:"+ Arrays.toString(annotations));

        Arrays.stream(annotations).forEach(System.out::println);

        AnnotationA annotationA = clazz.getAnnotation(AnnotationA.class);//该元素如果存在指定类型的注解，则返回这些注解，否则返回 null。
        System.out.println(annotationA);

        Annotation[] annotations1 = clazz.getDeclaredAnnotations();//返回直接存在于此元素上的所有注解不包括继承
        Arrays.stream(annotations1).forEach(System.out::println);

        // Java 8中新增加的元注解@Repeatable，它表示在同一个位置重复相同的注解
        // Java8中 ElementType 新增两个枚举成员，TYPE_PARAMETER 和 TYPE_USE,新增的TYPE_PARAMETER可以用于标注类型参数，
        // 而TYPE_USE则可以用于标注任意类型(不包括class),TYPE_PARAMETER用来支持在Java的程序中做强类型检查.配合第三方插件工具,可以在编译的时候检测出runtimeerror
    }

}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface AnnotationA {
    // 1，默认是 public abstract
    // 2, 同时可选择使用default提供默认值,public abstract String value() default "";
    String value();
    // 3, 元素数据类型,可以是所有基本类型（int,float,boolean,byte,double,char,long,short），String，Class，enum，Annotation，及其类型的数组
    boolean hasNext() default false;
    enum Status {RED,BLUE}
    Class<?> testCase() default String.class;
    long[] lValues();
    AnnotationB annotationB() default @AnnotationB(value="AnnotationB",hasTrue=true);
}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Repeatable(AnnotationC.class)
@Inherited
public @interface AnnotationB {
    boolean hasTrue() default false;
    String value() default "";
}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface AnnotationC {
    AnnotationB [] value();
}

@AnnotationB("23")
@AnnotationB("231")
public class AnnotationSub {
    //
}

```