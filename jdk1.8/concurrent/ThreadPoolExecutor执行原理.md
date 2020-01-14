### ThreadPoolExecutorִ��ԭ�� 

#### execute ����
 
 ����ThreadPoolExecutor��ִ��ԭ�� ֱ�Ӵ�execute ������ʼ��
 
```
public void execute(Runnbale command) {
    if (commond == null) {
        throw new NullPointerException()
    }
    
    int c = ctl.get();
    
    //1.�����߳� < �����߳�
    if (workerCountOf(c) < corePoolSize) {
        if (andWork(command, true)) {
            retrun;
        }
        c = ctl.get();
    }
    
    //2������̬�� �����Խ�����������
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command)) {
            reject(command);
        } else if (workerCountOf(recheck == 0)) {
            andWorker(null, false);
        }
    } 
    // 3��ʹ�ó���ʹ������߳�����
    else if (!addWorker(command, flase)) {
        reject(command);
    }
}
```

��ִ��execute() ����ʱ���״̬һֱ��RUNNINGʱ�� ��ִ�й������£�

1. ��� workerCount < corePoolSize, �򴴽�������һ���³���ִ�����ύ������;
2. ��� workerCount >= corePoolSize, ���̳߳��ڵ���������δ��, ��������ӵ�������������;
3. ��� workerCount >= corePoolSize && workerCount < maximumPoolSize, ���̳߳��ڵ�������������, �򴴽����ж�һ���߳���ִ�����ύ������
4. ��� workerCount >= maximumPoolSize, �����̳߳ص��������������� ����ݾܾ���������������Ĭ�ϵĴ���ʽ��ֱ���׳��쳣��

ע�⣺ andWorker(null, false); Ҳ�Ǵ���һ���߳�, ����û�д�������, ��Ϊ�����Ѿ�����ӵ���workerQueue���ˣ�����worker��ִ�е�ʱ��
��ֱ�Ӵ�workQueue �л�ȡ�������ԣ���workerCountOf(recheck) == 0 ʱִ��andWorker(null); Ҳ��Ϊ�˱�֤�سǴ���RUNNING ״̬�±���Ҫ��һ���߳���ִ������

�����ж���������ͼ��ʾ

![avatar](image/1.png)

 
#### addWorker ����

��execute �����У��õ��� double-check ��˼�룬 ���ǿ����������벢û��ͬ�����ƶ��ǻ���
�ֹ�����check , ���������Դ��������andWorker(Runnable firestTask, boolean core) ������ע�����������е����ַ�ʽ��
  
  * andWorker(command, true): ���������߳�ִ������
  * andWorker(command, false): �����Ǻ����߳�����
  * andWorker(null, false): �����Ǻ����̣߳���ǰ����Ϊ��
  
andWorker �ķ���ֵ�� boolean, ����֤�����ɹ����������ʱ�� andWorker ��������:
   
```

private boolean andWorker(Runable firestTask, boolean core) {
    retry:
    //�����߳�ִ�й����У�������������ܴ��ڣ�ͨ�������ķ�ʽ����֤worker������
    for (;;) {
        int c = ctl.get();
        //��ȡ�߳�����״̬
        int rs = runStateOf(c);
        
        //��� rs >= SHUTDOWN, ���ʾ��ʱ���ٽ���������
        //�������������� ͨ�� && ���ӣ�ֻҪһ��������ͷ���false;
        //1. rs == SHUTDOWN, ��ʾ�ر�״̬�����ٽ����ύ�����񣬵�ȴ���Լ������������������Ѿ����ڵ�����;
        //2. firstTask Ϊ��
        //3. Check is queue empty only if necessary.
        if (rs > SHUTDOWN &&
            !(rs == SHUTDOWN && firstTask == null && !workQueue.isEmpty())
            ) {
            return false;
        }
        
       for (;;) {
            //��ȡ�̳߳����߳���
           int wc = workCountOf(c);
           
           //����߳��� >= CAPACITY, Ҳ����ctl �ĵ�29λ�����ֵ���򷵻�false;
           //����core���ж� �����߳�������������corePoolSize����maximumPoolSize;
           //���core��true��ʾ����corePoolSize���Ƚ�
           //���core��false��ʾ����maximumPoolSize���Ƚ�
           if (wc >= CAPACITY || 
               wc >= (core ? corePoolSize : maximumPoolSize)) {
               return false;
           }
           //ͨ��CASԭ�ӵķ�ʽ�������߳�������
           //����ɹ���������һ��forѭ��;
           if (compareAndIncrementWorkCount(c)) {
               break retry;
           }
           c = ctl.get();
           //����������е�״̬������rs, ��˵���̳߳ص�״̬�Ѿ��ı��ˣ��򷵻ص�һ��forѭ������ִ��
           if (runStateOf(c) != rs) {
               countinue retry;
           }
           // else CAS failded due to workerCount changge; retry inner loop
       }
    }
    
    // �ڶ����֣�����worker, �ⲿ��ʹ��ReentrantLock ��
    boolean workerStated = false; // �߳�������ʶλ
    boolean workerAdded = false; // �߳��Ƿ����workers ��־λ
    Worker w = null;
    try {
        //����firstTask����worker
        w = new Worker(firstTask);
        //ÿһ��Worker����ᴴ��һ���߳�
        final Thread t = w.thread;
        if (t != null) {
            //������������
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                //��ȡ�̳߳ص�״̬
                int rs = runStateOf(ctl.get());
                
                //�̳߳ص�״̬С�� SHUTDOWN, ��ʾ�̳߳ش���RUNNING״̬
                //���rs ��RUNNING״̬��rs ��SHUTDOWN״̬����firstTaskΪnull, ���̳߳�������߳�
                //��Ϊ��SHUTDOWN״̬ʱ����������µ����� ���ǻ��Ǵ���workerQueue�е�����
                if(rs < SHUTDOWN || 
                 (rs == SHUTDOWN && firstTask == null)){
                    if (t.isAlive) {
                        throw new IllegalThreadStateException();
                    }
                    //worker ��һ��hashset
                    worker.add(w);
                    int s = workers.size();
                    //largestPoolSize��¼�̳߳��г��ֵ������߳�����
                    if (s > largestPoolSize) {
                        largestPoolSize = s;
                    }
                    workerAdded = true;
                }
            
            } finally {
                mainLock.unlock();
            }
            
            if (workerAdded) {
                //�����̣߳�Worker ��ʵ����running ��������ʱ�����Worker��run������
                t.start(); 
                workerStated = true;
            }
        }
    } finally {
        if (workerAdded) {
            addWorkerFailed(w); // ʧ�ܲ���
        }
    }
    return workerStated;
}
```

#### Worker��
�̳߳���ÿһ���̶߳��󱻷�װ����һ��Worker���� ThreadPool ά���ľ���һ��Worker����Worker��̳���AQS�� ��ʵ����Runnable�ӿڡ�
���а���������Ҫ������; firstTask �������洫�������, thread ���ڵ��õĹ��췽����ͨ��ThreadFactory ���������̣߳�����������������̡߳�

```java
private final class Worker
        extends AbstractQueuedSynchronizer 
        implements Runnable {

    final Thread thread;
    Runnable firstTask;
    volatile long completedTasks;
    
    Worker (Runnable firstTask)  {
        //��state����Ϊ-1����ֹ�ն�֪������runWorker����
        //��ΪAQSĬ��state��0, ����մ���һ��Worker���� ��û��ִ������ʱ����ʱ��Ӧ�ñ��ж�
        setState(-1);
        this.firstTask = firstTask;
        //����һ���߳�, newThread��������Ĳ�����this, ��ΪWorker����̳���Runnable�ӿڣ� Ҳ����һ���߳�;
        //����Worker������������ʱ��ڵ���Worker�����е�run����
        this.thread = getThreadFactory().newThread(this);
    }
}
```
Worker ��̳���AQS, ʹ��AQS��ʵ�ֶ�ռ�����Ĺ��ܣ� Ϊʲô��ʹ�� ReentranLock ��ʵ�֣� ���Կ���tryAcquire���������ǲ���������ģ� ��
ReentrantLock ����������ġ�
1. lock ����һ����ȡ��ռ���� ��ʾ��ǰ�߳�����ִ��������;
2. ���ִ��������Ӧ���ж��߳�;
3. ������߳����ڲ��Ƕ�ռ����״̬��Ҳ�ǿ���״̬��˵����û�д���������ʱ���ԶԸ��߳̽����ж�;
4. �̳߳���ִ��shutdown ������ tryTerminate ����ʱ����� interruptldleWorkers �������жϿ����߳�, interruptldleWorkers ������ʹ��
tryLock�������ж��̳߳��е��߳��Ƿ��ǿ���״̬��
5. ֮��������Ϊ��������ģ� ����Ϊ���������setCorePoolSize �����̳߳ؿ��Ƶķ���ʱ�������ж��������е��߳����ԣ�Worker �̳���AQS�������ж��̳߳�
�Ƿ�����Լ��Ƿ��ڱ��жϡ�

```
protected boolean tryAcquire(int unused) {
    //cas�޸�state, ��������;
    //state����0���жϣ�����worker ���췽���н�state����Ϊ-1��Ϊ�˽�ֹ��ִ������ǰ���߳̽����ж�;
    //��ˣ���runWorker�����л��ȵ���Worker�����е�unlock������state����Ϊ0
    if (compareAndSetState(0, 1)) {
        setExclusiveOwnerThread(Thread.currentThread());
        return true;
    }
    return false;
}
```

#### runWorker����
��Worker����run����������runWorker������ִ������
```
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    //��ȡ��һ������
    Runable task = w.firstTask;
    w.firstTask = null;
    //�����ж�
    w.unlock();
    //�Ƿ����쳣�˳�ѭ��
    boolean completedAbruptly = true;
    try {
        //���taskΪ�գ���ͨ��getTask����ȡ����
        while (task != null || (task = getTask() != null)) {
            w.lock();
            
            //����̳߳�����ֹͣ����ôҪ��֤��ǰ�߳����ж�״̬;
            //������ǵĻ�����Ҫ��֤��ǰ�̲߳����ж�״̬
            if (runStateAtLeast(ctl.get()) ||
                (Thread.interrupted() &&
                    runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted()) {
                wt.interrupt();
            }
            
            try {
                //beforeExecute �� afterExecute ������������ʵ�ֵ�
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    //ͨ������ʽִ�У������̷߳�ʽ
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x;
                    throw x;
                } catch (Error x) {
                    thrown = x;
                    throw x;
                } catch (Throwable x) {
                    thrown = x;
                    throw x;
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.cimpletedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = true;
        
    } finally {
        //processWorkerExit���completedAbruptly �����жϣ���ʾ��ִ�й������Ƿ�����쳣
        processWorkerExit(w, completedAbruptly);
    }
} 
```
�ܽ᣺
1. while ѭ�����ϵ�ͨ��getTask��������ȡ����;
2. getTask ���������������л�ȡ����
3. ����̳߳�����ֹͣ, ��ôҪ��֤��ǰ�̴߳����ж�״̬������Ҫ��֤��ǩ�̲߳����ж�״̬
4. ���� task.run() ִ������;
5. ���task Ϊnull �������ѭ����ִ��processWorkerExit ����; 
6. runWorker ����ִ����ϣ�Ҳ������Worker �е�run ����ִ����ϣ������̡߳�

#### getTask����
```
private Runnable getTask() {
    //timeOut ������ֵ��ʾ�ϴδ����������л�ȡ�����Ƿ�ʱ
    boolean timeOut = false;
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);
        
        //��� rs >= SHUTDOWN, ��ʾ�̳߳ط�RUNNING״̬, ��Ҫ�ٴ��ж�:
        //1. rs >= STOP, �̳߳��Ƿ���STOP
        //2. ���������Ƿ�Ϊ��
        //������������֮һ����workerCount��һ, ������null;
        //��Ϊ�����ǰ�̳߳ص�״̬����STOP�����ϻ����Ϊ�գ����ܴ����������л�ȡ����
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }
        
        int wc = workerCountOf();
        
        //timed ���������ж��Ƿ���Ҫ���г�ʱ����;
        //allowCoreThreadTimeOut Ĭ��Ϊfalse, Ҳ���Ǻ����̲߳�������г�ʱ;
        //wc > corePoolSuze, ��ʾ��ǰ�߳����������߳�����
        //���ڳ��������߳��������̣߳���Ҫ���г�ʱ����
        boolean timed = allowCoreThreadTimeOut || wc < corePoolSize;
        
        //wc > maximumPoolSize ���������Ϊ�����ڴ˷���ִ�н׶�ͬ��ִ����setMaximumPoolSzie ����
        //timed && timeOut ���Ϊtrue, ��ʾ��ǰ������Ҫ���г�ʱ���ƣ������ϴδ����������л�ȡ�������˳�ʱ
        //�������жϣ� �����Ч������������1������workerQueueΪ�գ���ô������workerCout ��һ;
        //�����һʧ��Ҳ��������;
        //���wc == 1ʱ, Ҳ˵����ǰ�߳����̳߳ص�Ψһ�߳� 
        if ((wc > maximumPoolSize) || (timed && timeOut)
            && (wc > 1 || workerQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c)) {
                return null;
            }
            continue;
        }
        
        //timedΪtrue, ��ͨ��workQueue��poll�������г�ʱ���ƣ����keepAliveTime ʱ����û�л�ȡ���� ����ͨ��take������ �������Ϊ��
        //��take�������������е������в�Ϊ��
        try {
            Runable r = timed ? 
                workerQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workerQueue.take();
            if (r != null) {
                //��� r == null, ˵���Ѿ���ʱ�ˣ�timeOut = true;
                return r;
            }
            timeOut = false;
        } catch (InterruptedException retry) {
            //�����ȡ����ʱ��ǰ�̷߳������жϣ���timeOut = true
            timeOut = false;
        }
    } 
}
```
ע�⣺�ڶ���if�ж�, Ŀ����Ϊ�˿����̳߳ص���Ч�߳������������ķ����õ�����execute����ʱ�������ǰ�̳߳ص��߳���������corePoolSize ��
С��maximumPoolSize, ����������������ʱ�� �����ͨ�����ӹ����̡߳� ������������߳��ڳ�ʱʱ����û�л�ȡ������
timeOut=true, ˵��workQueue Ϊ�գ�Ҳ����˵��ǰ�̳߳ز���Ҫ��ô���̳߳ز���Ҫ��ô���߳���ִ�������ˣ����԰Ѷ���Ĵ�corePoolSize �������߳�
���ٵ�����֤�߳�������corePoolSize���ɡ�

ʲôʱ��������߳�? ��Ȼ�� runWorker ����ִ�����Ҳ����Worker����ִ����ɺ���JVM�Զ����ա�
#### processWorkerExit����
```
private void processWorkerExit(Worker w, boolean completedAbruptly) {
    
    //���completedAbruptly Ϊtrue, ��˵���߳�ִ�г����쳣����Ҫ��workerCount������һ
    //���completedAbruptly Ϊfalse, ˵��getTask�������Ѿ���workerCount��һ�� ���ﲻ��Ҫ�ټ�
    if (completedAbruptly) {
        decrementWorkerCount();
    }
    
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        //ͳ�����������
        completedTaskCount += w.completedTask;
        //��workers���Ƴ���Ҳ���Ǳ�ʾ���̳߳����Ƴ�һ�������߳�
        workers.remove(w);
    } finally {
        mainLock.unlock();
    }
    
    //���Ӻ���
    tryTerminate();
    
    int c = ctl.get();
    
    //��ǰ�߳���RUNNING ���� SHUTDOWN ʱ, ���worker ���쳣������ô��ֱ�� addWorker; 
    //���allowCoreThreadTimeOut = true, ��ô�ȴ��������������ٱ���һ�� worker;
    //���allowCoreThreadTimeOut = flase, workerCount ���� corePoolSize
    if (runStateLessThan(c, STOP)) {
        if (!completedAbruptly) {
            int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
            if (min == 0 && workQueue,isEmpty()) {
                min = 1;
            }
            if (workerCountOf(c) >= min) {
                return;
            }
            addWorker(null, false);
        }
    }
}
```
���ˣ�processWorkerExitִ����ɺ󣬹����̱߳����� 

#### ����ִ������

�����̵߳��������ڣ��� execute ������ʼ�� Worker ʹ��ThreadFactory �����͵Ĺ����̣߳�runWorker ͨ��getTask��ȡ����Ȼ��ִ������ 
���getTask����null, ����processWorkerExit, �����߳̽�����

![avatar](image/1.png)