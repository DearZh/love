# 先普及一个基本概念：Java中基本数据类型的装箱和拆箱操作
## 自动装箱
在JDK5以后，我们可以直接使用Integer num = 2；来进行值的定义，但是你有没有考虑过？Integer是一个对象呀，为什么我可以不实例化对象，就直接来进行Value的定义呢？

一般情况下我们在定义一个对象的时候，顶多赋值为一个null 即空值；
比如：Person pserson = null；但是肯定不可以Person person =2；这样操作吧，
那为什么Integer，Float，Double，等基本数据类型的包装类是可以直接定义值的呢？

究其原因无非是编译器在编译代码的时候，重新进行了一次实例化的操作而已啦：
比如当我们使用Integer num = 2 的时候，在JVM运行前的编译阶段，此时该Integer num = 2 将会被编译为
Integer num = new Integer(2); 那么此时编译后的这样一个语法 new Integer(2) 则是符合JDK运行时的规则的，而这种操作就是所谓的装箱操作；

注意：（不要拿Integer和int类型来进行对比，int，float，这些是JDK自定义的关键字，
本身在编译的时候就会被特殊处理，而Integer，Float，Double等则是标准的对象，对象的实现本身就是要有new 的操作才是合理；
所以对于这些基本类型的包装类在进行 Integer num = 2的赋值时，则的确是必须要有一个装箱的操作将其变成对象实例化的方式这样也才是一个标准的过程；）
## 自动拆箱
那么当你了解了对应的装箱操作后，再来了解一下对应拆箱的操作：

当我们把一个原本的Integer num1 = 2; 来转换为 int num1 = 2的时候实际上就是一个拆箱的操作，及把包装类型转换为基本数据类型时便是所谓的拆箱操作；
一般当我们进行对比的时候，编译器便会优先把包装类进行自动拆箱：如Integer num1 = 2 和 int num2 = 2；当我们进行对比时 
if(num1 == num2) 那么此时编译器便会自动的将包装类的num1自动拆箱为int类型进行对比等操作；

## 装箱及拆箱时的真正步骤
上述已经说过了自动装箱时，实际上是把 Integer num =2 编译时变更为了 Integer num = new Integer(2)；
但实际上JDK真的就只是这么简单的进行了一下new的操作吗？当然不是，在自动装箱的过程中实际上是调用的Integer的valueOf(int i)的方法，来进行的装箱的操作；
我们来看一下这个方法的具体实现：我会直接在下述源码中加注释，直接看注释即可
```
    public static Integer valueOf(int i) {
            //在调用valueOf进行自动装箱时，会先进行一次所传入值的判断，当i的值大于等于IntegerCache.low 以及 小于等于IntegerCache.high时，则直接从已有的IntegerCache.cache中取出当前元素return即可；
            if (i >= IntegerCache.low && i <= IntegerCache.high){
                            return IntegerCache.cache[i + (-IntegerCache.low)];
            }
            //否则则直接new Integer(i) 实例化一个新的Integer对象并return出去；
            return new Integer(i);
    }
    //此时我们再看一下上述的IntegerCache到底是做的什么操作，如下类：（注意：此处IntegerCache是 private 内部静态类，所以我们定义的外部类是无法直接使用的，此处看源码即可）
    
    private static class IntegerCache {
            //定义一个low最低值 及 -128；
            static final int low = -128;
            //定义一个最大值（最大值的初始化详情看static代码块）
            static final int high;
            //定义一个Integer数组，数组中存储的都是 new Integer()的数据；（数组的初始化详情看static代码块）
            static final Integer cache[];
    
            static {
                //此处定义一个默认的值为127；
                int h = 127;
                //sun.misc.VM.getSavedProperty() 表示从JVM参数中去读取这个"java.lang.Integer.IntegerCache.high"的配置，并赋值给integerCacheHighPropValue变量
                String integerCacheHighPropValue = sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
                //当从JVM中所取出来的这个java.lang.Integer.IntegerCache.high值不为空时
                if (integerCacheHighPropValue != null) {
                    try {
                        //此处将JVM所读取出的integerCacheHighPropValue值进行parseInt的转换并赋值给 int i;
                        int i = parseInt(integerCacheHighPropValue);
                        //Math.max()方法含义是，当i值大于等于127时，则输出i值，否则则输出 127；并赋值给 i；
                        i = Math.max(i, 127);
                        //Math.min()则表示，当 i值 小于等于 Integer.MAX_VALUE时，则输出 i，否则输出 Integer.MAX_VALUE，并赋值给 h
                        //此处使用：Integer.MAX_VALUE - (-low) -1 的原因是由于是从负数开始的，避免Integer最大值溢出，所以这样写的，此处可以先不考虑
                        h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                    } catch( NumberFormatException nfe) {
                        // If the property cannot be parsed into an int, ignore it.
                    }
                }
                //最后把所得到的最终结果 h 赋值给我们亲爱的 high 属性；
                high = h;
    
                //以下赋值当前cache数组的最大长度；
                cache = new Integer[(high - low) + 1];
                int j = low;
                //然后进行cache数组的初始化循环；
                for(int k = 0; k < cache.length; k++)
                    //注意：此处new Integer(j++);是先实例化的j，也就是负数-128，所以也才会有上述的Integer.MAX_VALUE - (-low) -1)的操作，因为数组中存储的是 -128 到 high 的所有实例化数据对象；
                    cache[k] = new Integer(j++);
    
                // range [-128, 127] must be interned (JLS7 5.1.7)
                assert IntegerCache.high >= 127;
            }
    
            private IntegerCache() {}
        }
```
朋友们，由上述的代码我们便可以知道，自动装箱时：

1、high的值如果未通过JVM参数定义时则默认是127，当通过JVM参数进行定义后，则使用所定义的high值，前提是不超出(Integer.MAX_VALUE - (-low) -1)的长度即可，如果超出这个长度则默认便是：Integer.MAX_VALUE - (-low) -1；

2、默认情况下会存储一个 -128 到 high的 Integer cache[]数组，并且已经实例化了所有 -128 到high的Integer对象数据；

3、当使用valueOf(int i)来自动装箱时，会先判断一下当前所需装箱的值是否(大于等于IntegerCache.low && 小于等于IntegerCache.high) 如果是，则直接从当前已经全局初始化好的cache数组中返回即可，如果不是则重新 new Integer();


**而当Integer对象在自动拆箱时则是调用的Integer的intValue()方法，方法代码如下：可以看出是直接把最初的int类型的value值直接返回了出去，并且此时返回的只是基本数据类型！**
```
    private final int value;

    public int intValue() {
        return value;
    }
```



**所以，朋友们，让我们带着上述的答案，来看下我们常在开发代码时碰到的一些问题：（请接着向下看哦，因为最后还会再涉及到一些JVM的说明哦）**

## Integer于Int进行==比较时的代码案例

``` 
 public static void main(String[] args) {
        Integer num1 = 2000;
        int num2 = 2000;
        //会将Integer自动拆箱为int比较，此处为true；因为拆箱后便是 int于int比较，不涉及到内存比较的问题；
        System.out.println(num1 == num2);
        Integer num3 = new Integer(2000);
        Integer num4 = new Integer(2000);
        //此处为false，因为 num3 是实例化的一个新对象对应的是一个新的内存地址，而num4也是新的内存地址；
        System.out.println(num3 == num4);
        Integer num5 = 100;
        Integer num6 = 100;
        //返回为true，因为Integer num5 =100的定义方式，会被自动调用valueOf()进行装箱；而valueOf()装箱时是一个IntegerCache.high的判断的，只要在这个区间，则直接return的是数组中的元素
        //而num5 =100 及返回的是数组中下标为100的对象，而num6返回的也是数组中下标为 100的对象，所以两个对象是相同的对象，此时进行 == 比较时，内存地址相同，所以为true
        System.out.println(num5 == num6);
        Integer num7 = new Integer(100);
        Integer num8 = 100;
        //结果为false；为什么呢？因为num7并不是自动装箱的结果，而是自己实例化了一个新的对象，那么此时便是堆里面新的内存地址，而num8尽管是自动装箱，但返回的对象与num7的对象也不是一个内存地址哦；
        System.out.println(num7 == num8);
    }
```
## 总结

* 1、由于我们在使用Integer和int进行==比较时，存在着自动拆箱于装箱的操作，所以在代码中进行Integer的对比时尽可能的使用 .equals()来进行对比；
比如我们定义如下一个方法：那么我们此时是无法知晓num1 和num2的值是否是直接new出来的？还是自动装箱定义出来的？就算两个值都是自动装箱定义出来的，那么num1 和num2的值是否超出了默认的-128到127的cache数组缓存呢？如果超出了
那么还是new 的Integer()，此时我们进行 == 对比时，无疑是风险最大的，所以最好的还是 .equals()进行对比；除非是拿一个Integer和一个int基本类型进行对比可以使用==，因为此时无论Integer是新new实例化的还是自动装箱的，在对比
时都会被自动拆箱为 int基本数据类型进行对比；
```
    public void test(Integer num1,Integer num2){
        //TODO
    }
```
* 2、合理的在项目上线后，使用-XX:AutoBoxCacheMax=20000 参数来定义自动装箱时的默认最大high值，可以很好的避免基本数据类型包装类被频繁堆内创建的问题；什么个意思呢，一般情况下我们在项目开发过程中，会大量使用
Integer num = 23;等等的代码，并且我们在操作数据库的时候，一般返回的Entity实体类里面也会定义一大堆的Integer类型的属性，而上述也提到过了，每次Integer的使用实际上都会被自动装箱，对于超出-128和127的值
则会被创建新的堆对象；所以如果我们有很多的大于127的数据值，那么每次都需要在堆中创建临时对象岂不是一个很可惜的操作吗，如果我们在项目启动时设置-XX:AutoBoxCacheMax=20000，那么对于我们常用的Integer为2W以下的
数字，则直接从IntegerCache 数组中直接取就行了，完全就没必要再创建临时的堆对象了嘛；这样对于整个JVM的GC回收来说，多多少少也是一些易处呀，避免了大量的重复的Integer对象的创建占用和回收的问题呢；不是嘛

* 3、之前在本人还是初初初级，初出茅庐程序猿的时候，就经常听到有的人说，JVM中关于-128到127的cache缓存是存在常量池里面的，有的人说当你在定义int类型时实际上是存储在栈里面的，搞的我也是很尴尬呀；很难抉择，
那么现在呢，就给出一个最终的本人总结后的答案，如下：

首先我们看了上述自动装箱的源码以后，可以知道，初始化的缓存数据是定义在静态属性中的：static final Integer cache[]; 所以，答案是：我们自动装箱的cache数组缓存的确是定义在常量池中的；每次我们自动装箱时的数组判断，的确是从常量池中拿的数据，
废话，因为是 static final 类型的呀，所以当然是常量池中存储的cache数组啦

但是：关于int类型中定义的变量实际上是存储于栈空间的，这个也是没错的，因为关于JVM栈中有一个定义是：针对局部变量中的基本类型的字面量则是存储在线程栈中的；（栈是线程的一个数据结构），
所以对于我们在方法中定义的局部变量：int a = 3 时，则的确是存储在线程栈中的；而我们在方法中定义局部变量 Integer a=300时，这个肯定是在堆或者常量池中啦（看是否自动装箱后使用常量池中cache）；

而对于我们在类中定义的成员属性来说，比如：static int a =3;此时则是在常量池中（无外乎什么类型因为他是静态的，所以常量池）而类的成员属性 int a=3（则是在堆中，无外乎什么属性，普通变量所对应的对象内存都是堆中）




