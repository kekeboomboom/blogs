# Java多线程转账

关键词：多线程，Java

以前的一道面试题，要求是使用Java多线程，实现一个转账业务。不考虑数据库，不考虑其他第三方系统。只考虑当前Java程序内各个账户进行转账，保证转账金额正确性和转账功能效率。

想起那大约还是两年前，是线上面试，面试官给完题目就关闭视频通话，让我自己去写代码，并且告知可以看浏览器。

要是放到现在可不行了哈！直接ChatGPT，分分钟就写好了，而且各种说辞都能准备好！

---

## 实现代码

[代码地址](https://github.com/kekeboomboom/concurrent_keboom/tree/master/src/main/java/transfer)

这里使用充血模型：Account

```java
package transfer;

import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @author keboom
 * @date 2021/8/24
 */
public class Account {

    private Long id;
    private double balance;
    private ReentrantLock lock = new ReentrantLock();
    // 记录转账次数，没什么用，只是为了验证transfer执行的次数
    AtomicInteger count = new AtomicInteger(0);

    public Account(Long id, double balance) {
        this.id = id;
        this.balance = balance;
    }

    public void deposit(double amount) {
        lock.lock();
        balance += amount;
        lock.unlock();
    }

    public void withdraw(double amount) {
        lock.lock();
        balance -= amount;
        lock.unlock();
    }

    public double getBalance() {
        return balance;
    }

    public boolean transfer(Account target, double amount) {
        if (this == target) {
            System.out.println("不能自己转给自己");
            return false; // 防止自己向自己转账
        }

        if (this.getBalance() < amount) {
            System.out.println("余额不足，转账失败");
            return false; // 余额不足，转账失败
        }
        // avoid deadlock
        // 使用两把锁，按照账户的 hashcode 顺序来避免死锁
        Account firstLock = this.hashCode() > target.hashCode() ? this : target;
        Account secondLock = this.hashCode() > target.hashCode() ? target : this;
        boolean flag = true;
        while (flag) {
            if (firstLock.lock.tryLock()) {
                try {
                    if (secondLock.lock.tryLock()) {
                        try {
                            this.withdraw(amount);
                            target.deposit(amount);
                            count.incrementAndGet();
                            flag = false;
                            Thread.sleep(1);
                        } catch (Exception e) {
                            throw new RuntimeException(e);
                        } finally {
                            secondLock.lock.unlock();
                        }
                    }
                } finally {
                    firstLock.lock.unlock();
                }
            }
        }
        return true;
    }

}

```

首先是余额的正确性，`balance` 并不需要使用 `volatile` 关键字修饰，因为 `balance` 变量的访问是通过锁来保护的。在使用 `ReentrantLock` 进行加锁和解锁的过程中，会保证对 `balance` 变量的读取和写入操作在同一时刻只能被一个线程访问，从而确保了线程间的可见性和原子性。

转账方法，会有多线程进行调用，因此需要锁来控制。为了防止死锁问题，我们通过hash值来得到一个加锁顺序。



接下来使用线程池并发调用转账方法：

```java
package transfer;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

/**
 * 这个平台来进行转账操作
 *
 * @author keboom
 * @date 2023/11/28
 */
public class TransferTask {

    public static void main(String[] args) throws InterruptedException {
        Account accountA = new Account(1L, 100000);
        Account accountB = new Account(1L, 100000);

        ExecutorService toA = Executors.newFixedThreadPool(3);
        ExecutorService toB = Executors.newFixedThreadPool(3);


        for (int i = 0; i < 1000; i++) {
            toB.execute(() -> {
                accountB.transfer(accountA, 10);
            });
        }

        for (int i = 0; i < 1000; i++) {
            toA.execute(() -> {
                accountA.transfer(accountB, 10);
            });
        }

        toA.shutdown();
        toB.shutdown();

        toA.awaitTermination(1, TimeUnit.MINUTES);
        toB.awaitTermination(1, TimeUnit.MINUTES);
        System.out.println("accountA: " + accountA.getBalance()+ "  " + accountA.count);
        System.out.println("accountB: " + accountB.getBalance()+ "  " + accountB.count);

    }
}

```

这里我们总共用6个线程去模拟。因为同一时刻只有一个线程能执行转账操作。如果线程过多，会发生大量的线程争抢，会有一些线程饿死。

比如我实验了使用20个线程并发执行转账。发现一个是执行时间变得很长，而且到最后返回时结果也不正确，发现执行次数少了。猜测应该是一些线程饿死，导致一些任务并没有执行。



