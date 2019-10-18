## CLH��MCS����ԭ��ʵ��

### ����
* SMP (Symmetric Multi-Processor)

 �Գƶദ�����ṹ�� ��������ڷǶԳƶ�����������Եġ�Ӧ��ʮ�ֹ㷺�Ĳ��м����������ּܹ��У�һ�������ɶ��CPU��ɣ��������ڴ��������Դ��
���е�CPU������ƽ�ȵķ����ڴ棬I/O���ⲿ�жϡ���Ȼͬʱʹ�ö��CPU�����Ǵӹ���ĽǶ����������ǵı��־���һ̨����һ��������ϵͳ������жԳ�
�ķֲ��ڶ��CPU�ϣ��Ӷ���������������ϵͳ�����ݴ�����������������CPU���������ӣ�ÿһ��CPU����Ҫ������ͬ���ڴ���Դ�������ڴ���ܻ��Ϊϵͳ
ƿ��������CPU��Դ�˷ѡ�

* NUMA (Non-Uniform Memory Access)

 ��һ�´���ʣ���CPU��ΪCPUģ�飬ÿ��CPUģ���ж��CPU��ɣ����Ҿ��ж����ı����ڴ桢I/O�۵ȣ�ģ��֮�����ͨ������ģ���໥���ʣ����ʱ����ڴ�
 ������CPUģ����ڴ棩���ٶ�ԶԶ���ڷ���Զ���ڴ棨������CPUģ����ڴ棩���ٶȣ���Ҳ�Ƿ�һ�´洢���ʵ����ɡ�NUMA�Ϻõؽ��SMP����չ���⣬��CPU
 ���ӵ�ʱ����Ϊ����Զ���ڴ��ӳ�ԶԶ���ڱ����ڴ棬����ϵͳ���ܲ����������ӡ�


### CLH ��
  CLH��һ�ֻ��ڵ�������ĸ����ܡ���ƽ��������������������߳�ͨ��ǰ���ڵ�ı���������������ǰ�ýڵ�����󣬵�ǰ�ڵ�������ѡ�������м�����
��SMP�ܹ��£�CLH���������ơ���NUMA�ܹ��£����ɵ�ǰ�ڵ���ǰ���ڵ��ڲ�ͬ��CPUģ���£���CPUģ����������Ŀ�������MCS����������NUMA�ܹ�

�������̣�
1. ��ȡ��ǰ�³ǵ����ڵ㣬���Ϊ�գ����г�ʼ����
2. ����������ȡ�����β�ڵ㣬������ǰ�ڵ�����Ϊβ�ڵ㣬��ʱԭ���Ľڵ�Ϊ��ǰ�ڵ��ǰ�ýڵ㡣
3. ���β�ڵ�Ϊ�գ���ʾ��ǰ�ڵ��ǵ�һ���ڵ㡣ֱ�Ӽ����ɹ���
4. ���β�ڵ㲻Ϊ�գ������ǰ�ýڵ����ֵ��locked == true�� ����������֪��ǰ�ýڵ������Ϊfalse��

�������̣�
1. ��ȡ��ǰ�̶߳�Ӧ���Ľڵ㣬����ڵ�Ϊ�ջ���Ϊfalse�������������ֱ�ӷ���
2. ͬ������Ϊβ�ڵ㸳��ֵ�����Ʋ��ɹ���ʾ��ǰ�ڵ㲻��β�ڵ㣬����Ҫ����ǰ�ڵ��locked = false�����ڵ㡣�����ǰ�ڵ���β�ڵ㣬������Ϊ�ýڵ����á�

Demo ���£�
```java
public class CLHLock {

    private final AtomicReference<Node> tail;
    private final ThreadLocal<Node> myNode;
    private final ThreadLocal<Node> myPred;

    public CLHLock() {
        tail = new AtomicReference<>(new Node());
        myNode = ThreadLocal.withInitial(Node::new);
        myPred = ThreadLocal.withInitial(null);
    }

    public void lock() {
        Node node = myNode.get();
        node.locked = true;

        Node pred = tail.getAndSet(node);
        myPred.set(pred);
        while (pred.locked) {
        }
    }

    public void unLock() {
        Node node = myNode.get();
        node.locked = false;
        myNode.set(myPred.get());
    }

    static class Node {
        volatile boolean locked = false;
    }


    private static ExecutorService executorService = Executors.newFixedThreadPool(4);

    public static void main(String[] args) {
        CLHLock lock = new CLHLock();
        Runnable runnable = new Runnable() {
            private int a;

            @Override
            public void run() {
                lock.lock();
                try {
                    a++;
                    Thread.sleep(500);
                    System.out.println("Thread Name:" + Thread.currentThread() + " ; a=" + a);
                } catch (Throwable t) {
                    t.printStackTrace();
                } finally {
                    lock.unLock();
                }
            }
        };
        for (int i = 0; i < 10; i++) {
            executorService.submit(runnable);
        }
    }
}
```

### MCS ��
  MSC��CLH���Ĳ�ͬ��������������ʽ������ʽ�������߳���ѡ�Ĺ���ͬ��CLH����ǰ���ڵ��locked��������ת�ȴ�����MCS���Լ��Ľڵ���locked��
����ѡ�ȴ���������ˣ��������CLH��NUMAϵͳ�ܹ��л�ȡlocked��״̬�ڴ��Զ�����⡣

MCS������ʵ�ֹ���
1. ���г�ʼ��û�нڵ㣬taIl = null
2. �߳�A��Ҫ��ȡ�������Լ����ڶ�β����������һ���ڵ㣬����locaked��Ϊfalse
3. �߳�B���߳�C��̼�����У�a -> next = b, b -> next = c, B ��Cû�л�ȡ���������ڵȴ�״̬������locked��Ϊtrue�� βָ��ָ���߳�C��Ӧ�Ľڵ㡣
4. �߳�A�ͷ�����˳������nextָ���ҵ����߳�B������B��locked ������Ϊfalse ,��һ�����ᴥ���߳�B��ȡ����
Demo ���£�
```java
public class MCSLock {

    private final AtomicReference<Node> tail;
    private final ThreadLocal<Node> myNode;

    public MCSLock(AtomicReference<Node> tail, ThreadLocal<Node> myNode) {
        this.tail = tail;
        this.myNode = myNode;
    }

    public MCSLock() {
        this.tail = new AtomicReference<>();
        this.myNode = ThreadLocal.withInitial(Node::new);
    }

    public void lock() {
        System.out.println("Thread Name:" + Thread.currentThread() + " ; - lock start --------------------");
        Node node = myNode.get();
        Node pred = tail.getAndSet(node);
        if (pred != null) {
            node.locked = true;
            pred.next = node;
            while (node.locked) {

            }
        }
        System.out.println("Thread Name:" + Thread.currentThread() + " ; - lock succ --------------------");
    }

    public void unLock() {
        System.out.println("Thread Name:" + Thread.currentThread() + " ; - unLock start --------------------");
        Node node = myNode.get();
        if (node.next == null) {
            if (tail.compareAndSet(node, null)) {
                return;
            }

            while (node.next == null) {
            }
        }
        node.next.locked = false;
        node.next = null;
        System.out.println("Thread Name:" + Thread.currentThread() + " ; - unlock succ --------------------");

    }

    class Node {
        volatile boolean locked = false;
        Node next = null;
    }


    private static ExecutorService executorService = Executors.newFixedThreadPool(4);

    public static void main(String[] args) {
        MCSLock lock = new MCSLock();
        Runnable runnable = new Runnable() {
            private int a;

            @Override
            public void run() {
                lock.lock();
                try {
                    for (int i = 0; i < 2; i++) {
                        a++;
                        Thread.sleep(500);
                    }
                    System.out.println("Thread Name:" + Thread.currentThread() + " ; a=" + a);
                } catch (Throwable t) {
                    t.printStackTrace();
                } finally {
                    lock.unLock();
                }
            }
        };
        for (int i = 0; i < 10; i++) {
            executorService.submit(runnable);
        }
    }
}
```