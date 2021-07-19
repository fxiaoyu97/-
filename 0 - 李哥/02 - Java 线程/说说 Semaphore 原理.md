# 说说 Semaphore 原理

`Semaphore` 直译为信号。实际上 `Semaphore` 可以看做是一个信号的集合。不同的线程能够从 `Semaphore` 中获取若干个信号量。当 `Semaphore` 对象持有的信号量不足时，尝试从 `Semaphore` 中获取信号的线程将会阻塞。直到其他线程将信号量释放以后，阻塞的线程会被唤醒，重新尝试获取信号量