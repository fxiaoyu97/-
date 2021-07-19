# 说说 CountDownLatch 原理

CountDownLatch 内部维护了一个整数 `n`，n（要大于等于0）在 **当前线程** 初始化 `CountDownLatch` 方法指定。当前线程调用 `CountDownLatch` 的 `await()` 方法阻塞当前线程，等待其他调用 `CountDownLatch` 对象的 `CountDown()` 方法的线程执行完毕。 其他线程调用该 `CountDownLatch` 的 `CountDown()` 方法，该方法会把 `n-1`，直到所有线程执行完成，`n` 等于 `0`，**当前线程** 就恢复执行。