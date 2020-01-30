### ThreadPoolExecutor ���췽��

#### �̳߳ش���

    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)

1. corePoolSize (�̳߳صĽڱ���С)�����ύһ�������̳߳�ʱ���̳߳ػᴴ��һ���߳���ִ�����񣬼�ʱ�������еĻ����³��ܹ�ִ��������Ҳ�ᴴ���̣߳�
 �ȵ���Ҫִ�е������������̳߳ػ�����Сʱ�Ͳ����ڴ����� ��������̳߳ص� prestartAllCoreThreads ������ �̳߳ػ���ǰ�������������л����̡߳�

2. maxmumPoolSize (�̳߳��������) �̳߳����������߳����� ����������ˣ������Ѿ��������߳���С������߳����� ���̳߳ػ��ٴ����µ��߳�ִ������
ֵ��ע����ǣ����ʹ�����޽������������������û�����ˡ�

3. keepAliveTime (�̻߳ʱ��)�� �̳߳ع����߳̿��к� ���ִ���ʱ�䡣�����������ܶ࣬ ����ÿһ������ִ�е�ʱ��Ƚ϶̣����Ե���ʱ������߳������ʡ�

4. TimeUnit (�̴߳��ʱ�䵥λ)�� ��ѡ�ĵ�λ���죨Days����Сʱ��HOURS�������ӣ�MINUTES�������루MILLISECONDS����΢�루MICROSECONDS�� ǧ��֮һ���룩
������ ��NANOSECOND�� ǧ��֮һ΢�룩

5. workQueue (�������)�� ���ڱ���ȴ�ִ�е�������������С� ����ѡ��һ�¼����������С�

  * ArrayBlockingQueue ��һ������������н��������У� FIFO

  * LinkedBlockingQueue ��һ����������ṹ��������У� FIFO�� ���ܸ���ArrayBlockingQueue, Executors.newFixedThreadPool()ʹ����������С�

  * SynchronousQueue һ�����洢Ԫ�ص��������С� ÿһ�������������ȵ�����һ���̵߳����Ƴ�����������������һֱ��������״̬�����ܸ���LinkedBlockingQueue()
��������Executors.newCachedThreadPool() ʹ��������С� 

  * PriorityBlockingQueue: һ���������ȼ��������������С� 

6. threadFactory ���ڴ����̵߳Ĺ����� ����ͨ���̹߳�����ÿһ���������߳������߳����ơ��Ƿ��Ǻ�̨�̵߳����ԡ�

7. handler �����Ͳ��ԣ��� �����к��̳߳ض����ˣ� ˵���̳߳ش��ڱ���״̬����ô�����ȡһ�ֲ������������ύ������. ����ģʽ���̳߳ظ����������������β���׸����