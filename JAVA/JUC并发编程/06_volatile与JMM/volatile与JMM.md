# 6. volatile与JMM

## 6.1 被volatile修饰的变量有两大特点

- 特点：
  - 可见性
  - 有序性：有排序要求，有时需要<font color='red'>禁重排序</font>
- 内存语义：
  - 当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值<font color='red'>立即刷新回主内存中</font>
  - 当读一个volatile变量时，JMM会把该线程对应的本地内存设置为无效，重新回到主内存中读取最新共享变量的值
  - 所以volatile的写内存语义是直接刷新到主内存中，读的内存语义是直接从主内存中读取
- volatile凭什么可以保证可见性和有序性？
  - 内存屏障Memory Barrier

## 6.2 内存屏障（面试重点必须拿下）

### 6.2.1 生活case

- 没有管控，顺序难保
- 设定规则，禁止乱序---->上海南京路武警当红灯
- 再说volatile两大特性：
  - **可见**：写完后<font color='red'>立即刷新回主内存并及时发出通知</font>，大家可以去主内存拿最新版，前面的修改对后面所有的线程可见
  - **有序性（禁重排）**：
    - 重排序是指编译器和处理器为了优化程序性能而对指令序列进行重新排序的一种手段，有时候会改变程序语句的先后顺序，若不存在数据依赖关系，可以进行重排序；<font color='red'>存在数据依赖关系，禁止重排序</font>；但重排序后的指令绝对不能改变原有的串行语义！<font color='red'>这点在并发设计中必须要重点考虑</font>

### 6.2.2 是什么

内存屏障（也称内存栅栏，屏障指令等）是一类同步屏障指令，是CPU或编译器在对内存随机访问的操作中的一个同步点，使得此点之前的所有读写操作都执行后才可以开始执行此点之后的操作，避免代码重排序。内存屏障其实就是一种JVM指令，Java内存模型的重排序规则会<font color='red'>要求Java编译器在生成JVM指令时插入特定的内存屏障指令</font>，通过这些内存屏障指令，volatile实现了Java内存模型中的可见性和有序性（禁重排），<font color='red'>但volatile无法保证原子性</font>

- <font color='blue'>内存屏障之前</font>的所有<font color='red'>写操作</font>都要<font color='red'>回写到主内存</font>
- <font color='blue'>内存屏障之后</font>的所有<font color='red'>读操作</font>都能获得内存屏障之前的所有写操作的最新结果（实现了可见性）

> <font color='blue'>写屏障(Store Memory Barrier)：</font>告诉处理器在写屏障之前将所有存储在缓存(store buffers)中的数据同步到主内存，也就是说当看到Store屏障指令，就必须把该指令之前的所有写入指令执行完毕才能继续往下执行
>
> <font color='blue'>读屏障(Load Memory Barrier)：</font>处理器在读屏障之后的读操作，都在读屏障之后执行。也就是说在Load屏障指令之后就能够保证后面的读取数据指令一定能够读取到最新的数据

因此重排序时，不允许把内存屏障之后的指令重排序到内存屏障之前。<font color='red'>一句话：对于一个volatile变量的写，先行发生于任意后续对这个volatile变量的读，也叫写后读</font>。

### 6.2.3 内存屏障分类

粗分两种：

- 读屏障（Load Barrier）：在读指令之前插入读屏障，让工作内存或CPU告诉缓存当中的缓存数据失效，重新回到主内存中获取最新数据。
- 写屏障（Store Barrier）：在写指令之后插入写屏障，强制把缓冲区的数据刷回到主内存中。

细分四种：

| 屏障类型   | 指令示例                 | 说明                                                         |
| :--------- | :----------------------- | :----------------------------------------------------------- |
| LoadLoad   | Load1;LoadLoad;Load2     | 保证Load1的读取操作在Load2及后续读取操作之前执行             |
| StoreStore | Store1;StoreStore;Store2 | 在store2及其后的写操作执行前，保证Store1的写操作已经刷新到主内存 |
| LoadStore  | Load1;LoadStore;Store2   | 在Store2及其后的写操作执行前，保证Load1的读操作已经结束      |
| StoreLoad  | Store1;StoreLoad;Load2   | 保证Store1的写操作已经刷新到主内存后，Load2及其后的读操作才能执行 |

### 6.2.4 困难内容

- 什么叫保证有序性？——>通过内存屏障禁止重排序
  - 重排序有可能影响程序的执行和实现，因此，我们有时候希望告诉JVM不必自动重排序。
  - 对于编译器的重排序，JMM会根据重排序的规则，禁止特定类型的编译器重新排序
  - 对于处理器的重排序，Java编译器在生成指令序列的适当位置，<font color='red'>插入内存屏障指令，来禁止特定类型的处理器排序</font>。
- happens-before之volatile变量规则

| 第一个操作 | 第二个操作：普通读写 | 第二操作：volatile读 | 第二个操作：volatile写 |
| ---------- | -------------------- | -------------------- | ---------------------- |
| 普通读写   | 可以重排             | 可以重排             | 不可以重排             |
| volatile读 | 不可以重排           | 不可以重排           | 不可以重排             |
| volatile写 | 可以重排             | 不可以重排           | 不可以重排             |

> 当第一个操作为volatile读时，不论第二个操作是什么，都不能重排序，这个操作保证了<font color='red'>volatile读之后</font>的操作不会被重排到volatile读之前
>
> 当第一个操作为volatile写时，第二个操作为volatile读时，不能重排
>
> 当第二个操作为volatile写时，不论第一个操作是什么，都不能重排序，这个操作保证了<font color='red'>volatile写之前</font>的操作不会被重排到volatile写之后

- JMM将内存屏障插入策略分为4种规则

  - 读屏障：在每个volatile读操作的**后面**插入一个LoadLoad屏障或者LoadStore屏障

  ![image.png](volatile与JMM.assets/1681203343902-7e14ea6b-094f-4f89-838a-a8cd0fc48d94.png)

  - 写屏障：在每个volatile写操作的前面插入StoreStore屏障；在每个volatile写操作的后面插入StoreLoad屏障

  ![image.png](volatile与JMM.assets/1681203485651-b5a20886-997a-43cb-8a30-6f473a59d2f1.png)

## 6.3 volatile特性

### 6.3.1 保证可见性

<font color='red'>保证不同线程对某个变量完成操作后结果及时可见，即该共享变量一旦改变所有线程立即可见</font>

- Code

  - 不加volatile，没有可见性，程序无法停止
  - 加了volatile，保证可见性，程序可以停止

  ```java
  public class VolatileSeeDemo {
  
      /**
       * t1	-------come in
       * main	 修改完成
       * t1	-------flag被设置为false，程序停止
       */
      static volatile boolean flag = true;
  
      public static void main(String[] args) {
          new Thread(() -> {
              System.out.println(Thread.currentThread().getName() + "\t-------come in");
              while (flag) {
  
              }
              System.out.println(Thread.currentThread().getName() + "\t-------flag被设置为false，程序停止");
          }, "t1").start();
  
          try {
              TimeUnit.SECONDS.sleep(2);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
  
          //更新flag值
          flag = false;
  
          System.out.println(Thread.currentThread().getName() + "\t 修改完成");
      }
  }
  ```

- volatile变量的读写过程（了解即可）

![image.png](volatile与JMM.assets/1681207881381-37c38d15-7c78-486c-9d10-4a4aaa6b4da4.png)

### 6.3.2 没有原子性

volatile变量的复合操作不具有原子性

- 对于volatile变量具备可见性，JVM只是保证从主内存加载到线程工作内存的值是最新的，<font color='red'>也仅仅是数据加载时是最新的</font>。但是多线程环境下，”数据计算“和”数据赋值“操作可能多次出现，若数据在加载之后，若主内存volatile修饰变量发生修改之后，线程工作内存的操作将会<font color='red'>作废</font>去读取主内存最新值，操作出现写丢失问题。<font color='red'>即各线程私有内存和主内存公共内存中变量不同步</font>，进而导致数据不一致。由此可见volatile解决的是变量读取时的可见性问题，<font color='red'>但无法保证原子性，对于多线程修改主内存共享变量的场景必须加锁同步</font>。

- 至于怎么去理解这个写丢失的问题，就是在将数据读取到本地内存到写回主内存中有三个不走：数据加载——>数据计算——>数据赋值，<font color='red'>如果第二个线程在第一个线程读取旧值与写回新值期间读取共享变量的值，那么第二个线程将会与第一个线程一起看到同一个值</font>，并执行自己的操作，一旦其中一个线程对volatile变量先行完成操作刷回主内存后，另一个线程会作废自己的操作，然后重新去读取最新的值再进行操作，这样的话，它自身的那一次操作就丢失了，这就造成了 线程安全失败，因此，这个问题需要使用synchronized修饰以保证线程安全性。

- 结论：volatile变量不适合参与到依赖当前值的运算，如i++，i=i+1之类的操作，通常用来保存某个状态的<font color='red'>boolean或者int值</font>，也正是由于volatile变量只能保证可见性，在不符合以下规则的运算场景中，我们仍要通过加锁来保证原子性：

  - 运算结果并不依赖变量的当前值，或者能够确保只有单一的线程修改变量的值
  - 变量不需要与其他的状态变量共同参与不变约束

- volatile为什么不具备原子性：举例i++的例子，在字节码文件中，i++分为三步，间隙期间不同步非原子操作

  - 对于volatile变量，JVM只是保证从主内存加载到线程工作内存的值是最新的，也就是数据加载时是最新的，如果第二个线程在第一个线程读取旧值和写回新值期间读取i的域值，也就造成了线程安全问题。

  ![image.png](volatile与JMM.assets/1681275465892-ca223f7f-6078-409a-95fa-02613915ab71.png)

### 6.3.3 指令禁止重排序

- 在每一个volatile写操作<font color='red'>前面</font>插入一个StoreStore屏障——StoreStore屏障可以保证在volatile写之前，其前面所有的普通写操作都已经刷新到主内存中。
- 在每一个volatile写操作<font color='red'>后面</font>插入一个StoreLoad屏障——StoreLoad屏障的作用是避免volatile写与后面可能有的volatile读/写操作重排序
- 在每一个volatile读操作<font color='red'>后面</font>插入一个LoadLoad屏障——LoadLoad屏障用来禁止处理器把上面的volatile读与下面的普通读重排序
- 在每一个volatile读操作<font color='red'>后面</font>插入一个LoadStore屏障——LoadTore屏障用来禁止处理器把上面的volatile读与下面的普通写重排序

## 6.4 如何正确使用volatile

- 单一赋值可以，但是含复合运算赋值不可以（i++之类的）

  - volatile int a = 10；
  - volatile boolean flag = true；

- 状态标志，判断业务是否结束

  - 作为一个布尔状态标志，用于指示发生了一个重要的一次性事件，例如完成初始化或任务结束

  ![image.png](volatile与JMM.assets/1681277553116-c8cc74c4-47aa-4398-9262-8dc8b25d917a.png)

- 开销较低的读，写锁策略

  - 当读远多于写，结合使用内部锁和volatile变量来减少同步的开销

  - 原理是：利用volatile保证读操作的可见性，利用synchronized保证符合操作的原子性

    ![image.png](volatile与JMM.assets/1681277613302-5adef887-27ed-449c-bd88-0e4f4e6b56bf.png)

- DCL双端锁的发布

  - 问题描述：首先设定一个加锁的单例模式场景

    ![image.png](volatile与JMM.assets/1681277769628-287b69ba-cc2f-4a20-b193-d255570d5dfa-2590288.png)

  - <font color='blue'>在单线程环境下（或者说正常情况下）</font>，在”问题代码处“，会执行以下操作，保证能获取到已完成初始化的实例：

  ![image.png](volatile与JMM.assets/1681277858260-07127c5a-8bb5-4af3-a253-149085d4b849.png)

  - 隐患：<font color='blue'>在多线程环境下</font>，在”问题代码处“，会执行以下操作，由于重排序导致2、3乱序，后果就是其他线程得到的是null而不是完成初始化的对象，其中第3步中实例化分多步执行（分配内存空间、初始化对象、将对象指向分配的内存空间），某些编译器为了性能原因，会将第二步和第三步重排序，这样某个线程可能会获得一个未完全初始化的实例：

  ![image.png](volatile与JMM.assets/1681277949781-4b436fdd-4cac-43b9-9089-0dfa91f4d812.png)

  - 多线程下的解决方案：加volatile修饰

  ![image.png](volatile与JMM.assets/1681278158734-fe21f869-71b1-4b80-a457-011b7b1832b0.png)

## 6.5 3句话总结

- volatile写之前的操作，都禁止重排序到volatile之后
- volatile读之后的操作，都禁止重排序到volatile之前
- volatile写之后volatile读，禁止重排序
