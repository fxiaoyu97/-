# 说说 CyclicBarrier 原理

`CyclicBarrier` 是一个同步辅助类,允许一组线程互相等待,直到到达某个公共屏障点(CommonBarrierPoint)。因为该 `barrier` 在释放等待线程后可以重用,所以称它为循环的 `barrier`。