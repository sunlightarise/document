### ThreadPoolExecutor 

#### ִ��ԭ��
 
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


�����ж���������ͼ��ʾ

![avatar](../image/1.png)


��execute �����У��õ��� double-check ��˼�룬 ���ǿ����������벢û��ͬ�����ƶ��ǻ���
�ֹ�����check , ���������Դ��������andWorker(Runnable firestTask, boolean core) ������ע�����������е����ַ�ʽ��
  
  * andWorker(command, true): ���������߳�ִ������
  * andWorker(command, false): �����Ǻ����߳�����
  * andWorker(null, false): �����Ǻ����̣߳���ǰ����Ϊ��
  
andWorker �ķ���ֵ�� boolean, ����֤�����ɹ����������ʱ�� andWorker ��������:
   
```

private boolean andWorker(Runable firestTask, boolean core) {
    // ��һ���֣� ������CAS���ض�ctl �Ƚ�ϣ�ֱ��ȷ���Ƿ���Դ���worker
    // ����������ѭ���������������򷵻�false
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);
        
        //Check is queue empty only if necessary.
        if (rs > SHUTDOWN &&
            !(rs == SHUTDOWN && firstTask == null && !workQueue.isEmpty())
            ) {
            return false;
        }
        
       for (;;) {
           int wc = workCountOf(c);
           if (wc >= CAPACITY || 
               wc >= (core ? corePoolSize : maximumPoolSize)) {
               return false;
           }
           if (compareAndIncrementWorkCount(c)) {
               break retry;
           }
           c = ctl.get();
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
        w = new Worker(firstTask);// ����worker
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // ��ȡ�����Ժ�������ctl, ��������һ����ȡ����������߳̿��ܻ�ı�runState
                // �� ThreadFactory ����ʧ�ܻ��سǴα� shutdown ��
                
                int rs = runStateOf(ctl.get());
                if(rs < SHUTDOWN || 
                 (rs == SHUTDOWN && firstTask == null)){
                    if (t.isAlive) {
                        throw new IllegalThreadStateException();
                    }
                    worker.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize) {
                        largestPoolSize = s;
                    }
                    workerAdded = true;
                }
            
            } finally {
                mainLock.unlock();
            }
            
            if (workerAdded) {
                t.start(); //�����߳�
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

andWorker �Ĺ�����Ϊ2����

  * ��һ���֣� ԭ�Ӳ����� �ж��Ƿ���Դ���worker. ͨ�������� CAS�� ctl �Ȳ����� �жϼ����������Ƿ���false, ��������һ��̡ܶ� 
  
  * �ڶ����֣� ͬ������worker, �������̡߳�
  