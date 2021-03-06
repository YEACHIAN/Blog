### Java面向对象

对于一个类来说，一般有三种常见的成员：

* 属性 field
* 方法 method
* 构造器 constructor

### JVM三个内存区

* 栈区 stack
  * 方法执行：每个方法的调用都创建一个栈帧
  * 线程管理：每个线程创建一个栈 
  * 栈是线程私有的
  * 连续的内存空间，系统自动分配，速度快
* 堆区 heap
  * 存放创建好的对象（如数组）
  * JVM只有一个堆（线程共享）
  * 不连续的内存空间，分配灵活，速度慢
  * 只要是在堆里面的**默认值就是0**
* 方法区 method area（静态堆）
  * JVM只有一个方法区（线程共享）
  * 方法区也是堆，只用来存放不变或者唯一的内容
    * 类信息（Class对象）
    * 静态变量
    * 字符串常量

### 构造方法

* 也叫构造器，constructor，用于创建和初始化对象
* 构造方法通过 new 关键字调用
* 构造方法不能定义返回值类型
  * 构造方法内不能使用可以单独使用 return 结束方法 但是不能让 return 返回值
  * 因为返回值是确定的，就是指定类的对象
* 如果没有定义构造器，编译器会定一个无参的构造方法（反之相反）
  * 如果自己定义，编译器就不会自动添加
* 构造器的方法名须和类名一致
* 构造方法的第一句总是 **super()**

### 垃圾回收 Garbage Collection

* 发现无用对象
* 回收无用对象占用的空间

#### 垃圾回收算法

* 引用计数法
  * 缺点：循环引用的无用对象
* 引用可达法（搜索算法）
  * 把引用关系看做一张图
  * 从 GC ROOT 节点开始，寻找对应的引用节点
  * 找到节点后，继续寻找该节点的引用节点
  * 所有的引用节点找到后，剩余的节点就是无用节点（没有被引用）

```java
// 循环引用
public class Student {
    String name;
    Student friend;
    
    public static void main (String[] args) {
        Student s1 = new Student();
        Student s2 = new Student();
        
        s1.friend = s2;
        s2.friend = s1;
        
        s1 = null;
        s2 = null;
    }
}
```

#### 分代垃圾回收

* 不同对象生命周期不一样，用不一样的回收算法
* 对象分三种状态
  * 年轻代
  * 年老代
  * 持久代
* JVM将堆内存划分为
  * Eden（年轻代，Minor GC）
  * Survivor（Major GC、Full GC）
  * Tenured/Old（年老代，Full GC）
* Java一般都是针对 Full GC 调优（尽量不要 Full GC）
  * 年老代被写满
  * 持久代（Perm）被写满
  * 显示调用 System.gc() （程序员无权调用GC，只是向JVM建议GC）
  * GC后 Heap 的分配策略动态变化

![堆内存的划分细节](D:\workspace\blog\Java\堆内存的划分细节.png)

### this的本质

* this代表当前对象（创建好的对象的地址）
* this用来区分局部变量和成员变量（同名）
* this作为方法代表构造方法
  * 构造方法不能直接调用构造方法（要通过this）
  * 调用构造方法只能放在构造方法的第一句
* this不能用在 static 方法中（静态方法没有当前对象这一说）

#### new对象分四步

* 分配空间，初始化为0
* 属性值显示初始化
* 执行构造方法（this）
* 返回对象的地址

### static关键字

* 静态成员：静态变量、静态常量、静态方法、静态块（static{}）
  * 静态块是为了初始化类（只在加载类的时候调用一次，通常是第一次new这个类时）
* 静态成员属于类，在方法区
  * 生命周期和类相同（程序的生命周期）
  * 普通成员属于对象，在堆
* 静态可以使用动态，反过来不行
* 静态成员会自然而然的继承（先从Object的静态成员开始）
  * 构造方法很像静态方法（先执行Object的构造方法）
  * 动态成员要显式的继承才能继承

### 面向对象的特征

* 继承、封装、多态
* 抽象类、接口、内部类

### 继承

* 目的是扩展类，手段是重用，解决了重复的问题
* 父类也称作超类、基类、派生类
* Java类只有单继承，接口有多继承
* 所有类默认继承 java.lang.Object 类（不使用 extends）
* 子类继承父类
  * 可以得到父类的全部属性和方法 （除了父类的构造方法）
  * 但不见得可以直接访问（父类私有的属性和方法）
  * 父类构造方法通过 super 调用
* 查看类的继承结构：Ctrl+T

#### instanceof

* 对象a是不是类B的对象 a instanceof B

### 方法重写

* 子类方法覆盖父类方法（修改父类的方法）
  * 方法名、形参子类**等于**父类（数量、类型、顺序）
  * 返回值类型，声明异常类型，子类**不大于**父类
  * 访问权限子类**不小于**父类
* 强调重写 @override