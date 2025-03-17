# High CPU usage by kworker/u64:2-events-unbound

症状：kworker 有几个单线程的进程，几乎占用100%的单核心，导致系统卡顿无法使用。

解决办法：增加内核参数 `amdgpu.sg_display=0 amdgpu.dcdebugmask=0x10`

References:
- [Reddit](https://www.reddit.com/r/archlinux/comments/1i8qt62/high_cpu_usage_by_kworkeru642eventsunbound/)
- [Arch Wiki - Kernel Parameters](https://wiki.archlinux.org/title/Kernel_parameters#GRUB)