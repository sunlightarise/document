### ThreadPoolExecutor

#### �������ο�java doc��

* �̳߳ؽ����������ͬ������
  * �������ܣ�����ͨ����Ҫִ�д������첽���񣬿��Լ���ÿ������ĵ��ÿ��������������ṩ��һ�����ƺ͹�����Դ�������̣߳��ķ������ǵ�����������
  * ͳ����Ϣ��ÿһ��ThreadPoolExecutor��ά����һЩ������ͳ����Ϣ��������ɵ��������ȡ�
  
* Ϊ�˹㷺������������ʹ�ã������ṩ�ṩ�����ɵ��������Ϳ���չ�Թ��ӡ����ǣ��ڳ��������У�����Ԥ�����˼����̳߳أ�����ϣ��������ʹ�ø�
�����Executors�Ĺ�������ֱ��ʹ�á�
  * Executors.newCachedThreadPool �޽��޳��̳߳أ��Զ��̻߳���
  * Executors.newFixedThreadPool �̶���С���̳߳�
  * Executors.newSingleThreadExecutor ��һ��̨�߳�
  
* Core and maximun pool sizes ���ĺ�����̳߳�����

һ�� ThreadPoolExecutor ���Զ������̳߳ش�С������ getPoolSize()������ corePoolSize  ��Χ���� getCorePoolSize()�� ��maximumPoolSzie
���� getMaximumPoolSize()��. ��һ���µ��������ڷ��� execute(Runnable)�ύ�� ���� corePoolSize �߳��������С��������µ��߳����������� ��ʹ
���������߳̿��С�����г��� corePoolSize �� maximumPoolSize ��ͬ���㽫����һ���̶���С�����̳߳ء� ͨ������maximumPoolSize �����޴� ��
Integer.MAX_VALUE�������̳߳��������Ⲣ���������� ����͵��Ǻ����߳���������߳���������ʱ���ã���Ҳ���Զ�̬����ʹ��
setCorePoolSize(int) �� setMaximumPoolSize(int) ��̬����

* ������ṹ

Ĭ�������, ��ʹ�����߳���������Ϳ�ʼ���񵽴�ʱ�� ������ǿ�����д��̬ʹ�÷��� prestartCoreThread ����ƽrestarAllCoreThreads 
������������̳߳ع���һ���ǿն��С�


* �����µ��߳�

�µ��߳�ʹ�õ��� ThreadFactory ���������û�����ã� ��ʹ��executors.defaultThreadFactory() ���ڴ����߳�����ͬһ�� ThreadGroup ͬ
NORM_PRIORITY ���Ⱥͷ��ػ�״̬��ͨ���ṩ��ͬ��threadFactory, �����Ը����̵߳����֣� �߳��飬 ���ȼ��� �ػ�״̬�ȡ� ���ThreadFactory
δ�ܴ���һ���̣߳� �����ʵ�newThread���� null����ʱ�޷������̣߳� ��ִ�г��򽫼����� �������޷�ִ������ �߳�Ӧ��ӵ�� ��modifyThread��
RuntimePermission ����̳߳صĹ����̻߳������̲߳����д�Ȩ�ޣ� �������ܻᱻ������ ���ø��Ŀ��ܲ��ἰʱ��Ч�����ҹر��سǴο��ܴ�����ֹ����δ���״̬��


* ���ʱ��

�����ǰ�̳߳ؾ��ж���corePoolSize�̣߳� ��������г��� keepAliveTime ���� getKeepAliveTime(TimeUnit) �� �������߳̽�����ֹ����
�ṩ�˵��߳�δ������ʹ���Ǽ�����Դ���ĵķ���������Ժ��̳߳ر�ø��ӻ�Ծ�� �������ǵ��̡߳��˲���Ҳ����ʹ�÷��� setKeepAliveTime(long, TimeUnit)
��̬���ġ� ʹ��Long.Max_VALUE, TimeUnit.NANOSECNDS ��Ч�ؽ��ÿ����߳��ڹر�֮ǰ��ֹ��Ĭ������£��������ڶ���corePoolSize �߳�ʱ�� ���ֻ����
�����á� ���Ƿ���allowCoreThreadTimeOut(boolean) Ҳ�������ڽ������ʱ����Ӧ���ں����̣߳� ֻҪkeepAliveTime ֵ��Ϊ�㡣

* ����
�κ� BlockingQueue �����ڴ���ͱ����ύ������ ������е�ʹ�����̳߳ش�С�໥����
  * ���С��corePoolSize �߳��������У�Executor �����һ���µ��̶߳������Ŷӡ�
  * ��� corePoolSize �������߳��������У�Executor �Ὣ�����Ŷ�������������һ���µ��̡߳�
  * ��������޷��Ŷӣ� ��ᴴ��һ���ȵ��̣߳� ��������������maximumPoolSize, �������񽫱��ܾ���
�Ŷ�һ�������ֲ��ԣ�

1. ֱ���л��� һ���������еĺܺõ�Ĭ��ѡ����һ�� SynchronousQueue, �����񽻸��̣߳� ����������ơ� ��������û���߳̿����������У� ��ô
�����Ŷ������ʧ�ܣ� ��˽�����һ���µ��̡߳�������ܾ����ڲ������Ĺ�ϵ������ʱ�� �˲�¼�ɱ��������� ֱ���л�ͬ����Ҫ�����Ƶ�maximumPoolSize,
�ѱ���ܾ����ύ������ �ⷴ�������������߳������Ŀ����ԣ� �����������ƽ���ٶȱ����ǿ��Դ�����ٶȸ���ص���ʱ��

2. �޽���У�ʹ���޽���У�����LinkedBlockingQueueû��Ԥ���������ᵼ���µ������ڶ����еȴ��� ������corePoolSize �̶߳���æ�� ��ˣ�
�����ٴ���corePoolSizes�߳� ����ˣ� ���ֵ��С��ֵû��Ӱ�죩 ÿһ��������ɶ�������������ʱ�� ��������ʵ��ģ� ������񲻻�Ӱ������ִ�У����磬
����ҳ�������У� ��Ȼ�����Ŷӷ�������������ƽ��˲̬ͻ�������� ���ǵ�������������ƽ���ٶȱȿ��Դ�����ٶȸ���ʱ�� �������޽繤�����������Ŀ����ԡ�

3. �б߽�Ķ��У����޶��У�����, ArrayBlockingQueue����������ʹ�� maxPoolSizes ʱ����ֹ��Դ�ľ��������ܸ����ѵ����Ϳ��ơ����д�С�����ش�С���ܱ˴˽���:
ʹ�ô���к�С�ͳؿ�������޶ȵļ���cpuʹ���ʣ�OS��Դ���������л������������ܵ�����Ϊ�ĵ��������� �������Ƶ�����������磬���������I/O�󶨣�����ϵͳ�����ܹ�����
��������ĸ����̵߳�ʱ�䡣 ʹ��С�Ͷ��г����Ҫ�ϴ���̳߳ش�С����������ʹCPU��æ�������ܻ��������ɽ��ܵĵ��ȿ����� Ҳ��������������


* ��������

���� execute(Runnable) ���ύ����������ִ�г���ر�ʱ���ܾ������ҵ�ִ�г��������̺߳͹�����������ʹ�����ޱ߽粢�ұ���ʱ�� ����һ����£�
execute �������� RejectedExecutionHandler��rejectedExcution(Runnable, ThreadPoolExecutor) ��ķ��� RejectedExecutionHandler. 
�ṩ��4��Ԥ����Ĵ��������ԡ�

1. ��Ĭ��ThreadPoolExecutor.AbortPolicy �� ����������������RejectedExecutionException�ųⷴӦ��

2. ��ThreadPoolExecutor.CallerRunsPolicy �У�����execute ������߳��������� ���ṩ��һ���򵥵ķ������ƻ��ƣ������������ύ���ٶȡ�

3. ��ThreadPoolExecutor.DiscardPolicy �У� �򵥵�ɾ���޷�ִ�е�����

4. ��ThreadPoolExecutor.DiscardOldestPolicy �У� ���ִ�г���û�йرգ� ��������ͷ��������ɾ���� Ȼ������ִ�У����ܻ��ٴ�ʧ�ܣ� �����ظ���

���Զ����ʹ���������͵�RejectedExecutionHandler �ࡣ ��������Ҫ�ر�ע�⣬ �ر��ǵ����Ա����Ϊ�����ض��������ŶӲ����¹���ʱ��

* ���ӷ���

�����ṩ����ÿһ������ִ��֮ǰ��֮����õ�protected ���ǵ� beforeExecute(Thread, Runnable) �� afterExecute(Runnable, Throwable) 
��������Щ������������ִ�л���; ���磺���³�ʼ��ThreadLocals �� �ռ�ͳ����Ϣ�������־��Ŀ�� ���⣬ ����terminated() �ɱ����ǣ� ��ִ��
ִ�г�����ȫ��ֹ����Ҫִ�е��κ����⴦��

����ڲ����ӻ�ص������׳��쳣�� �ڲ������߳̿��ܻ�ʧ�ܲ�ͻȻ��ֹ��

* ����ά��

����getQueue() ������ʹ������н��м��Ӻ͵��ԡ� ǿ�Ҳ������˷��������κ�������Ŀ�ġ� ���ṩ�����Ŷ�����ȡ��ʱ�� �����ṩ�ķ��� remove(Runnable)
�� purge() ������Э�����д洢���ա� 

* ��չʾ��

�����󲿷ָ�����һ�������ܱ����Ĺ��ӷ����� ���磬 ������һ�����һ���򵥵���ͣ/�ָ����ܵ����ࡣ
```java
import java.util.concurrent.*;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class PausableThreadPoolExecutor extends ThreadPoolExecutor {

    private boolean isPaused;
    private ReentrantLock pauseLock = new ReentrantLock();
    private Condition unoaused = pauseLock.newCondition();

    public PausableThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory, handler);
    }

    @Override
    protected void beforeExecute(Thread t, Runnable r) {
        super.beforeExecute(t, r);
        pauseLock.lock();
        try {
            while (isPaused) {
                unoaused.wait();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            pauseLock.unlock();
        }
    }


    public void pause() {
        pauseLock.lock();
        try {
            isPaused = true;
        } finally {
            pauseLock.unlock();
        }
    }

    public void resume() {
        pauseLock.lock();
        try {
            isPaused = false;
            unoaused.signalAll();
        } finally {
            pauseLock.unlock();
        }
    }

}
``` 



