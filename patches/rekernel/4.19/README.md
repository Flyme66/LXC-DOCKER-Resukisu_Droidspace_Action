# Re:Kernel 4.19（LOS kona/sm8250，alioth）

面向 `MOSSVENC/android_kernel_xiaomi_sm8250`（`lineage-23.2`）的内核内嵌形态
Re:Kernel；上游素材与 4.9/4.14 同源（`Sakion-Team/Re-Kernel` v11.6，快照
`localworkspace/mirrors/Re-Kernel` @`5adec48`）。

## 组成

| 补丁 | 内容 |
|---|---|
| `0001-drivers-rekernel-netlink-server.patch` | 新增 `drivers/rekernel/` 四件。与 4.9/4.14 的差异仅在 `rekernel.h`：本树具备 `JOBCTL_TRAP_FREEZE`（`include/linux/sched/jobctl.h`），故用上游原文直读该位，不带走 fallback |
| `0002-binder-frozen-transaction-notify.patch` | `drivers/android/binder.c`：include + `binder_transaction()` 内 target_proc 建立后的上报（reply / transaction / oneway-async 空间不足） |
| `0003-signal-frozen-kill-notify.patch` | `kernel/signal.c`：include + `do_send_sig_info()` 上报 |

## 锚点与形态

- binder.c 锚点为旧式引用计数 `target_proc->tmp_ref++;`，插入点取
  `binder_inner_proc_unlock()` 之后（该树此处无厂商块）。
- `include/linux/proc_fs.h` 无 `struct proc_ops` → `rekernel.c` 走
  `file_operations` 原文，无需 `patches.sh` 的那步 sed 转换。
- async 事务合并块同样省略：本树 `enum transaction_flags` 无 `TF_UPDATE_TXN`
  （`0x40` 亦未命名）。

## 状态

- 对 `71b13e6`（`lineage-23.2` tip）洁净树三件顺序 `git apply` 通过；引用符号
  均在树内：`struct binder_proc.tsk`、`alloc.free_async_space` /
  `alloc.buffer_size`、`cgroup_freezing`、`JOBCTL_TRAP_FREEZE`、`TF_ONE_WAY`。
- 接线面：`build-alioth.yml` 尚无 `enable_rekernel` 步骤——mix2s 先行，编译面
  待 4.9 的 CI 结果再辐射到本版本。
- 目标树来源：`localworkspace/kernels/partial/alioth-4.19/`（sparse，`lineage-23.2` tip）。
