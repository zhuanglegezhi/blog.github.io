# AQS

AQS：`AbstractQueuedSynchronizer`，即队列同步器。在基于AQS构建的同步器中，只能在一个时刻发生阻塞，从而降低上下文切换的开销，提高了吞吐量。AQS的主要使用方式是继承，子类通过继承同步器并实现它的抽象方法来管理同步状态。

AQS使用一个int类型的成员变量state来表示同步状态，当state>0时表示已经获取了锁，当state = 0时表示释放了锁。它提供了三个方法（getState()、setState(int newState)、compareAndSetState(int expect,int update)）来对同步状态state进行操作，当然AQS可以确保对state的操作是安全的。

AQS通过内置的FIFO同步队列来完成资源获取线程的排队工作，如果当前线程获取同步状态失败（锁）时，AQS则会将当前线程以及等待状态等信息构造成一个节点（Node）并将其加入同步队列，同时会阻塞当前线程，当同步状态释放时，则会把节点中的线程唤醒，使其再次尝试获取同步状态。



核心方法

acquireInterruptibly(int arg)等  搞个表格



## CLH队列

![AQS-CLH队列结构](/Users/zz/coding/git_project/zhuanglegezhi/blog.github.io/resource/AQS-CLH队列结构.jpg)

入列

`addWaiter(Node node)`先通过快速尝试设置尾节点，如果失败，则调用`enq(Node node)`方法设置尾节点，两个方法都是通过CAS方法`compareAndSetTail(Node expect, Node update)`来设置尾节点，该方法可以确保节点是线程安全添加的。

```java
AbstractQueuedSynchronizer.class

private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
```

```java
private Node enq(final Node node) {
    //保证节点可以正确添加
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

![AQS-入队](/Users/zz/coding/git_project/zhuanglegezhi/blog.github.io/resource/AQS-入队.jpg)

出列

CLH同步队列遵循FIFO，首节点的线程释放同步状态后，将会唤醒它的后继节点（next），而后继节点将会在获取同步状态成功时将自己设置为首节点，这个过程非常简单，head执行该节点并断开原首节点的next和当前节点的prev即可，注意在这个过程是不需要使用CAS来保证的，因为只有一个线程能够成功获取到同步状态。

![AQS-出列](/Users/zz/coding/git_project/zhuanglegezhi/blog.github.io/resource/AQS-出列.jpg)

## 同步状态的获取和释放

### 独占式

获取

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&  acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

- `tryAcquire`：模版方法，由子类的同步组件自己实现，需保证线程安全的获取同步状态。尝试获取锁，获取成功则设置锁状态并返回true，否则返回false。
- `addWaiter`：将当前线程加入到CLH同步队列尾部。
- `acquireQueued`：当前线程会根据公平性原则来进行阻塞等待（自旋）,直到获取锁为止；并且返回当前线程在等待过程中有没有中断过。
- `selfInterrupt`：产生一个中断。

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            //只有其前驱节点为头结点才能够尝试获取同步状态
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

acquire流程如下：

![AQS-acquire流程](/Users/zz/coding/git_project/zhuanglegezhi/blog.github.io/resource/AQS-acquire流程.png)



释放

```java
public final boolean release(int arg) {
    //释放同步状态
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            //唤醒后继节点
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

### 共享式



ReentrantLock

Semaphore

CountDownLatch
