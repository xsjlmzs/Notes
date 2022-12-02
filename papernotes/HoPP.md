# HoPP

背景：

disaggregated memory systems 需要：

1、透明度。原有应用不用修改。

2、性能。

3、通用性，支持广泛的数据中心应用。

现有预取技术的限制：

1、仅使用 page fault 时的地址训练算法

2、被预取的页面仍然会触发 page fault 因为 page table 在预取期间还未建立（？）

方案：

**通过一个不同的预取机制解决 kernel-based 系统的虚拟内存瓶颈**

## 2 Design and Implementation of of HoPP

将 cache line-sized 的 LCC miss 转换为page级别的跟踪