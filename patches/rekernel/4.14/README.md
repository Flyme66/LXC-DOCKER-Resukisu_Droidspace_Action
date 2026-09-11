# Re:Kernel 4.14（MTK realme AndroidS 树）

面向 `MOSSVENC/realme_...AndroidS-kernel-source`（MTK 4.14.186）的内核内嵌
形态 Re:Kernel；上游素材与 4.9 同源（`Sakion-Team/Re-Kernel` v11.6，快照
`localworkspace/mirrors/Re-Kernel` @`5adec48`）。

## 组成

| 补丁 | 内容 |
|---|---|
| `0001-drivers-rekernel-netlink-server.patch` | 与 4.9 同一份（新增 `drivers/rekernel/` 四件，路径与目标树无关） |
| `0002-binder-frozen-transaction-notify.patch` | `drivers/android/binder.c`：include + `binder_transaction()` 内 target_proc 建立后的上报（reply / transaction / oneway-async 空间不足） |
| `0003-signal-frozen-kill-notify.patch` | `kernel/signal.c`：include + `do_send_sig_info()` 上报 |

## 与 4.9 port 的差异

- binder.c 锚点是 MTK 旧式引用计数写法 `target_proc->tmp_ref++;`（4.9 树为
  `atomic_inc(&target_proc->tmp_ref);`），插入点取
  `binder_inner_proc_unlock()` 之后、厂商 `#ifdef BINDER_WATCHDOG` 块之前。
- `rekernel.h` 沿用同一份 4.9 适配（`JOBCTL_TRAP_FREEZE` 回退；两树均无该位）。
- async 事务合并块同样省略：本树 `0x40` 是厂商 `TF_ASYNC_BOOST`。

## 状态

- 补丁面：对 `f0c2afc4d` 洁净树（`mirrors/q2-kernel-4.14`）三件顺序 `git apply`
  通过；引用符号均在树内：`struct binder_proc.tsk`、`alloc.free_async_space` /
  `alloc.buffer_size`、`cgroup_freezing`，`task_tgid_nr` 经
  `net/sock.h → linux/sched.h` 可达。
- 接线面：`build-RMX2117.yml` 的 `Integrate Re:Kernel (enable_rekernel)` 步骤
  （默认 off）取本目录三件；该步骤在 `mirrors/q2-kernel-4.14` 的 worktree 上真实
  执行，`drivers/rekernel/` 四件落位、`drivers/Kconfig` 与 `drivers/Makefile`
  各注入一行、fragment 两行产物、binder 与 signal 上报钩子各一处。
- 编译面待 CI 构建；运行面（netlink unit 对接、上报路径）待刷机实测。
