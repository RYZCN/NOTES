##  线程
### 简述
***
*   每个线程都包含有执行环境所必须的信息，其中包含:
    - 线程ID
    - 寄存器值
    - 栈
    - 调度优先级相关信息
    - 信号屏蔽字
    - errno
    - 线程私有数据
*   进程内的多个线程共享
    - 可执行代码
    - 程序全局与堆内存
    - 栈（等待堪误）
    - 文件描述符
***
### 线程控制
***
*   线程标识`pthread_t`
    - 判等
    ```c
    pthread_equal (pthread_t __thread1, pthread_t __thread2);
    ```
    - 查询
    ```c
    pthread_t pthread_self (void);
    ```
*   线程创建
    ```c
    pthread_create (pthread_t *__restrict __newthread,
			   const pthread_attr_t *__restrict __attr,
			   void *(*__start_routine) (void *),
			   void *__restrict __arg);
    ```
*   线程终止
    - 返回方式
        - 正常返回
        - 调用`pthread_exit`
            ```c
            void pthread_exit (void *__retval);
            ```
        - 被其余线程取消(仅仅请求)
            ```c
            int pthread_cancel (pthread_t __th);
            ```
    - 获取返回值
      - `pthread_exit`带有
      - `pthread_join`带有
        ``` c
        //__thread_return 无返回为PTREAD_CANCEL
        //对已经join线程调用未定义
        int pthread_join (pthread_t __th, void **__thread_return);
        ```
    - 线程退出清理
      - 线程清理函数栈压入处理程序
        ``` c
        //pthread_exit默认调用
        //pthread_cancel默认调用
        //pthread_cleanup_pop主动调用
        pthread_cleanup_push(routine, arg) 
        ```
      - 主动弹出清理函数栈
        ``` c
        //参数为0时弹出不执行
        pthread_cleanup_pop(execute)
        ```
    - 线程状态回收
      - `pthread_join`直接回收
      - `pthread_detach`直接分离
        ``` c
        int pthread_detach (pthread_t __th);
        ```
***
### 线程同步
***
* 互斥量`pthread_mutex_t`
  - 功能设计 
    原子操作查看锁的状态后选择是否从调度层面休眠阻塞线程
  - 构造与析构
    ``` c
    /* Initialize a mutex. */
    int pthread_mutex_init (pthread_mutex_t *__mutex,
			       const pthread_mutexattr_t *__mutexattr);
    /* Destroy a mutex.  */
    int pthread_mutex_destroy (pthread_mutex_t *__mutex);
     ```
  - 加锁与解锁
    ``` c
    /* Try locking a mutex.  */
    int pthread_mutex_trylock (pthread_mutex_t *__mutex);
    /* Lock a mutex.  */
    int pthread_mutex_lock (pthread_mutex_t *__mutex);
    /* Wait until lock becomes available, or specified time passes. */int pthread_mutex_timedlock (pthread_mutex_t *__restrict __mutex,
    const struct timespec *__restrict __abstime);
    ```
  - 死锁
    - 产生：线程互相请求对方线程拥有的资源
    - 避免方式：
    1. 线程请求互斥量顺序一致
    2. `try_lock`后释放占有锁再重试
* 读写锁`pthread_rwlock_t`
  - 简述：
    读写锁允许更高并行性。读写锁具有两个锁：读锁和写锁
    1. 写锁时任意加锁将被阻塞
    2. 读锁时读操作可以访问，写操作阻塞
    3. 读锁时申请写，将阻塞后续读操作，避免写操作长时间得不到资源
  - 构造与析构
    ``` c
    /* Initialize read-write lock RWLOCK using attributes ATTR, or use the default values if later is NULL.  */ 
    int pthread_rwlock_init(pthread_rwlock_t*__restrict __rwlock,const pthread_rwlockattr_t *__restrict __attr);
    /* Destroy read-write lock RWLOCK.  */
    int pthread_rwlock_destroy (pthread_rwlock_t *__rwlock);
     ```
  - 加锁与解锁
    ``` c
    /* Acquire read lock for RWLOCK.  */
     int pthread_rwlock_rdlock (pthread_rwlock_t *__rwlock);
     /* Try to acquire read lock for RWLOCK.  */
    int pthread_rwlock_tryrdlock (pthread_rwlock_t *__rwlock);
    /* Try to acquire read lock for RWLOCK or return after specfied time.  */
    int pthread_rwlock_timedrdlock (pthread_rwlock_t *__restrict __rwlock,const struct timespec *__restrict __abstime);

    /* Acquire write lock for RWLOCK.  */
    int pthread_rwlock_wrlock (pthread_rwlock_t *__rwlock);
    /* Try to acquire write lock for RWLOCK.  */
    int pthread_rwlock_trywrlock (pthread_rwlock_t *__rwlock);
    /* Try to acquire write lock for RWLOCK or return after specfied time.  */
    int pthread_rwlock_timedwrlock (pthread_rwlock_t *__restrict __rwlock,const struct timespec *__restrict __abstime);

    /* Unlock RWLOCK.  */
    int pthread_rwlock_unlock (pthread_rwlock_t *__rwlock);
    ```
* 条件变量`pthread_cond_init`
  - 简述
    线程同步机制，与互斥量使用，允许线程以无竞争方式等待特定条件发生，条件量需要互斥量保护。
  - 构造与析构
    ``` c
    /* Initialize condition variable COND using attributes ATTR, or use the default values if later is NULL.  */
    int pthread_cond_init (pthread_cond_t *__restrict __cond,const pthread_condattr_t *__restrict __cond_attr);
    /* Destroy condition variable COND.  */
    int pthread_cond_destroy (pthread_cond_t *__cond);
     ```
  - 等待与信号
    ``` c
    /* Wake up one thread waiting for condition variable COND.  */
    int pthread_cond_signal (pthread_cond_t *__cond);
    /* Wake up all threads waiting for condition variables COND.  */
    int pthread_cond_broadcast (pthread_cond_t *__cond);
    /* Wait for condition variable COND to be signaled or broadcast.MUTEX is assumed to be locked before.This function is a cancellation point and therefore not marked with __THROW.  */
    int pthread_cond_wait (pthread_cond_t *__restrict __cond,pthread_mutex_t *__restrict __mutex);
    /* Wait for condition variable COND to be signaled or broadcast until ABSTIME.  MUTEX is assumed to be locked before.  ABSTIME is an absolute time specification; zero is the beginning of the epoch(00:00:00 GMT, January 1, 1970).This function is a cancellation point and therefore not marked with __THROW.  */
    int pthread_cond_timedwait (pthread_cond_t *__restrict __cond,pthread_mutex_t *__restrict __mutex,const struct timespec *__restrict __abstime);
    ```
* 自旋锁`pthread_spinlock_t`
  - 简述
    忙等阻塞线程，有的互斥量底层实现先自旋后休眠
  - 构造与析构
    ``` c
    /* Initialize the spinlock LOCK.  If PSHARED is nonzero the spinlock can be shared between different processes.  */
    int pthread_spin_init (pthread_spinlock_t *__lock, int __pshared);
    /* Destroy the spinlock LOCK.  */
    int pthread_spin_destroy (pthread_spinlock_t *__lock);

    ```
  - 加锁与解锁
    ``` c
    /* Wait until spinlock LOCK is retrieved.  */
    int pthread_spin_lock (pthread_spinlock_t *__lock);
    /* Try to lock spinlock LOCK.  */
    int pthread_spin_trylock (pthread_spinlock_t *__lock);
    /* Release spinlock LOCK.  */
    int pthread_spin_unlock (pthread_spinlock_t *__lock);
    ```
* 屏障`pthread_barrier_t`
  - 简述
    一种多个线程参与，在到达一个节点等待全部同步后继续执行的机制
  - 构造与析构
    ``` c 
    /* Initialize BARRIER with the attributes in ATTR.  The barrier is opened when COUNT waiters arrived.  */
    int pthread_barrier_init (pthread_barrier_t *__restrict __barrier,const pthread_barrierattr_t *__restrict __attr, unsigned int __count);
    /* Destroy a previously dynamically initialized barrier BARRIER.  */
    int pthread_barrier_destroy (pthread_barrier_t *__barrier);
    ```
  - 等待
    ```c 
    /* Wait on barrier BARRIER.  */
    int pthread_barrier_wait (pthread_barrier_t *__barrier);
  ```
***
### 线程控制
***
* **线程控制描述**
线程、同步对象与各自属性对象一一对应，属性对象提供以下接口：
  1. 对象初始化为默认值`init`
  2. 销毁属性对象资源`destroy`
  3. 获取属性值`getattr`
  4. 设置属性值`setattr`
* **线程属性**
  <分离状态，栈警戒缓冲区大小，栈最低地址，栈最小长度>
  <detachstate,guardsize,stackaddr,stacksize>
  * detachstate有两个属性detach，join
  * mmap可以配合stack重设函数的栈空间
* **同步属性**
  * 互斥量
    <进程共享，健壮，类型>
    1. 进程共享默认为禁止，允许后多个进程在共享内存中进行同步
    2. 健壮的互斥量用于进程共享时，进程持有锁终止后，其余等待上锁返回时将产生需要恢复的错误码，声明互斥量一致性后可保证互斥量持续使用
    3. 类型<标准型，错误检查型，可重入，默认(可重新映射)>
  * 读写锁
    <进程共享>
    1. 进程共享默认为禁止，允许后多个进程在共享内存中进行同步
  * 条件变量
    <进程共享，时钟类型>
    1. 进程共享默认为禁止，允许后多个进程在共享内存中进行同步
    2. 系统时间，进程时间，线程时间，不带副跳数的时间
  * 屏障属性
    <进程共享>
    1. 进程共享默认为禁止，允许后多个进程在共享内存中进行同步
* **重入问题**
  线程安全：函数同一时间可以被多线程安全调用
  _r可重入版本
  信号处理程序要求：异步信号安全
* **线程特定数据**
  - 简述
  又称线程私有数据例如errno方便维护线程数据和基于进程的接口适应多线程环境。通过多个线程键使用同一个键，绑定线程的值。
  - 构造与释放键
    ``` c
    /* Create a key value identifying a location in the thread-specificdata area.  Each thread maintains a distinct thread-specific datam area.  DESTR_FUNCTION, if non-NULL, is called with the value associated to that key when the key is destroyed. DESTR_FUNCTION is not called if the value associated is NULL when the key is destroyed.  */
    int pthread_key_create (pthread_key_t *__key,void (*__destr_function) (void *));
    /* Destroy KEY.  */
    int pthread_key_delete (pthread_key_t __key);
    ```
  - 保证只初始化一个键
    ``` c
    /* Guarantee that the initialization function INIT_ROUTINE will be called only once, even if pthread_once is executed several times with the same ONCE_CONTROL argument. ONCE_CONTROL must point to a static or extern variable initialized to PTHREAD_ONCE_INIT. The initialization functions might throw exception which is why this function is not marked with __THROW.  */
    int pthread_once (pthread_once_t*__once_control,void (*__init_routine);
    ```
  - 绑定值
    ``` c
    /* Return current value of the thread-specific data slot identified by KEY.  */
    extern void *pthread_getspecific (pthread_key_t __key);
    /* Store POINTER in the thread-specific data slot identified by KEY. */
    extern int pthread_setspecific (pthread_key_t __key,const void *__pointer)  ;
    ```
* **线程取消问题**
  - 简述
    线程可被其余线程调用`pthread_cancel`申请取消，线程本身有允许取消和不允许取消的状态;
    1. *取消申请*的处理在*取消点*出现，调`pthread_cancel`时不会阻塞申请发起线程
    2. *取消点*在发起一系列系统调用时或者显式调用`pthread_testcancel`
  - 系统调用
  ``` c
    /* Set cancelability state of current thread to STATE, returning oldstate in *OLDSTATE if OLDSTATE is not NULL.  */
    extern int pthread_setcancelstate (int __state, int *__oldstate);//原子操作
    /* Set cancellation state of current thread to TYPE, returning the old type in *OLDTYPE if OLDTYPE is not NULL.  */
    int pthread_setcanceltype (int __type, int *__oldtype);
    /* Cancel THREAD immediately or at the next possibility.  */
    int pthread_cancel (pthread_t __th);
    ```
* **线程信号问题**
  - 信号发送问题：硬件故障信号发送到引起事件的线程，其余时间发送到任意进程，`pthread_kill`可指定线程发送
  - 单个线程关于信号的设施：独享信号屏蔽集，`pthread_sigmask`设置信号屏蔽集
  - 处理信号方式:`sigwait`阻塞线程到移除信号实例返回
  - 多个线程处理信号的关系：等待同一个信号的两个线程，一个捕获信号返回后，另一个返回或激活处理程序依赖于实现
  - 相关接口:
    ```c
    /* Select any of pending signals from SET or wait for any to arrive.This function is a cancellation point and therefore not marked with__THROW.  */
    int sigwait (const sigset_t *__restrict __set, int *__restrict __sig);
  ```
* **子进程与线程问题**
  - fork时子进程状态：继承父进程副本，包含同步设施的状态，但没有线程副本
  - POSIX声明：exec()前只能调用异步信号安全的代码
  - 可通过`pthread_atfork`释放两个进程的所有锁
* **IO与线程问题**
  - `pread`和`pwrite`将偏移量设定和数据读取原子化
***