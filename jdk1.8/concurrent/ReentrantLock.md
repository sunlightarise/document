### ReentrantLock

���������������¼���������

* ����һ�������뻥������������ʹ��synchronized ��������������ʵ���ʽ����������ͬ����Ϊ�����壬�����ܸ���ǿ��

* ReentrantLock ���ᱻ���һ�γɹ���ȡ��������δ�ͷ����ĵ��߳�ӵ�С�����û�б������߳�ӵ�У�����lock���߳̽��ɹ���ȡ����
�����ء�����ʹ��isHeldByCurrentThread() �� getHoldCount() ��������������Ƿ�����

* ����Ĺ��ŷ�������һ����ѡ�� ��ƽ ������������Ϊtrueʱ���ڶ���߳̾���������£���Щ�������ڷ�������ȴ�ʱ������̡߳�
����������޷���֤�κ��ض�����˳�������Ĭ�����ã��ǹ�ƽ������ȣ�ʹ�ù�ƽ���ĳ����ڶ��̷߳���Ч�ʽϵͣ������ڻ�����ͱ�֤������ľ�����
ʱ�����С������Ҫע����ǣ���ƽ�����ܱ�֤�̵߳��ȵĹ�ƽ�ԡ����ʹ�ù�ƽ�����ڶ��߳��е�һԱ���ܻ�ö౶�ĵĳɹ����ᣬ�������������
�������û�б�������Ŀǰ��δ������ʱ����Ҫע����ǣ� δ��ʱ��tryLock������û��ʹ�ù�ƽ���á���Ϊ��ʹ�����߳��ڵȴ���ֻҪ�����ǿ��õ�
�˷����Ϳ��Ի�óɹ�

* ���͵Ĵ��뽨��
```
    class X {
        private final ReentrantLock lock = new ReentrantLock();
        // ....
        public void m() {
            lock.lock();
            try {
                // ... method body
            } finally {
                lock.unlock();
            }
        }
    }
```

* �������ʵ��Lock�ӿ��⣬�������� isLock�� getLockQueueLength �������Լ���ص�protected��������Щ�����Լ��ͼ��ӿ������á�

* ��������л�������������Ϊ��ʽ��ͬ��һ�������л��������ڽ������״̬���������Ǳ����л���״̬����ô���ġ�

* �������֧��ͬһ���̷߳���� 2147483648 (Integer.MAX_VALUE) ���ݹ�������ͼ���������ƻᵼ�����������׳���Error��

#### ��������

�������ԣ� ����һ���̻߳�ȡ��������Ȼ�����ڳ����ڲ��ٴΣ���Σ������������һ���߳��Ѿ��������Ȼ���ڲ����Զ������
�����ɹ�����ô���ǾͿ��Գ�Ϊ����Ϊ����������

���������⣺���������ȡN�Σ���ôֻ�����ڱ��ͷ�ͬ����n��֮�󣬸�������������ȫ�ͷųɹ���
���Ĵ��룺

1.�ǹ�ƽ���ж��ܹ���ȡ����


    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        //1.�������Υ���κ��߳�ռ�У������ܱ���ǰ�̻߳�ȡ
        if (c == 0) {
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        //2.����ռ�У����ռ���߳��Ƿ��ǵ�ǰ�߳�
        else if (current == getExclusiveOwnerThread()) {
            //3.�ٴλ�ȡ����������һ
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
    
2.�ͷ���


    protected final boolean tryRelease(int releases) {
        //1.ͬ��״̬��һ
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        if (c == 0) {
            //2.ֻ�е�ǰͬ��״̬Ϊ0ʱ����ʾ�����ɹ��ͷţ�����true
            free = true;
            setExclusiveOwnerThread(null);
        }
        //3.��δ����ȫ�ͷŷ���false
        setState(c);
        return free;
    }

#### ��ƽ�Ժͷǹ�ƽ��

��ƽ�Ժͷǹ�ƽ�������Ի�ȡ�����Եģ������һ����ƽ������ô���Ļ�ȡ˳����Ͼ���ʱ�������˳������FIFO��
���Ĵ���:

1.����ǹ�ƽ��

    public ReentrantLock() {
        sync = new NonfairSync();
    }

�����ṩ��һ�ַ�ʽ������һ��booleanֵ��true��ʶΪ��ƽ����false ��ʶΪ�ǹ�ƽ��

    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }

2.��ȡ�������ǿ��Կ�����Ψһ��ͬ����������hasQueuedPredecessors���жϣ������ǰ������˵�����̱߳ȵ�ǰ�̸߳����������Դ�����ݹ�ƽ�ԣ�
��ǰ�߳�������Դʧ�ܡ�����е�ǰ�ڵ�û��ǰ���ڵ�Ļ�������������߼��жϵı�Ҫ�ԡ�**��ƽ��ÿ�ζ��Ǵ�ͬ�������еĵ�һ���ڵ��ȡ������
���ǹ�ƽ����һ�����п��ܸ��ͷ������߳��ܹ��ٴλ�ȡ����**��

    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
    
    
����

1. ��ƽ��ÿ�λ�ȡ�õ���Ϊͬ�������еĵ�һ���ڵ㣬**��֤������Դʱ���ϵľ���˳��**�� ���ǹ�ƽ���п��ܸ��ͷŵ����߳��´μ�����ȡ������
���п��ܵ��������߳���Զ�޷���ȡ������**��ɡ�����������**��

2. ��ƽ��Ϊ�˱�֤ʱ���ϵľ���˳����ҪƵ�����л������ģ����ǹ�ƽ���ή��һ�����������л����������ܿ�������ˣ�
ReentrantLockĬ��ѡ����Ƿǹ�ƽ��������Ϊ�˼���һ�����������л���**��֤ϵͳ�����������**��

### 
