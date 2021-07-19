# 说说 CountDownLatch 与 CyclicBarrier 区别

1. CountDownLatch 的作用是允许 `1` 或 `N` 个线程等待其他线程完成执行;而 CyclicBarrier 则是允许 `N` 个线程相互等待。
2. CountDownLatch 的计数器无法被重置; CyclicBarrier 的计数器可以被重置后使用,因此它被称为是循环的 barrier。