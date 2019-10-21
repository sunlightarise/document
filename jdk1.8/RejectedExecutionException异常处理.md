### RejectedExecutionException�쳣����

#### һ���쳣���ܷ���
ͨ����ThreadPoolExecutorler �ķ���������java.util.concurrent.RejectedExecutionException��Ҫ��2�����
1.���̳߳���ʾ�ĵ���shutdown()֮�������̳߳����ύ�����ʱ����������õľܾ�������ThreadPoolExecutor.AbortPolicy��
����쳣Ҳ�ᱻ�׳�����

2.������Ŷ��������Ϊ�н��޶��У����������˾ܾ��Ĳ���ΪThreadPoolExecutor.AbortPolicy, ���̴߳ε������ﵽ��
maximumPoolSize��ʱ�����������ύ���񣬾ͻ��׳�RejectedExecutionException�쳣Դ������
```java

/**
 * A handler for rejected tasks that throws a
 * {@code RejectedExecutionException}.
 */
public static class AbortPolicy implements RejectedExecutionHandler {
    /**
     * Creates an {@code AbortPolicy}.
     */
    public AbortPolicy() { }

    /**
     * Always throws RejectedExecutionException.
     *
     * @param r the runnable task requested to be executed
     * @param e the executor attempting to execute this task
     * @throws RejectedExecutionException always
     */
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        throw new RejectedExecutionException("Task " + r.toString() +
                                             " rejected from " +
                                             e.toString());
    }
}
```


#### ���������Ŀ����

1. �ڳ������̳߳����õ���������
```java
@Configuration
public class MarketMessageConfig {

    @Bean
    public ThreadPoolExecutor marketMessageThreadPool() {
        int nThreads = Runtime.getRuntime().availableProcessors() * 2;
        return new ThreadPoolExecutor(nThreads, nThreads * 2,
                0, TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(1024),
                new ThreadFactory() {
                    private final AtomicInteger threadNumber = new AtomicInteger(1);

                    @Override
                    public Thread newThread(Runnable r) {
                        return new Thread(r, "market-message-" + threadNumber.getAndIncrement());
                    }
                },
                new ThreadPoolExecutor.AbortPolicy());

    }
}
```
2. ��������̳߳��Ƕ�ʱ�������ύ�����õ������ܻ����һ����2000�����ϵ�������Ҫ����

#### �����������
1.��������maximumPoolSize����workQueue ���磺Integer.MAX_VALUE, ���˽��������еĴ�С����Ϊ�̳߳������̹߳����
����CPU�л��ĳɱ��϶���������Ч�ʡ�

```java
public class MarketMessageConfig {

    @Bean
    public ThreadPoolExecutor marketMessageThreadPool() {
        int nThreads = Runtime.getRuntime().availableProcessors() * 2;
        return new ThreadPoolExecutor(nThreads, nThreads * 2,
                0, TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(Integer.MAX_VALUE),
                new ThreadFactory() {
                    private final AtomicInteger threadNumber = new AtomicInteger(1);

                    @Override
                    public Thread newThread(Runnable r) {
                        return new Thread(r, "market-message-" + threadNumber.getAndIncrement());
                    }
                },
                new ThreadPoolExecutor.AbortPolicy());

    }
}
```