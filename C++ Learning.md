# C++ Learning

**explicit** 指定构造函数为显式，即它不能被用于隐式转换

**noexcept** 写在函数后作为标识符，表示函数不会抛出异常，如果报错则进程直接结束。

## 线程绑核

**cpu_set_t** : 该数据集是一个位集，其中每个位代表一个CPU。

**void CPU_ZERO (cpu_set_t *set)** : 此宏初始化CPU集set成为空集。

**void CPU_SET (int cpu, cpu_set_t *set) ** : 将某个cpu加入cpu集中

### pthread库 为线程设置亲和性

```c++
pthread_attr_t attr;
pthread_attr_init(&attr);
CPU_ZERO(&cpuset);
CPU_SET(3, &cpuset);
pthread_attr_setaffinity_np(&attr, sizeof(cpu_set_t), &cpuset);
```

