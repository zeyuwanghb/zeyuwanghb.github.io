---
layout: post
title:  "Java并发(一)"
categories: Java并发
tags:   Java并发
author: ZyWang
excerpt: Java并发相关知识 
---

## 并发和并行 ##

并发和并行都是完成多任务的更加有效率的方式。

**并发**:同一时刻只能有一条指令执行，但多个进程指令被快速的轮换执行，指的是两个或多个事件在同一时间间隔发生，同一实体上的多个事件。

**并行**:同一时刻，多条指令在多个处理器上同时执行，指的是两个或多个事件在同一时刻发生，是在不同实体上的多个事件。

## 进程和线程的区别 ##

本质、切换、创建开销、通信方式不同、健壮性不同

**进程**:是操作系统资源分配的最小单位，是程序执行时的一个实例，程序运行时系统就会创建一个进程，并为他分配资源，然后把该进程放入进程就绪队列，进程调度器选中它时就会为它分配cpu时间，程序真正开始运行。一个程序至少有一个进程，一个进程至少有一个线程。每个进程都有独立的地址空间建立数据表来维护代码段、堆栈段、数据段，切换、创建会消耗较大的资源，一个进程崩溃后，在保护模式下不会对其他进程有影响，进程间通信以IPC方式进行，比较麻烦。

**线程**:是cpu调度和执行的最小单位，时进程的一个执行流，同一个进程内线程共享进程的数据空间，线程之间切换开销较小，有自己独立的栈和程序计数器。而一个线程死掉会导致整个进程死掉，所以多进程的程序健壮性要比多线程高，但效率差。线程间通信方便，因为共享全局变量、静态变量等数据。

##线程状态##

![](https://s1.ax1x.com/2020/08/02/aY6zOH.jpg)

线程创建后调用start()方法开始运行，当调用wait()、join()、LockSupport.lock()方法线程会进入到WAITING状态，而同时wait(long timeout)、sleep(long timeout)、join(long timeout)等增加了超时等待的功能，也就是调用这些方法后线程会进入到TIMED_WAITING状态，当超时等待时间到达后，线程会切换到Runnable的状态，另外WAITING和TIMED_WAITING状态时可以通过Object.notify()和Object.notifyAll()方法使线程转换到Runable状态。当线程出现资源竞争时，即等待获取锁的时候，线程进入BLOCKED状态，当线程获取到锁，进入到Runable状态。线程运行结束后，线程进入到TERMINATED状态，线程状态转换可以说是线程的生命周期。

![](https://s1.ax1x.com/2020/08/02/aYgqxO.jpg)

就绪态是调用了start()方法，但还有分配到cpu执行权。

## 线程创建的几种方式 ##

一个java程序从main()方法开始执行，看似没有线程参与，但实际上java程序本身就是一个多线程程序，包含了 1.分发给JVM信号的线程，垃圾回收线程 2.调用对象finalize方法的线程 3.清除Reference的线程 4.main线程，用户程序的入口。

创建新线程的几种方式:

1.通过继承Thread类，重写run方法。

2.通过实现Runable接口。

3.通过实现callable接口。

4.通过线程池创建新线程。

	 public class CreateThreadDemo {
	 
	     public static void main(String[] args) {
	         //1.继承Thread
	         Thread thread = new Thread() {
	             @Override
	             public void run() {
	                 System.out.println("继承Thread");
	                 super.run();
	             }
	         };
	         thread.start();
	         //2.实现runable接口
	         Thread thread1 = new Thread(new Runnable() {
	             @Override
	             public void run() {
	                 System.out.println("实现runable接口");
	             }
	         });
	         thread1.start();
	         //3.实现callable接口
	         ExecutorService service = Executors.newSingleThreadExecutor();
	         Future<String> future = service.submit(new Callable() {
	             @Override
	             public String call() throws Exception {
	                 return "通过实现Callable接口";
	             }
	         });
	         try {
	             String result = future.get();
	             System.out.println(result);
	         } catch (InterruptedException e) {
	             e.printStackTrace();
	         } catch (ExecutionException e) {
	             e.printStackTrace();
	         }
	     }
	 
	 }

## 守护线程 ##

守护线程是一种特殊的线程，在后台默默地守护一些系统服务，比如垃圾回收线程，JIT线程就可以理解为守护线程。与之对应的就是用户线程，用户线程可以认为是系统的工作线程，完成整个系统的业务操作。

线程可通过setDaemon(true)的方法将线程设置为守护线程，且设置守护线程要先于start()方法。

守护线程在退出的时候并不会执行finnaly块中的代码，所以将释放资源等操作不要放在finnaly块中执行，这种操作是不安全的。

## 线程间的操作 ##

**join**

join方法可以看作是线程互相协作的一种方式，例如一个线程实例A执行了threadB.join()，含义是 当前线程A会等待threadB线程终止后threadA才会继续执行。

Thread类还提供了超时等待的方法，如果线程threadB在等待时间内还没有结束的话，threadA就会在超时后继续执行。

**sleep**

Thread的静态方法，很显然它是让当前线程按照指定的时间休眠，其休眠时间的精度取决于处理器的计时器和调度器。sleep方法并不会释放锁，经常于Object.wait()方法进行比较，主要区别在于:

	1.sleep方法是Thread类中的静态方法，而wait是Object实例方法。
	2.wait()必须在同步代码块或者同步方法中调用，也就是说必须已经或的对象锁。而sleep()方法没有这个限制，可在任何地方使用。
	3.wait()方法会释放占有的对象锁，使得线程进入等待池中，等待下一次获取资源。而sleep()方法只是会让出cpu的执行权，而不会释放锁。
	4.sleep()方法会在休眠时间到达后再次获得cpu执行权时继续执行，而wait()方法必须等待object.notify/notifyAll通知后，才会离开等待池，并再次获得CPU时间片才会继续执行。

**yield**

Thread类的静态方法，一旦执行，就会让出CPU执行权，但是，让出CPU后同样可以参与下一次竞争，如果竞争到CPU时间片又会继续执行。注意，让出的时间片只会分配给当前线程相同优先级的线程。

另外需要注意的是，sleep()和yield()方法，同样都是当前线程会交出处理器资源，而它们不同的是，sleep()交出来的时间片其他线程都可以去竞争，也就是说都有机会获得当前线程让出的时间片。而yield()方法只允许与当前线程具有相同优先级的线程能够获得释放出来的CPU时间片。

在Java程序中，通过一个整型成员变量Priority来控制优先级，优先级的范围从1~10.在构建线程的时候可以通过**setPriority(int)**方法进行设置，默认优先级为5，优先级高的线程相较于优先级低的线程优先获得处理器时间片。