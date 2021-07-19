# synchronize 实现原理

同步代码块是使用 `monitorenter` 和 `monitorexit` 指令实现的，同步方法（在这看不出来需要看 JVM 底层实现）依靠的是方法修饰符上的 `ACC_SYNCHRONIZED` 实现。