#### 1.Java中Lock接口比synchronized块的优势是什么？
#### 2.你需要实现一个高效的缓存，它允许多个用户读，但只允许一个用户写，以此来保持它的完整性，你会怎样去实现它？

一、synchronized</br>
1) synchronized 是JAVA提供的强制原子性的内置锁机制。每个Java对象都可以作为一个用于同步的锁的角色，这些内置的锁被成为内部锁，线程进入synchronized块之前会自动获得锁，退出、报错异常、时会释放锁。内部锁是一种互斥锁，这就是说，至多只有一个线程可以获得锁，所以被synchronized声明的方法或代码块至多只有一个线程可以进入,从而保证了线程安全。</br>
2) synchronized实现同步与线程通信 </br>
  . java中每个对象都可以作为锁 </br>
  . 对于同步方法，锁是当前实例对象 </br>
  . 对于静态同步方法，锁是当前类 </br>
  . 对于同步方法块，锁是Synchonized括号里配置的对象 </br>
    【互斥】java 使用对象锁保证工作在共享的数据集上的线程互斥执行。 </br>
    【通信】java 使用synchronized与wait()/notify()/notifyAll()实现线程通信 </br>
 
 3) 锁原理 </br>
  . Monitor原理  </br>
  每个对象都关联一个Monitor监视器，线程进入临界区前，先要获取Monitor锁，如果获得锁，进入临界区，执行操作；如果获取锁失败，进入锁对象的等待队列。任何时刻，只能有一个线程拥有该锁对象，从而实现互斥。 </br>
  拥有监视器对象的线程，可以调用该对象的await()方法，释放锁，进入锁对象的等待队列（主动操作，往往是需要等待某种资源）； </br>
  执行结束，调用锁对象的notify()/notifyAll()方法，唤起锁对象等待队列上的线程，去竞争锁对象。</br>
 . 实现原理 </br>
 同步代码块采用monitorenter、monitorexit指令显式的实现；同步方法则使用ACC_SYNCHRONIZED标记符隐式的实现 </br>
 monitorenter、monitorexit的加锁、解锁过程：</br>
 * 当线程执行到monitorenter，尝试获取monitor; </br>
 * 如果monitor的进入数为0，或当前线程已经获得该monitor，monitor的进入数加1【可重入】 </br>
 * 如果monitor的进入数不为0，且当前线程不是monitor的持有者，进入monitor等待队列  </br>
 * 拥有monitor锁对象的线程执行到monitorexit指令处，monitor进入数减1，当进入数为0时，释放改锁对象。  </br>
 . 对象的锁 </br>
 MarkWord（对象头）存放对象的锁标记： </br>
 
 
 
 
 
 
4) 锁优化 </br>
  . 锁粗化 </br>
  . 锁消除 </br>
  . 自旋与适应性自旋 </br>
  . 四种锁状态：无锁状态、偏向锁、轻量级锁、重量级锁 </br>

二、 Lock </br>
1）jdk Lock框架</br>
 

2）实现原理 </br>
  . AQS:</br>
    state: getState()\setState(int state)\compareAndSetState(int expectState, int update) </br>
    双端队列: head、tail </br>
    owner </br>
  . 互斥锁与共享锁的实现: </br>
  
  
3）优势 </br>

- 读写锁实现高效缓存 </br>
  

4）举例
