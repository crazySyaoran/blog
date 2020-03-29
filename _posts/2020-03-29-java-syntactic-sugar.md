---
layout: post
title: Java语法糖
date: 2020-03-29
Author: Syaoran
tags: [java, note]
comments: true
---

语法糖（Syntactic Sugar），也称糖衣语法，是由英国计算机学家 Peter.J.Landin 发明的一个术语，指在计算机语言中添加的某种语法，这种语法对语言的功能并没有影响，但是更方便程序员使用。Java 中最常用的语法糖主要有泛型、变长参数、条件编译、自动拆装箱、内部类等。

### Java泛型

**类的类型的参数化，就是泛型。** 或者说： **类的类型也可以作为参数变量传递给类或类的方法。**  
java1.5后可以用尖括号`<>`声明一个泛型。

Oracle官方的Documentation中对泛型（Generic Types）的解释：
> A generic type is a generic class or interface that is parameterized over types.  

官方给出了以下例程：
```
/**
 * Generic version of the Box class.
 * @param <T> the type of the value being boxed
 */
public class Box<T> {
    // T stands for "Type"
    private T t;

    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
```
为了便于阅读，在Java中有人使用 T E K V 等字母。这些对编译器来说都是一样的，可以是任意字母。
> T ： Type （类型） E :  Element（元素） K ： Key（键） V ： Value（值）  

泛型的类型参数只能是引用类型，不能是原始类型（int,char,double,long等）。

另外尖括号`<>`中可以使用`?`通配符，`extends`和`super`关键字：  
```
// `?`通配符可以用来表示任意类型
List<?> barList;

// 限定上边界
public <T extends Comparable<T>> T maximum(T x, T y, T z) {}

// 限定下边界
public class Student<? super Integer> {}

```

另外，<>中可以加入多个类型参数（Type Parameters），官方给出了以下例程：  
```
public interface Pair<K, V> {
    public K getKey();
    public V getValue();
}

public class OrderedPair<K, V> implements Pair<K, V> {

    private K key;
    private V value;

    public OrderedPair(K key, V value) {
    this.key = key;
    this.value = value;
    }

    public K getKey()   { return key; }
    public V getValue() { return value; }
}

// The following statements create two instantiations of the OrderedPair class:
Pair<String, Integer> p1 = new OrderedPair<String, Integer>("Even", 8);
Pair<String, String>  p2 = new OrderedPair<String, String>("hello", "world");
```  

> Java中的参数化类型只在源码中存在，在编译后的字节码中，已经被替换为原来的原生类型了，并且在相应的地方插入了强制转换代码。对于运行期的Java 语言来说，Integer型的ArrayList和String型的ArrayList就是同一个类。所以说Java语言中的泛型实现方法称为类型擦除，基于这种方法实现的泛型称为伪泛型。
> 
```
(new ArrayList<Integer>()).getClass() == (new ArrayList<String>()).getClass()
```


### Java自动装箱和拆箱

在Java SE5之前，如果要生成一个数值为10的Integer对象，必须这样进行
```
Integer i = new Integer(10);
```
而在从Java SE5开始就提供了自动装箱的特性，如果要生成一个数值为10的Integer对象，只需要这样就可以了：
```
Integer i = 10;
```
这个过程中会自动根据数值创建对应的Integer对象，这就是**装箱**。跟装箱对应，**拆箱**就是自动将包装器类型转换为基本数据类型
```
原始类型{boolean, byte, char,      float, int,       long, short, double}
包装类型{Boolean, Byte, Character, Float, iInterger, long, Short, Double}
```
实际上是由编译器对这个过程自动进行了翻译：
```
// 装箱源代码
int i = 10; 
Integer n = i;

// 反编译后
int i = 10; 
Integer n = Integer.valueOf(i); 

// 拆箱源代码
Integer i = 10; 
int n = i; 

// 反编译后
Integer i = Integer.valueOf(10); 
int n = i.intValue(); 
```
装箱过程是通过调用包装器的`valueOf`方法实现的，而拆箱过程是通过调用包装器的`xxxValue`方法实现的。

装拆箱操作会在以下情况时发生：
- 进行 = 赋值操作（装箱或拆箱）
- 进行+，-，*，/混合运算 （拆箱）
- 进行>,<,==比较运算（拆箱）
- 调用equals进行比较（装箱）
- ArrayList,HashMap等集合类 添加基础类型数据时（装箱）

然后就引出了一道非常有意思的Java面试题：
```
public void testAutoBox2() {
	//1
    int a = 100;
    Integer b = 100;
    System.out.println(a == b);
      
    //2
    Integer c = 100;
    Integer d = 100;
    System.out.println(c == d);
     
    //3   
    c = 200;
    d = 200;
    System.out.println(c == d);
}
```
第一个因为基础类和包装类进行比较，包装类会被拆箱成基础类，基础类int的比较是值的比较，因此是true
> 包装类的比较是引用的比较，因此例如`new Integer(1) == new Integer(1)`的结果是false

第二个和第三个的结果很有意思，第二个的结果是true，而第三个的结果是false。
原因和Integer.valueOf()的实现有关：
> MAC版IDEA使用`Command+左键点击方法名`可以跳转到源码   

```
public final class Integer extends Number implements Comparable<Integer> {
// ...
    private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }  
    public static Integer valueOf(int i) {  
        if (i >= IntegerCache.low && i <= IntegerCache.high)  
            return IntegerCache.cache[i + (-IntegerCache.low)];  
        return new Integer(i);  
    }  
// ...  
}  
```

可以看出，包装器Integer初始化了一个叫做IntegerCache的静态数组，这样当需要产生一个在[-128, 127]之间的Integer对象时，无需调用Integer的构造函数，只需从IntegerCache[]中取出对应元素就可以了，这样节省了空间和调用构造函数的时间，只有在需要产生一个不在[-128, 127]区间的Integer才会单独生成一个Integer对象。
因此题目中值为100的Integer都是IntegerCache[]中的同一个元素，因此是相同的引用；而值为100的Integer是新生成的对象，是不同的引用。






























