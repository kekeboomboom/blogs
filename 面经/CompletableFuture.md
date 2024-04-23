# CompletableFuture Demo

题目：有一个数据库client，从数据库中取数据A和数据B，然后求和。请使用并发的知识，尽快的完成操作。



```java
/**
 * {@code @author:} keboom
 * {@code @date:} 2024/3/8
 * {@code @description:}
 */
public class DataBaseClient {

    @SneakyThrows
    public int getAge() {
        Thread.sleep(randomSpeed());
        return 18;
    }

    @SneakyThrows
    public int getOtherAge() {
        Thread.sleep(randomSpeed());
        return 20;
    }

    private int randomSpeed() {
        return (int) (Math.random() * 1000);
    }

    @SneakyThrows
    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        DataBaseClient dataBaseClient = new DataBaseClient();
        ArrayList<Integer> resultList = new ArrayList<>(10);
        Semaphore semaphore = new Semaphore(0);
        for (int i = 0; i < 10; i++) {
            CompletableFuture<Integer> t1 = CompletableFuture.supplyAsync(dataBaseClient::getAge);
            CompletableFuture<Integer> t2 = CompletableFuture.supplyAsync(dataBaseClient::getOtherAge);
            CompletableFuture<Integer> result = t1.thenCombine(t2, (age1, age2) -> age1 + age2);
            result.thenAccept((resultAge) -> {
                resultList.add(resultAge);
                semaphore.release();
            });

            // 这是串行的写法
            // int a2 = dataBaseClient.getOtherAge();
            // int a1 = dataBaseClient.getAge();
            // resultList.add(a1 + a2);
            // 这是串行的写法
        }

        semaphore.acquire(10);
        long end = System.currentTimeMillis();
        System.out.println(end - start);
    }
}
```

main函数中是我们自己写的代码，我们通过CompletableFuture异步的从DataClient获取数据，然后求和，放到resultList中。我这里只模拟了10次。最终结果大概耗时 1.6s 左右。

如果是串行的话，是10s左右。



如果有更好的写法，欢迎评论区分享。