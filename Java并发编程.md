# Java并发编程

## 0.1 认识CPU、核心与线程
### CPU与核心
#### 物理核

* 物理核数量=cpu数(机子上装的cpu的数量)*每个cpu的核心数

#### 虚拟核

* 所谓的4核8线程，4核指的是物理核心。通过超线程技术，用一个物理核模拟两个虚拟核，每个核两个线程，总数为8个线程。
* 在操作系统看来是8个核，但实际上是4个物理核
* 通过超线程技术可以实现单个物理核实现线程级别的并行计算，但是性能比不上两个物理核

#### 单核cpu和多核cpu

* 都是一个cpu，不同的是每个cpu上的核心数
* 多核cpu是多个单核cpu的替代方案，多核cpu减小了体积，同时减少了功耗
* 一个核心只能同时执行一个线程

### 进程和线程

#### 理解

* 进程是操作系统进行资源(包括cpu、内存、磁盘IO等)分配的最小单位
* 线程是cpu调度和分配的基本单位
* 我们打开的微信、浏览器等都是一个进程
* 进程可能有多个子任务，比如微信要接受消息，发送消息，这些子任务就是线程
* 资源分配给进程，线程共享进程资源

#### 对比

 对比 | 进程| 线程 |
|:----:| --- | ----- | --- | --- |
|定义 |进程是程序运行的一个实体的运行过程，是系统进行资源分配和调配的一个独立单位 | 线程是进程运行和执行的最小调度单位|
|系统开销|创建撤销切换开销大，资源要重新分配和收回|仅保存少量寄存器的内容，开销小，在进程的地址空间执行代码|
|拥有资产|资源拥有的基本单位|基本上不占资源，仅有不可少的资源(程序计数器，一组寄存器和栈)|
|调度|资源分配的基本单位|独立调度分配的单位|
|安全性|进程间相互独立，互不影响|线程共享一个进程下面的资源，可以互相通信和影响|
|地址空间|系统赋予的独立的内存地址空间|由相关堆栈寄存器和线程控制表TCB组成，寄存器可被用来存储线程内的局部变量|

#### 线程切换
* cpu给线程分配时间片(也就是分配给线程的时间)，执行完时间片后会切换到另一个线程
* 切换之前会保存线程的状态，下次时间片再给这个线程时才能知道当前状态
* 从保存线程A的状态再到切换到线程B时，重新加载线程B的状态的过程叫做上下文切换
* 上下文切换时会消耗大量的cpu时间。

#### 线程开销
* 上下文切换开销
* 线程创建和消亡的开销
* 线程需要保存维持线程本地栈，会消耗内存

### 串行，并发与并行

#### 串行
* 多个任务，执行时一个执行完再执行另一个

#### 并发
* 多个线程在单个核心运行，同一时间一个线程运行，系统不停切换线程，看起来像同时运行，其实是线程不断切换

#### 并行
* 每个线程分配给独立的核心，线程同时运行

### 多核下线程数量选择

#### 计算密集型
* 程序主要为复杂的逻辑判断和复杂的运算
* cpu的利用率高，不用开太多线程，开太多线程反而会因为线程切换时切换上下文而浪费时间

#### IO密集型
* 程序主要为IO操作，比如磁盘IO(读取文件)和网络IO(网络请求)
* 因为IO操作会阻塞线程，cpu的利用率不高，可以开多点线程，阻塞时可以切换到其他就绪线程，提高cpu的利用率

### 总结
* 提高硬件水平，处理速度或核心数
* 根据场景，合理设置线程数，软件上提高cpu利用率

## 0.2 多线程技能

### 并发历史

* 没有操作系统时，一台计算机只执行一个程序
* 为了提高资源利用率、提高公平性、提高便利性，计算机加入操作系统
* 增加了线程，线程可以共享进程的资源

### 线程优势
发挥多处理器的强大功能

### 线程状态

* 新建(New): 创建尚未启动的线程
* 运行(Runable)：包括了操作系统线程中的Running和Ready，处于此状态的线程可能正在执行或等待CPU分配时间片
* 无限期等待(Waiting):等待被其他线程显式唤醒
* 限期等待(Timed Waiting):一定时间后由系统自动唤醒
* 阻塞(Blocked)：等待获取到一个排它锁
* 结束(Terminated)：线程执行结束

### 多线程编程的两种方式

* 继承Thread
* 实现Runnable接口

### 通过继承Thread

代码的执行顺序与代码的顺序无关

	public class T1 {
    public static void main(String[] args) {
        MyThread myThread=new MyThread();
        myThread.start();
        System.out.println("代码的执行结果与代码的顺序无关");
    }
	}
	class MyThread extends Thread
	{
    	public void run()
    	{
        	System.out.println("创建的线程");
    	}
	}
	
如果直接执行run方法是同步(主线程调用)，start方法是让系统来找一个时间来调用run方法(子线程调用)
	
	public class T1 {
    public static void main(String[] args) {
        MyThread myThread=new MyThread();
        myThread.run();
        System.out.println("如果是直接执行run方法，肯定是按代码顺序执行的，因为是通过主线程调用的");
    }
	}
	class MyThread extends Thread
	{
   		public void run()
    	{
        	System.out.println("创建的线程");
    	}
	}
	
### 通过实现Runnable接口

比继承Thread的方式更有优势

* Java不能多继承，所以如果线程类已经有一个父类，那么无法再继承Thread类


		public class MyRunnable implements Runnable {
    	@Override
    	public void run() {
        	System.out.println("运行中!");
    	}
		}
		public class Run {

    	public static void main(String[] args) {
        	Runnable runnable=new MyRunnable();
        	Thread thread=new Thread(runnable);
        	thread.start();
        	System.out.println("运行结束!");
    	}

		}
		
### 线程数据非共享

* 这里的i并不共享，每一个线程维护自己的i变量

### 线程数据共享

* 由于i++不是原子操作(先获取i的值，然后再加一，再把结果赋给i)，所以输出的值会有重复的情况

### synchronized关键字让i++同步执行

* synchromized关键字，给方法加上锁，多个线程尝试拿到锁，拿到锁的线程执行方法，拿不到的不断尝试获取锁

### Thread方法

* currentThread() 获得当前线程
* isLive()线程是否活跃状态
* sleep()方法，让线程休眠
* getId()方法，获得线程的唯一标示

### 停止线程的方法

* 线程自己执行完后自动终止
* stop强制终止，不安全
* 使用interrupt方法

interrupt方法

* 线程对象有一个boolean变量代表是否有中断请求，interrupt方法将线程的中断状态设置为true，单没有立刻终止线程

### 判断线程是否中断

* interrupted方法判断当前线程是否中断，清除中断标志
* isInterrupt方法判断线程是否中断，不清除中断标志

#### interrupted方法

* thread线程执行Interrupt方法后thread的中断状态应该为true，但由于interrupted方法判断当前线程的中断状态(这个case是main线程)，所以他的中断状态是false
* interrupted方法有清除中断标志的作用，所以再次执行输出false

#### isInterrupt方法

* 判断调用者中断状态，并且不清除中断标志

### 停止线程

* 执行interrupt方法并没有中断进程，可以在run方法中进行判断，判断中断状态，为true停止run方法。
* 也可以直接排除异常进行捕获，这样就可以不执行后面的代码
* 也可以直接return，但是抛出异常比较好，后面可以继续将异常抛出，让线程中断事件得到传播

### sleep与interrupt

* 中断状态，进入sleep抛出异常
* 睡眠进入中断状态，抛出异常

### 暂停线程

* suspend会直接锁住同步方法
* 因为把这个锁占住
* Sytem.out.println()替换printString方法同样也会锁住

### yield方法
* 放弃当前cpu资源，让其他任务去占用，但是什么时候放弃不知道，因为放弃后，可能又开始获得时间片

### 线程优先级

* cpu先执行优先级高的线程的对象任务，setPriority方法可以设置线程的优先级
* 线程优先级具有继承性

### 优先级特性

* 规则性，cpu尽量将资源给优先级高的
* 随机性，优先级高的不一定执行完run方法

### 守护线程
* 线程有两种，一种是用户线程，一种是守护线程
* 垃圾回收线程就是典型的守护线程，当jvm中还有非守护线程，守护线程就一直还在，直到非守护线程不存在了，守护线程才销毁

### 总结
* 线程提高了资源利用率
* 线程的实现可以通过继承Thread类，也可以通过实现Runnable接口
* 线程中断方式有三种，常用的是interrupt方法，该方法没有立即中的线程，只是做了一个中断标示
* interrupted和isInterrupt两种方法都可以查看线程中断状态，第一种查看当前线程的中断状态，第二章查看该方法调用者的中断状态
* interrupted方法会消除中断状态，isInterrupt不会清除中断状态
* interrupt方法没有真正中断线程，所以可以在run方法里面判断中断状态，然后通过抛出异常或者return来中断线程
* 当线程中的状态位true，再进入sleep会抛出异常，反之一样
* 线程暂停容易将锁站住
* 线程具有优先级，可以通过方法设置线程优先级，差别不大时优先级高的不一定执行完run方法
* 守护线程&用户线程

## 0.3 对象变量的并发访问

### 线程安全
* 线程安全就是多线程访问时，采用了加锁机制，提供了数据访问保护。线程不安全就是不提供数据访问保护，有可能出现多个线程先后更改数据造成所得到的数据是脏数据

### 局部变量并不会数据共享
* 局部变量不同线程并不共享

### 实例成员变量数据共享
* 不同线程访问同一个变量时会发生覆盖，出现线程安全问题

### synchronized关键字可以避免线程安全问题
* 增加synchronized，避免线程安全问题

### 总结
* 如果多个线程访问同一个对象方法中的局部变量，那么这个变量并不共享，不同线程对此变量的操作互不影响
* 如果多个线程访问的是同一个对象方法中的成员变量，那么这个变量共享，如果不处理好线程问题，可能会出现线程安全问题
* 通过synchronized关键字可以使方法同步

### 多个线程访问的是两个不同实例的同一个同步方法
* 锁对象不同不造成互斥作用

### 多线程调用同一个实例的两个不同(一个同步，一个非同步)方法
* 并不会同步执行

### 多线程调用同一个实例的两个不同的同步方法
* 只添加synchronized关键字时，方法同步执行

### 总结

* 多个线程调用不同实例的同步方法，线程不互斥
* 如果两个线程的锁对象一样，两个线程分别调用同步方法和非同步方法，线程不会同步执行
* 如果调用同一个锁对象的不同同步方法，因为锁对象一直，两个线程同步执行
* 一个实例有两个方法，一个修改值，一个读值，因为不会争抢锁，可能产生脏读的情况

### 重入锁
* 一个线程获得锁后，还没释放可以再次获取锁

### 出现异常会释放锁
* 如果同步方法里面出现异常，会自动将锁释放

### 同步方法不会继承
* 如果父类的方法是同步的，子类重载方法时没有synchronized关键字，那么没有同步作用

### 总结
* 重入锁，一个获得锁的线程没执行完可以继续获得锁
* 线程占用锁的时候，如果执行的同步出现异常，会将锁让出
* 父类方法同步，子类重写该方法，没有synchronized关键字修饰，是没有同步作用的

### 同步代码块
* 当多个线程访问同一个对象的synchronized代码块时，一段时间内只有一个线程能执行

### 同步代码块的锁对象
* 将this换成自己创建的锁(一个对象)，同样可以实现同步功能

### 部分同步，部分异步
* 不在同步代码块中的代码可以异步执行，在同步代码块中的代码同步执行

### 不同方法里面的synchromized代码块同步执行
* 两个线程访问同一个对象的两个同步代码块，这两个代码块是同步执行的

### 锁不同没有互斥作用
* 将this改成s，也就是改变锁对象，发现两个方法并不是同步执行的

### synchronized方法和synchronized(this)代码块是锁定当前对象的
* 将service代码块改成synchronized方法，发现输出结果是同步的，说明锁的都是当前对象

### 总结
* 同步代码块的锁对象可以使本对象，也可以是其他对象。同一个锁对象可以产生互斥作用，不同锁对象不能产生互斥作用。
* 一个方法中有同步代码块和非同步代码块，同步代码块是同步执行的，而非同步代码块的代码可以异步执行
* 一个对象中的不同同步代码块是互斥的，执行完一个代码块再执行另一个代码块
* 同步代码块(this)和synchronized方法锁定的都是当前对象this

### synchronized static 同步静态方法
* 在这里锁对象就不是service对象，而是Class

### Class锁
* 效果和上面的今天天的synchronized一样

### 静态类中非静态同步方法
* 锁的还是对象

### 总结
* Class类也可以是锁，Class锁的实现可以通过静态的synchronized方法，也可以通过静态方法里面的同步代码块
* 静态类的同步方法锁对象还是该类的一个实例

### 死锁
* AB线程分别开始执行时，因为锁不同，所以不互斥，当A执行到另一同步代码块时，由于锁被B线程占有了，所以只能等待，同样B线程也是如此，AB互相抢对方的锁造成了死锁

### 锁对象发生改变
* 修改锁对象的属性不影响结果

### volatile
* 如果把vm运行参数设置为-server，会发现程序线程安全性出现问题
* 变量存放在公共堆栈和线程的私有堆栈中，对其赋值时只对公共堆栈进行更新，设置-server运行参数后，读取的是线程私有栈的内容，也就造成了死循环。在变量前加上volatile关键字，这个时候访问的是公共堆栈，不会造成死循环
* 单线程时没有这个问题，但当多个线程进行读写操作时由于没有同步机制保证，可能会出现这个问题
* volatile不处理数据原子性
* synchronized不仅保证了对同一个锁线程的互斥，还保证了数据的同步

### 总结
* volitile增加了实例变量在多个线程之间的可见性，保证我们获得的是变量的最新值
* volatile在读上面保持了同步作用，但是在写上面不保持同步
* synchronized的同步作用不仅保证了对同一个锁线程的互斥，还保证了数据的同步

### volatile和synchronized之间的对比
* 两者修饰的不同，volatile修饰的是变量，synchronized修饰的是方法和代码块
* 两者的功能不同，volatile保证数据的可见性，synchronized是现场同步执行(间接保证数据可见性，让线程工作内存变量和公共内存同步)
* volatile性能比synchronized性能高

### 用原子类实现i++同步
* 将上面的count++用原子类AtmoicInteger改变

## 0.4 等待通知机制

### 非等待通知
* 两个线程实现了通信，线程B不断轮询共享变量，很占资源
* 如果轮询的时间间隔小，浪费资源更加严重
* 如果轮询的时间间隔大，可能会错过想要的数据
* 采用共享变量轮询方式实现通信，1.浪费cpu资源，2.可能读取到错误的数据

### 什么是等待通知机制
* 线程A等待线程B发出通知才执行，线程A执行wait方法，等待线程B执行Notify方法唤醒线程A

### 等待通知机制实现
* 通过等待通知机制，避免线程A不断轮询造成的资源浪费

### 消息通知机制注意点
* wait和notify必须在同步方法和同步代码块里面调用，不然会抛出异常
* notify方法继承自object类，可以唤醒在此对象监视器等待的线程，也就是说唤醒的是同一个锁的线程
* notify方法调用后，不会马上释放锁，而是运行完该同步方法或者是运行完该同步代码块的代码
* 调用notify后随即唤醒的是一个线程
* 调用wait方法后会将锁释放
* wait状态下中的线程会抛出异常
* wait(long)，超过设置的时间后会自动唤醒，还没超过该时间也可以通过其他线程唤醒
* notifyAll可以唤醒同一锁的所有线程
* 如果线程没有处于等待状态，其他线程进行唤醒，不会起作用

### 生产者，消费者模式
* 一个生产者、一个消费者，这个时候会交替进行
* 创建多个生产者，多个消费者，如果唤醒的是同类线程，可能会出现家私状态，把notify缓存notifyAll可以解决这个问题

### 消息通知机制需要注意的地方
* 线程唤醒的是否是同类线程会造成影响
* 生产者消费者模式，判断条件if和while的选择

### 通过管道进行线程间通信
* PipedInputStream和PipedOutputStream(对应PipedReader和PipedOutputWriter)可以实现线程间流的同学，将管道输出流和输入流链接，实现一个线程往管道发送数据，一个线程从管道读取数据

### join方法
* 当前线程阻塞(main线程)，调用线程(threadTest)正常执行，执行完后当前线程(main)继续执行
* 如果线程B执行完了join方法时被中断，那么此时抛出异常，但是线程A正常运行

### join(long)和sleep(long)的区别
* 从join方法的源代码可以发现，核心方法是wait，说明join也会释放锁
* join方法是非静态的，而sleep是静态的

### ThreadLocal
* 解决变量在各个线程的隔离性，每个线程绑定自己的值
* 每个线程都设置了值，但是得到的值却是自己的，互相隔离
* 如果不开始不设置值，那么得到的值都是null，可以通过继承ThreadLocal，重载initalValue方法，设置初始值
* InheritableThreadLocal，子线程可以继承父线程的值

## ReentranLock的使用
### ReentrantLock特性
* 尝试获得锁
* 获取到锁的线程能够响应中断

### ReentranLock(重入锁)
效果和synchronized一样，都可以同步执行，lock方法获得锁，unlock方法释放锁

### await方法
* 通过创建Condition对象来使线程wait，必须先执行lock.lock方法获得锁

### signal方法
* condition对象的signal方法可以唤醒wait线程

### 创建多个condition对象
* 一个condition对象的signal方法和该对象的await方法是一一对应的，一个condition对象的signal方法不能唤醒其他condition对象的await方法
* ReentrantLock类可以唤醒指定条件的线程，而object的唤醒是随机的

### Condition类和Object类
* Condition类的await方法和Object类的wait方法等效
* Condition类的signal方法和Object类的notify方法等效
* Condition类的signalAll方法和Object类的notifyAll方法等效

### Lock的公平锁和非公平锁
	Lock lock = new ReentrantLock(true);//公平锁
	Lock lock = new ReentrantLock(false);//非公平锁
	
* 公平锁指的是线程获取锁的顺序是按照加锁顺序来的，而非公平锁是抢锁机制，先lock的线程不一定先获得锁

### ReentranLock类的方法

* getHoldCount()查询当前线程保持此锁的次数买也就是执行此线程执行lock方法的次数
* getQueueLength()返回整等待获取此锁的线程估计数，比如启动10个线程，一个获得锁，此时返回9
* getWaitQueueLength()返回等待与此锁相关的给定条件的线程估计数。
* hasWaiters(Condition condition)查询是否有线程等待与此锁有关的给定条件
* hasQueuedThread查询给定线程是否等待获取此锁
* hasQueuedThread 查询给定线程是否等待获取锁
* hasQueuedThreads() 是否有现场等待此锁
* isFair()该锁是否公平锁
* isHeldByCurrentThreads()是否有线程等待此锁
* isLock()此锁是否有任意线程占用
* lockInterruptibly() 如果当前线程未被中断，获取锁
* tryLock()尝试获得锁，仅在调用时锁未被线程占用，获得锁
* tryLock(long timeout TimeUnit unit)如果锁在给定等待时间内没有被另一个线程保持，则获取该锁

### tryLock 和 lock 和 lockInterruptibly的区别
* tryLock能获得锁就返回true，不能就立即返回false，可以增加时间限制
* lock能获得锁就返回true，不能的话一直等待获得锁
* lock和lockInterruptiblym，前者不抛出异常，后者抛出异常

### 读写锁
* ReentranLock是完全互斥排他的，这样其实效率是不高的
* 读锁，此时多个线程可以获得读锁
* 写锁，此时只有一个线程能获得写锁
* 结果是读锁释放后才执行写锁的方法，说明读锁和写锁是互斥的

### 总结
* Lock类也可以实现线程同步，而Lock获得锁需要执行lock方法，释放锁需要执行unLock方法
* Lock类也可以创建Condition对象，Condition对象用来使线程等待和唤醒线程，Condition对象的唤醒是用同一个Condition执行await方法的线程，所以也就可以实现唤醒指定类的线程
* Lock类分公平锁和不公平锁，公平锁安装加锁顺序来的，不公平锁是不按顺序的
* Lock类有读锁和写锁，读读共享，写写互斥，读写互斥

## 0.6 Synchronized ReentrantLock volatile Atomic 原理分析
### Synchronized
#### 原理

* synchronized关键字通过字节码指令来实现
* synchronized关键字编译后会在同步块前后形成monitorenter和monitorexit两个字节码指令
* 执行monitorenter指令时需要先获得对象的锁(每个对象有一个监视器锁monitor)，如果这个对象没被锁或者当前线程已经获得此锁(也就是重入锁)，那么锁的计数器+1。如果获取失败，那么当前线程阻塞，直到锁被另一个线程释放
* 执行monitorexit指令时，计数器减一，当为0时候锁释放

		class Test
		{
			public int i-1;
			public void test(){
				synchronized (this){
					i++;
				}
			}
		}
		
### volatile
#### 作用
* 保证变量对所有线程的可见性，当一个线程修改了这个变量的值，其他线程可以立即知道这个新值(Java内存模型会导致可见性的问题)

#### 原理
* 所有变量都存在主内存，每个线程有自己的工作内存，工作内存保存了呗该线程使用的变量的主内存副本拷贝
* 线程对变量的所有操作都必须在工作内存中进行，不能直接读写主内存的变量，也就是必须先通过工作内存
* 一个线程不能访问另一个线程的工作内存
* volatile保证了变量更新的时候能够立即同步到主内存，使用变量的时候能立即从主内存刷新到工作内存，这样就保证了变量的可见性
* 实际上是通过内存屏障来实现的。语义上，内存屏障之前的所有写操作都要写入内存;内存屏障之后的读操作都可以获得同步屏障之前的写操作的结果

### Atomic
#### 作用
* 当有多个线程同时对单个(包括基本类型和引用类型)变量进行操作时，具有排他性，即当多个线程同时对该变量进行更新时，仅有一个线程能成功，而未成功的线程可以像自旋锁一样继续尝试，知道执行成功

#### 原理
* CAS操作，通过一个cpu指令来实现，该指令是一个原子指令，指令有3个操作数ABC，A为内存位置，B为预期值，C为新值，如果A符合旧预期值B，那么用V更新A，如果不符合就不更新，这个过程是原子操作
* 通过硬件级别的cpu指令来实现，而不是像synchronized一样阻塞线程
* 如果此时有两个线程，线程A得到current值为1，线程B得到current也为1，此时线程A执行CAS操作，成功将值改为2，而此时线程B执行CAS操作，发现此时内存中的值并不是读到current值1，所以返回false，此时线程B继续进行循环，最后成功加1

### Lock
#### 作用
* 显式加锁

#### 原理
* 通过同步器AQS（AbstractQueuedSynchronized类）来实现，AQS根本上是通过一个双向队列来实现
* 线程构造成一个节点，一个线程先尝试获得锁，如果获取锁失败，就将该线程加到队列尾部
* 非公平锁的lock方法，调用的sync的lock方法
* NonfairSync的lock方法，acquire的是Sync的父类AQS的acquire方法
* AQS的acquire方法
* tryAcquire方法，尝试获得锁
* 如果获取锁失败，将节点加入尾节点
* 如果节点为空或者节点不为空并且由其他线程插入(CAS返回false)，执行enq
* 进入队列的线程尝试获得锁
* 线程是否可以挂起
* 挂起当前线程，返回线程中断状态并重置

## 1.线程的生命周期、线程各个状态之间的切换

### 线程的生命周期
* 新建状态(New)
* 就绪状态(Runnable)
* 运行(Running)
* 阻塞(Blocked)
* 死亡(Dead)

### 线程状态之间的转换

#### 新建和就绪状态
当程序使用 new关键字 创建了一个线程之后，该线程就处于新建状态，仅仅由Java虚拟机为其分配内存，未表现出线程的动态特征，也不会执行线程的线程执行体

当程序调用了 start()方法之后，该程序就处于就绪状态。JVM会创建方法调用栈和程序计数器，此时线程并没有开始运行，何时运行取决于JVM里线程调度群的调度

启动线程使用start()方法，而不是run()方法，使用start()方法启动线程，系统会把该run()方法当成线程执行体来处理;如果直接调用run方法，只是普通方法，不是线程执行体。调用run()方法之后，不再处于新建状态，只能对处于新建状态的线程调用start()方法。
	
就绪状态相当于"等待执行"

#### 运行和阻塞状态

##### 线程调度
处于就绪状态的线程获得了CPU，开始执行run()方法的线程执行体，则该线程处于运行状态。如果计算机只有一个CPU，那么任何时刻只有一个线程处于运行状态。

* 抢占式调度策略:每个可执行线程只分配一个小时间段
* 协作式调度策略:必须由该线程主动放弃所占用的资源(sleep()或yield())

##### 线程阻塞
发生如下情况时，线程将进入阻塞状态

* 线程调用sleep()方法主动放弃所占用的处理器资源
* 线程调用一个阻塞式IO方法，在该方法返回之前，该线程被阻塞
* 线程试图获得一个同步监视器，但该同步监视器正被其他线程所只有
* 线程在等待某个通知(Notify)
* 程序调用了线程的suspend()方法将该线程挂起。但这个方法容易导致死锁，应避免使用

当前正在执行的线程被阻塞之后，其他线程就可以获得执行的机会。被阻塞的线程会在合适的时候重新进入就绪状态，重新等待线程调度器调度

##### 解除阻塞
解除阻塞，重新进入就绪状态的情况：

* 调用sleep()方法的线程经过了指定的时间
* 线程调用的阻塞式IO方法已经返回
* 线程成功获得了试图取得的同步监视器
* 线程正在等待某个通知时，其他线程发出了通知
* 处于挂起状态的线程被调用了resume()恢复方法

#### 线程死亡

##### 死亡状态

* run()或call()方法执行完成，线程正常结束
* 线程抛出一个未捕获的Exception或Error
* 直接调用该线程stop()方法来结束该线程--容易导致死锁，不推荐

##### 程序设计

当主线程死亡时，其他线程不受任何影响，一旦子线程起来就拥有和主线程相同的地位。可以调用线程对象的isAlive()方法测试线程是否死亡，当线程处于就绪、运行、阻塞状态时，方法返回true;当线程处于新建、死亡状态时，返回false。

	不要试图对一个已经死亡的线程调用start()方法使它重新启动，死亡后的线程不可再次作为线程启动
	
如果在线程已死亡的情况下调用start来启动，会引发IllegalThreadStateException一场，表明处于死亡状态的线程无法再次运行

## 2.ReentrantLock、ArrayBlockingQueue、LinkedBlockingQueue源码

### ReentrantLock 可重入锁
* 可重入锁，撸了一遍

### Java阻塞队列BlockingQueue
BlockingQueue接口提供三个添加元素方法

* add:添加元素到队列里，添加成功返回true，由于容量满了添加失败抛出illegalStateException
* offer:添加元素到队列里，添加成功返回true，添加失败返回false
* put:添加元素到队列里，如果容量满了会阻塞直到容量不满

三个删除元素的方法
* poll:删除队列头部元素，如果队列为空，返回null。否则返回元素
* remove:基于对象找到对应的元素，并删除。删除成功返回true，否则返回false
* take:删除队列头部元素，如果队列为空，一直阻塞到队列有元素并删除

#### ArrayBlockingQueue
使用一个可重入锁和这个锁生成的两个条件对象进行并发控制

是一个带有长度的阻塞队列，初始化的时候必须制定队列长度，指定长度之后不允许进行修改

具有如下属性

		// 存储队列元素的数组，是个循环数组
		final Object[] items;
		
		//拿数据索引
		int taskIndex;
		
		//放数据索引
		int putIndex;
		
		//元素个数
		int count;
		
		//可重入锁
		final ReentrantLock lock;
		
		//非空条件对象，由lock创建
		private final Condition notEmpty;
		
		//未满条件对象，由lock创建
		private final Condition notFull;
		
##### 数据的添加

add方法

		public boolean add(E e){
			if (offer(e))
				return true;
			else throw new IllegalStateException
		}
		
add方法内部调用offer方法如下

		public boolean offer(E e){
			checkNotNull(e); //不允许元素为空
			final ReentrantLock lock = this.lock;
			lock.lock(); //加锁，保证调用offer方法的时候只有一个线程
			try{
				if (count == items.length) //如果队列已满
					return false; //直接返回false，添加失败
				else{
					insert(e); //数组没满的话调用insert方法
					return true; //返回true，添加成功
				}
			} finally {
				lock.unlock(); 释放锁，让其他线程可以调用offer方法
			}
		}
		
insert方法如下

		private void insert(E x) {
			items[putIndex] = x; //元素添加到数组里
			putIndex = inc(putIndex); //放数据索引+1，当索引满了变成0
			++count; //元素个数+1
			notEmpty.signal();//使用条件对象notEmpty通知，比如使用take方法的时候队列里没有数据
		}
		
put方法:

		public void put(E e) throws InterruptedException {
			checkNotNull(e); //不允许元素为空
			final ReentrantLock lock = this.lock;
			lock.lockInterruptibly(); //加锁，保证调用put方法时候只有一个线程
			try{
				while (count == items.length) // 如果队列满了，阻塞当前线程，并加入到条件对象notFull的等待队列里
					notFull.await(); //线程阻塞并被挂起，同时释放锁
				insert(e);//调用insert方法
			} finally {
				lock.unlock(); //释放锁，让其他线程可以调用put方法
			}
		}
		
总结如下

* add方法内部调用offer方法，如果队列满了，抛出illegalStateException异常，否则返回true
* offer方法如果队列满了，返回false，否则返回true
* put方法如果队列满了会阻塞线程，直到有线程消费了队列里的线程才唤醒

#####数据的删除
poll,take,remove方法

poll方法:

		public E poll(){
			 final ReentrantLock lock = this.lock;
			 lock.lock();加锁,保证调用poll方法的时候只有一个线程
			 try{
			 	return (count == 0) ? null : extract();//如果队列里没元素了，返回null，否则调用extract方法
			 } finally {
			 	lock.unlock(); //释放锁，让其他线程可以调用poll方法
			 }
		}
		
poll方法内部调用extract方法:

		private E extract() {
			final Object[] items = this.items;
			E x = this.<E>cast(items[takeIndex]);//得到取索引位置上的元素
			items[takeIndex] = null; //对应取索引上的数据清空
			takeIndex = inc(takeIndex); //取数据索引+1，当索引满了变成0
			--count; //元素个数 - 1
			notFull.signal();//使用条件对象notFull通知
			return x; //返回元素
		}
		
take方法:

		public E take() throws InterruptedException {
			final ReentrantLock lock = this.lock;
			lock.lockInterruptibly();//加锁，保证调用take方法的时候只有一个线程
			try {
				while (count == 0) //如果队列空，阻塞当前线程，并加入到条件对象的等待队列中
					notEmpty.await();//线程阻塞并被挂起，同时释放锁
				return extract();//调用extract方法
			} finally {
				lock.unlock();//释放锁，让其他线程可以调用take方法
			}
		}
		
remove方法:

		public boolean remove(object o) {
			if (o == null) return false;
			final Object[] items = this.items;
			final ReentrantLock lock = this.lock;
			lock.lock(); //加锁，保证调用remove方法的时候只有1个线程
			try {
				for (int i = takeIndex, k = count; k > 0; i= icn(i), k--){//遍历元素
					if (o.equals(items[i])){//两个对象相等的话
						removeAt(i); //调用removeAt方法
						return true; //删除成功，返回true
					}
				}
				return false;//删除成功，返回false
			} finally {
				lock.unlock();//释放锁，让其他线程可以调用remove方法
			}
		}
		
removeAt方法:

		void removeAt(int i){
			final Object[] items = this.items;
			if (i==takeIndex) { //如果要删除数据的索引是取索引位置，直接删除索引位置上的数据
				items[takeIndex] = null;
				takeIndex = inc(takeIndex);
			} else { // 如果要删除数据的索引不是取索引位置，移动元素，更新取索引和放索引的值
				for (;;){
					int nexti = inc(i);
					if (nexti != putIndex){
						items[i] = items[nexti];
						i=nexti;
					} else {
						items[i] = null;
						putIndex = i;
						break;
					}
				}
			}
			--count; //元素个数-1
			notFull.signal();//使用条件对象notFull通知
		}
 
 poll对于队列为空的情况，返回null，否则返回队列头部元素
 
 remove方法取的元素基于对象的下标值，删除成功返回true，否则返回false
 
 poll和remove方法不会阻塞线程
 
 take方法对于队列为空的情况，会阻塞并挂起当前线程，知道有数据加入到队列中
 
#### LinkedBlockingQueue
LinkedBlockingQueue是一个使用链表完成队列操作的阻塞队列。链表是单向链表，而不是双向链表。

内部使用放锁和拿锁，这两个锁实现阻塞("two lock queue" algorithm)

它带有的属性如下

		//容量大小
		private final int capacity;
		
		//元素个数，因为有2个锁，存在竞态条件，使用AtomicInteger
		private final AtomicInteger count = new AtomicIntefer(0);
		
		//头结点
		private transient Node<E> head;
		
		//尾结点
		private transient Node<E> last;
		
		//拿锁
		private final ReentrantLock takeLock = new ReentrantLock();
		
		//拿锁的条件对象
		private final Condition notEmpty = takeLock.newCondition();
		
		//放锁
		private final ReentrantLock putLock = new ReentrantLock();
		
		//放锁的条件对象
		private final Condition notFull = putLock.newCondition();
		
ArrayBlockingQueue只有一个锁，添加和删除数据时候只能有1个被执行，不允许并行执行。

LinkedBlockingQueue有两个锁，拿锁和放锁，添加和删除数据可以并行进行。当然添加和删除数据时只有一个线程各自执行。

##### 数据的添加(add, offer, put)

add方法内部调用offer方法

		public boolean offer(E e){
			if (e == null) throw new NullPointerException();//不允许空元素
			final AtomicInteger count = this.count;
			if (count.get() == capacity)
				return false;
			int c= -1;
			Node<E> node = new Node(e);//容量没满，以锌元素构造节点
			final ReentrantLock putLock = this.putLock;
			putLock.lock();//放锁加锁，保证调用offer方法的时候只有一个线程
			try{
				if (count.get() < capacity){//再次判断容量是否已满，因为可能拿锁在进行消费数据，没满的话继续执行
					enqueue(node);//节点添加到链表尾部
					c = count.getAndIncrement();//
					if (c + 1 < capacity) //如果容量还没满
						notFull.signal(); //在放锁的条件对象notFull上唤醒正在等待的线程，表示可以再次往队列里面加数据
				}
			} finally {
				putLock.unlock();//释放放锁，让其他线程可以调用offer方法
			}
			if(c==0)//可能拿锁在消费数据
				signalNotEmpty();
			return c>=0;//添加成功返回true，否则返回false
		}
		
put方法

		put void put(E e) throws InterruptedException {
			if(e == null) throw new NullPointerException(); //不允许空元素
			int c = -1;
			Node<E> node = new Node(e); //以锌元素构造节点
			final ReentrantLock putLock = this.putLock;
			final AtomicInteger count = this.count;
			putLock.lockInterruptibly();//放锁加锁，保证调用的时候只有一个线程
			try{
				while (count.get == capacity){//如果容量满了
					notFull.await();//阻塞并挂起当前线程
				}
				enqueue(node);//节点添加到链表尾部
				c = count.getAndIncrement();//元素个数+1
				if(c+1 < capacity)//如果容量还没满
					notFull.signal();//在放锁的条件对象noFull上唤醒正在等待的线程
			} finally {
				putLock.unlock();//释放放锁，让其他线程可以调用Put方法
			}
			if (c==0)//由于拿锁和放锁同时存在，count可能存在变化
				signalNotEmpty()//唤醒拿锁
		}
		
ArrayBlockingQueue中放入数据阻塞的时候，需要消费数据才能唤醒

LinkedBlockingQueue内部有两个锁，可以并行执行放入和消费数据，不仅在消费数据的时候进行唤醒插入阻塞的线程，同时在插入的时候如果容量还没满，也会唤醒插入阻塞的线程

##### 数据的删除(poll, take, remove)

poll方法

		public E poll() {
			final AtomicInteger count = this.count;
			if (count.get() == 0)//如果元素个数为0
				return null;//返回null
			E x = null;
			int c = -1;
			final ReentrantLock takeLock = this.takeLock;
			takeLock.lock();//拿锁加锁，保证调用poll方法的时候只有1个线程
			try {
				if (count.get() > 0){//判断队列里是否还有数据
					x = dequeue(); //删除头结点
					c = count.getAndDecrement(); //元素个数-1
					if (c > 1)
						notEmpty.signal();//在拿锁的notEmpty对象上唤醒等待的线程
				}
			}finally {
				takeLock.unlock();//释放拿锁
			}
			if (c == capacity) //两锁存在的原因
				signalNotFull();//在放锁的条件下唤醒等待的线程
			return x;
		}
		
take方法

		public E take() throws InterruptedException {
			E x;
			int c = -1;
			final AtomicInteger count = this.count;
			final ReentranLock takeLock = this.takeLock;
			takeLock.lockInterruptibly();//拿锁加锁，保证调用take方法的时候只有一个线程
			try {
				while(count.get()==0){//如果队列里已经没有元素了
					notEmpty.await();//阻塞并挂起当前线程
				}
				x = dequeue();//删除头结点
				c = count.getAndDecrement();//元素个数-1
				if (c > 1)//如果队列里还有元素
					notEmpty.signal();//在拿锁的条件对象notEmpty上唤醒正在等待的线程
			} finally {
				takeLock.unlock();//释放拿锁，让其他线程可以调用take方法
			}
			if (c == capacity)//由于存在放锁和拿锁
				signalNotFull();
			return x;
		}
		
remove方法

		public boolean remove(object o) {
			if (o == null) return false;
			fullyLock();remove操作要移动的位置不固定，2个锁都需要加锁
			try {
				for (Node<E> trail = head, p = trail.next;//从链表头结点开始遍历
					p != null;
					trail = p, p = p.next){
					if (o.equals(p.item)){//判断是否找到对象
						unlink(p, trail);//修改节点的链接信息，同时调用NotFull的signal方法
					}
				}
				return false;
			}
			} finally {
				fullyUnlock();//两个锁解锁
			}
		}
		
take方法在没数据的情况下会阻塞,poll方法删除链表头结点，temove方法删除指定对象
		
		
## 3.自旋锁
### 一.自旋锁的概念
与互斥锁类似，用于线程(进程)之间的同步。当处理器阻塞一个线程引擎的线程上下文切换的代价高于等待资源代价的时候，可以不放弃CPU时间片，在原地忙等。自旋锁是一种非阻塞锁。

### 二.自旋锁可能引起的问题
* 1.过多占据CPU时间:可以设定超时时间，当持有者超时不释放锁时，等待着放弃CPU阻塞时间片
* 2.死锁问题，有一个线程两次试图获得自旋锁，第一次这个线程获得了锁，当第二次试图加锁的时候，检测到锁已被占用(其实是自己占用)，那么这时，线程会一直等待自己释放锁而不是继续执行，这样就引起了死锁。递归程序决不能在持有自旋锁时调用自己，也决不能在递归调用时试图获得相同的自旋锁。

可重入的自旋锁

		public class SpinLock {
			AtomicReference<Thread> owner = new AtmoicReference<Thread>();//持有自旋锁的线程对象
			private int count;//用一个计数器来做重入锁获取次数的计数
			public void lock(){
				Thread cur = Thread.currentThread();
				if (cur == owner.get()){
					count++;
					return;
				}
				while(!owner.compareAndSet(null, cur)){}//lock函数将owner设置为当前线程，并且预测原来的值为空，unlock函数将owner设置为null，并且预测为当前线程。当有第二个线程调用lock操作时由于owner值不为空，导致循环一直被执行，直至第一个线程调用unlock函数将owner设置为null，第二个线程才能进入临界区
			}
			public void unLock(){
				Thread cur = Thread.currentThread();
				if (cur == owner.get()){
					if (count > 0){
						count--;
					} else{
						owner.compareAndSet(cur, null);
					} 
				}
			}
		}
		
### 三. Java CAS

系统原语，Compare And Set的缩写

## 4.volatile、内存屏障

### 一.内存屏障（Memory barrier）
#### 为什么会有内存屏障
* 每个CPU都会有自己的缓存(甚至会有L1,L2,L3)，缓存的目的是为了提高性能，避免每次都要向内存取。但是这样的弊端也很明显:不能实时和内存发生信息交换，分在不同CPU执行的不同线程对同一个变量的缓存值不同。
* 用valitle关键字修饰变量可以解决上述问题。volatile通过内存屏障来解决问题。内存屏障是硬件层的概念，不同的硬件平台实现内存屏障的手段不同，java通过jvm来统一生成内存屏障

#### 内存屏障是什么
* 硬件层的内存屏障分为两种: Load Barrier和Store Barrier，即读屏障和写屏障
* 内存屏障有两个作用:

		1.阻止屏障两侧的指令重排序
		2.强制把写缓冲区/高速缓存中的脏数据等写回主内存，让缓存中相应的数据失效
* 对应Load Barrier来说，在指令前插入Load Barrier，可以让高速缓存中的数据失效，强制重新从主内存加载数据;强制所有在load屏障指令之后的load指令，都在改load屏障指令执行之后才被执行，并且一直等到load缓冲区被该CPU读完之后才能执行之后的load指令。这使得从其他CPU暴露出来的程序状态对该CPU可见
* 对于Store Barrier来说，在指令后插入Store Barrier，能让写入缓存中的最新数据更新写入主内存，让其他线程可见。强制所有在store屏障指令之前的store指令，都在改store屏障指令执行之前被执行，并把store缓冲区的数据都刷到CPU缓存，这使得程序状态对其他CPU可见。

#### Java内存屏障
* Java内存屏障包含四种:LoadLoad,StoreStore,LoadStore,StoreLoad,通过对读屏障和写屏障的组合来完成屏障和数据同步功能
* LoadLoad屏障:Load1;LoadLoad;Load2，在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕
* StoreStore屏障:Store1;StoreStore;Store2，在Store2及后续写入操作执行前，保证Store1的写入操作对其他处理器可见
* LoadStore屏障:Load1;LoadStore;Store2，在Store2及后续写入操作被刷出前，保证Load1要读取的数据读取完毕，保证其他CPU的程序操作的状态对此CPU可见
* StoreLoad屏障:Store1;StoreLoad;Load2，在Load2及后续所有读取操作前保证Store1的写入对所有处理器可见。开销最大，万能屏障

### 二.volatile语义中的内存屏障

* volatile的内存屏障策略非常严格保守，悲观且毫无安全感心跳
		
		在每个volatile写操作之前插入StoreStore屏障，在写操作后插入StoreLoad屏障
		在每个volatile读操作之前插入LoadLoad屏障，在读操作后插入LoadStore屏障
		
* 由于内存屏障的作用，避免了volatile变量和其他指令重排序、线程之间实现了同学，使得volatile表现出了锁的特性

### 三.final语义中的内存屏障
* 对于final域，编译器和CPU遵循两个排序规则

		1.新建对象过程中，构造体中对final的初始化写入和这个对象赋值给其他引用变量，这两个操作不能重排序
		2.初次读包含final域的对象引用和读取这个final域，这两个操作不能重拍下
		
* 必须保证一个对象所有final域被写入完毕后才能引用和读取。这也是内存屏障起的作用
* 写final域：在编译器写final域完毕，构造体结束之前，会插入一个storestore屏障，保证前面的对final写入对其他线程/CPU可见
* 读final域：在读final域前插入LoadLoad屏障

## 5.线程池

### 线程池的优点
* 重复利用已创建的线程，减少创建线程和销毁线程的开销
* 提高响应速度，不需要等到线程创建就能立即执行
* 使用线程池可以进行统一分配，调优和监控
* 总的来说：降低资源销毁，提高响应速度，提高线程可管理性

### 线程池原理
* 提交任务
* 核心线程池(corePoolSize)是否已满，如果未满的话就创建线程执行任务
* 否则查看队列(BlockingQueue)是否已满，已满的话，将任务存储在队列里
* 如果已经满了，看线程池(maximumPoolSize)是否已满，如果满的话按照拒绝处理任务策略(handler)处理无法执行的任务
* 如果未满，创建线程执行任务

### ThreadPoolExecutor重要方法
* execute: 在将来某个时间执行给定任务
* submit: 提交一个任务用于执行，并返回一个表示该任务的Future
* shutdown: 按过去执行已提交任务的顺序发起一个有序的关闭，但是不接受新任务。即中断没有正在执行任务的线程，等待任务执行完毕
* shutdown: 尝试停止所有活动执行任务，暂停等待任务的处理，并返回等待执行的任务列表

### 线程池状态
* RUNNING:创建线程池后，初始状态为RUNNING
* SHUTDOWN:执行shutdown方法后，线程池处于SHUTDOWN状态
* STOP:执行shutdownNow方法后，线程池处于STOP状态
* TERMINATED:当线程池处于SHUTDOWN或STOP状态，并且所有工作线程已经销毁，任务缓存队列已经清空或执行结束后，线程池被设置为TERMINATED状态。

### 源码分析
* command是否为空，是的话抛出异常
* 如果当前线程小于corePoolSize，执行addIfUnderCorePoolSize方法，创建线程并执行任务
* 如果当前线程池线程数大于等于corePoolSize或者执行addIfUnderCorePoolSize返回false，将当前任务放到队列中
* 如果线程池状态不是RUNNING状态或者任务无法加入到队列，并且线程数大于最大线程数，执行reject方法
* 流程跟上面的线程池原理相同

### 配置线程池
* cpu密集型和io密集型线程数的选择,cpu密集型不需要太多的线程，可以充分利用cpu资源，io密集型适当的多线程，io阻塞时可以切换至另一线程
* 优先级不同的任务可以使用PriorityBlockingQueue来处理
* 建议使用有界队列，能够增加系统的稳定性(如果使用无界队列，当出现问题时，队列无限增长，可能会占用大量内存，导致系统出现问题)和预警能力(当出现队列占满的时候，抛出异常，可以让开发人员及时发现)

### 线程池使用
* 创建了一个ThreadPoolExecutor，而Executor给我们提供了几个静态方法用于创建线程池，本质上只是设置了给定的参数而已
* FixedThreadPool，顾名思义，一个固定大小的线程池，队列使用的无限制大小的链表阻塞队列
* 单线程线程池，同样队列使用的无限制大小的链表阻塞队列
* 无界线程池，无论多少任务，直接运行`

## 6.内核态与用户态
* 当一个任务(进程)执行系统调用而陷入内核代码中执行时，我们称进程处于内核运行态(简称为内核态)。此时处理器处于特权级最高的(0级内核代码中执行)。
* 当进程在执行用户自己的代码时，则称其处于用户运行态(用户态)，处理器在特权级最低的(3级)用户代码中运行。

### a.用户态和内核态的概念区别
#### 1.示例程序

		void testfork(){
		 if (0==fork()){
		 	printf("create new process success!\n");
		 }
		 printf("testfork ok\n");
		}

testfork()最初运行在用户态进程下，当它调用fork()最终触发sys_fork()执行时，就切换到了内核态

#### 2.特权级
fork实际上是以系统调用的方式完成相应的功能，具体工作由sys_fork负责实施。

特权级是非常有效的管理和控制程序执行的手段，x86架构CPI一般有0-3四个特权级，0级最高，3级最低

#### 3.用户态和内核态
当程序运行在3级特权级上时，就可以称之为运行在用户态;反之运行在0级特权级上时，可以称之为运行在内核态

特权级不同，权利不同。例如用户态下的程序不能直接访问操作系统内核数据结构和程序。

### b.用户态和内核态的转换
#### 1.用户态切换到内核态的3种方式
* 系统调用 使用操作系统为用户特别开放的中断来实现
* 异常 当CPU在执行运行在用户态的程序时，发生了某些事先不可知的异常，这时会触发由当前运行进程切换到处理此异常的内核相关程序中，也就转到了内核态
* 外围系统的中断

#### 2.具体的切换操作
* 从当前进程的描述符中提取其内核栈的ss0及esp0信息
* 使用ss0和esp0指向的内核栈将当前进程的cs,eip,eflags,ss,esp信息保存起来，这个过程也完成了由用户栈到内核栈的切换过程，同时保存了被暂停执行的程序的下一条指令
* 将先前由中断向量检索得到的中断处理程序的cs,eip信息装入相应的寄存器，开始执行中断处理程序，这时就转到内核态的程序执行了。


		
## 7.lock()、 tryLock()、lockInterupttibly()的区别
* lock() 调用后一直阻塞到获得锁
* tryLock() 尝试是否能获得锁 如果不能获得立即返回
* lockInterruptibly() 调用后一直阻塞到获得锁，但是接受中断信号

## 8.jdk线程池实现原理(ThreadPoolExecutor如何复用线程)
Java 1.5引入，将任务的提交和执行解耦

### ThreadPoolExecutor
Executors是Java线程池的工厂类，可以通过它快速初始化一个符合业务需求的线程池。

#### corePoolSize
线程池中的核心线程数

#### maximumPoolSize
线程池中允许的最大线程数，如果阻塞队列满了且继续提交任务，创建新线程

#### keepAliveTime
大于corePoolSize时的线程空闲时的存活时间

#### unit
keepAliveTime的单位

#### workQueue
用来保存等待被执行任务的阻塞队列

* ArrayBlockingQueue 基于数组的游街阻塞队列
* LinkedBlockingQueue 基于链表结构的阻塞队列，吞吐量高于前者
* SynchronousQueue 不存储元素的阻塞队列，吞吐量更高
* priorityBlockingQueue:具有优先级的无界阻塞队列

#### threadFactory

创建线程的工厂

#### handler

线程池饱和策略

* AbortPolicy:直接抛出异常，线程池默认策略
* CallerRunsPolicy:用调用者所在的线程来执行任务
* DiscardOldestPolicy:丢弃阻塞队列中最靠前的任务来执行当前任务
* DiscardPolicy:直接丢弃任务

### Executors

Executors工厂类提供了线程池的初始化接口，主要有如下几类

#### newFixedThreadPool
初始化一个指定线程数的线程池

#### newCachedThreadPool
* 初始化一个可缓存线程的线程池，线程数可达到Integer.MAX_VALUE
* 线程空闲时会释放线程，没有空闲线程创建新线程

一定要注意并发任务数，否则创建大量的线程可能导致严重的性能问题

#### newSingleThreadExecutor
初始化的线程池中只有一个线程，如果该线程异常结束会重新创建一个新的线程执行任务。保证任务的顺序执行

#### newScheduledThreadPool
初始化的线程池可以做指定的时间内周期性的执行所提交的任务

### 实现原理

#### 线程池内部状态

* RUNNING，接收新任务，处理阻塞队列中的任务
* SHUTDOWN，不会接收新任务，但会处理阻塞队列中的任务
* STOP，不会接收新任务，也不会处理阻塞队列中的任务，并且会中断正在运行的任务
* TIDYING
* TERMINATED

#### 任务提交

##### Executor.execute()
通过Executor.execute()方法提交的任务，必须实现Runnable接口，该方式提交的任务不能获取返回值

##### ExecutorService.submit()
可以获取任务执行完的返回值

#### 任务执行

##### execute实现
* 1. 根据ctl的低29位得到线程池当前线程数，如果线程数小于corePoolSize，执行addWorker创建新线程执行任务，否则执行步骤2
* 2. 如果线程池处于RUNNING状态，且把提交的任务成功放入阻塞队列中，则执行步骤3，否则执行步骤4
* 3. 再次检查线程池状态，如果线程池没有RUNNING，且成功从阻塞队列中删除任务，则执行reject方法处理任务
* 4. 执行addWorker方法创建新的线程执行任务，如果addWorker失败，则执行reject方法处理任务

##### addWorker实现
addWorker主要负责创建新的线程并执行任务

* 1. 判断线程池状态，如果线程池的状态值大于或等于SHUTDOWN，则不处理提交的任务，直接返回
* 2. 通过参数core判断当前创建的线程是否为核心线程，如果core为true，且当前线程数小于corePoolSize，则跳出循环，开始创建新的线程

线程池的工作线程通过Worker类实现，在ReentrantLock锁的保证下，把Worker实例插入到HashSet后，并启动Worker中的线程

Worker类设计如下

* 1. 继承了AQS类，可以方便的实现工作线程的中止操作
* 2. 实现了Runnable接口，可以将自身作为一个任务在工作线程中执行
* 3. 当前提交的任务firstTask作为参数传入worker的构造方法

从Worker类的构造方法实现可以发现：线程方法在创建线程thread时，将Worker实例本身作为this方法传入，thread的start方法，本质是执行了Worker的runWorker方法

##### runWorker实现
runWorker方法是线程池的核心

* 1. 线程启动后，通过unlock方法释放锁，设置AQS的state为0，表示运行中断
* 2. 获取第一个任务firstTask，执行任务的run方法，不过在执行任务前，会进行加锁操作，任务执行完会释放锁
* 3. 在执行任务的前后，可以根据业务场景自定义beforeExecute和afterExecute方法
* 4. firstTask执行完成后，通过getTask方法从阻塞队列中获取等待的任务，如果队列中没有任务，getTask方法会被阻塞并挂起，不会占用CPU资源

##### getTask实现
整个task在自旋下完成

1. workQueue.take 如果阻塞队列为空，当前线程会被挂起等待，当队列中有任务加入时，线程被唤醒，take方法返回任务，并执行
2. workQueue.pool 在keepAliveTime时间内，阻塞队列还是没有任务，则返回null

#### Future和Callable实现
通过ExecutorService.submit()方法提交的任务们可以获取任务执行完的返回值。

实际业务场景中，Future和Callable基本是成对出现的，Callable负责产生结果，Future负责获取结果

* 1. Callable接口类似于Runnable，只是Runnable没有返回值
* 2. Callable任务除了返回正常结果之外，如果发生异常，该异常也会被返回，即Future可以拿到异步执行任务各种结果
* 3. Future.get方法会导致主线程阻塞，直到Callalble任务执行完成

##### submit实现
通过submit方法提交的Callable任务会被封装成一个FutureTask对象

##### FutureTask

* 1. FutureTask在不同阶段拥有不同的状态state，初始化为NEW
* 2. FutureTask类实现了Runnable接口，这样就可以通过Executor.execute方法提交FutureTask到线程池中等待被执行，最终执行的是FutureTask的run方法

##### FutureTask.get实现

内部通过awaitDone方法对主线程进行阻塞，具体实现如下

* 1. 如果主线程被中断，则抛出中断异常
* 2. 判断FutureTask当前的state，如果大于COMPLETING，说明任务已经执行完成，则直接返回
* 3. state等于COMPLETING时，主线程通过yield方法让出cpu资源，等待state变成NORMAL
* 4. 通过WaitNode类封装当前线程，并通过UNSAFE添加到waiters链表
* 5. 最终通过LockSupport的park或parkNanos挂起线程

##### FutureTask.run实现
FutureTask.run方法在线程池中被执行，而非主线程

* 1. 通过执行Callable任务的call方法
* 2. 如果call执行成功，则通过set方法保存结果
* 3. 如果call执行有异常，则通过setException保存异常

##### finishCompletion

* 1. 执行FutureTask类的get方法时，会把主线程封装成WaitNode节点并保存在waiters链表中
* 2. FutureTask任务执行完成后，通过UNSAFE设置waiters的值，并通过LockSupport类unpark方法唤醒主线程
