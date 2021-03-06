<h2 align="center"> jvm总结</h2>
#### 1.Java内存区域

---

#### 2.Java垃圾回收算法和垃圾回收器

**G1回收器**:将堆分为一个个大小相等的小块(Region)，新生代老年代不需要连续，G1 使用了**停顿预测模型**来满足用户指定的停顿时间目标，并基于目标来选择进行垃圾回收的区块数量。G1 采用增量回收的方式，每次回收一些区块，而不是整堆回收。

> [Java Hotspot G1 GC的一些关键技术](https://tech.meituan.com/2016/09/23/g1.html)

---

#### 3.变量初始化顺序

---

#### 4.类加载机制和顺序，双亲加载机制

- **类加载过程：**

  - **加载**：通过类的全限定名来获取定义此类的二进制字节流，将这个字节流结构代表的静态存储文件转化为方法区的运行时数据结构，在内存中生成一个此类的Class对象（HotSpot保存在方法区里面），作为方法区这个类各种数据的访问入口

  - **连接**：

    - **验证**：确保Class文件的信息符合当前虚拟机的要求：文件格式验证（版本号，魔数开头），元数据验证（是否有父类，是否继承final类），字节码验证（不会跳转到方法体以外的字节码指令上），符号引用验证（发生在解析阶段，即符号引用转为直接引用的时候）

    - **准备**：为类变量（仅static变量，实例变量将在对象实例化时在堆中分配内存）分配内存，设置类变量初始值，这些变量使用的内存都位于方法区，public static int v = 12，此时分配为默认值0

    - **解析**：将符号引用转换为直接引用（Class文件格式中的CONSTANT_Class_info等）

      > 符号引用：以一组符号描述所引用的目标，符号引用的字面量形式定义在虚拟机规范的Class文				   件格式中
      >
      > 直接引用：可以是直接指向目标的指针或是偏移量，或是能间接定位到目标的句柄，例如解析一个类或接口的直接引用

  - **初始化**：执行类构造器<clinit>()方法，执行静态代码块，为类变量赋值

- **类加载器**：通过类的全限定名来获取此类的二进制字节流，实现这个操作的代码就叫类加载器，

  - **分类**：
    - 启动类加载器（Bootstrap ClassLoader）：C++实现，是虚拟机的一部分，加载/lib 目录下的，被虚拟机识别的类库，如rt.jar
    - 扩展类加载器（Extension ClassLoader）：负责加载/lib/ext目录下的类库
    - 应用程序类加载器（Application ClassLoader）: 也称系统类加载器，负责加载用户类路径上指定的类库
  - **双亲委派模型**：一个类收到了类加载请求首先委派给父类加载器，具备了一种优先级的层次关系，自定义java.lang.开头的类加载时会直接抛出异常

---

#### 5.Java中堆栈的区别。堆栈的增长方向有哪些不同

堆是由低地址向高地址增加，栈是由高地址向低地址增加

---

#### 6.垃圾回收时如何解决循环引用问题

引用计数法存在循环引用问题，可达性分析法不存在这种问题，通过GCRoot通过引用链来确定一个对象是否可以被回收

---

