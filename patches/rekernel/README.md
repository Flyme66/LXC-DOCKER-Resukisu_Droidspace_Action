# Re:Kernel 内核内嵌形态补丁（4.9 / 4.14 / 4.19）

来源：Sakion-Team/Re-Kernel（GPL-2.0，`Integrate/` 素材）；本地工作区快照
`localworkspace/mirrors/Re-Kernel/`（v11.6）。≤5.4 非 GKI 走内核内嵌形态；
≥5.10 上游走 ko/Magisk 模块，不在本仓库。

## 组成（4.9/、4.14/、4.19/）

三版各三件；`0002`/`0003` 的锚点按各树上下文落位，`0001` 里
`rekernel.h` 的 `jobctl_frozen()` 形态按树内是否具备 `JOBCTL_TRAP_FREEZE`
选择（4.9/4.14 带走 fallback，4.19 用上游原文）；差异见各自 README。

### 4.9/

| 补丁 | 内容 |
|---|---|
| `0001-drivers-rekernel-netlink-server.patch` | 新增 `drivers/rekernel/`（`rekernel.c`/`rekernel.h`/`Kconfig`/`Makefile`）：netlink unit 22–26 探测、user port 100 上报；上报前统一过滤（目标非冻结组、与源同 uid 丢弃） |
| `0002-binder-frozen-transaction-notify.patch` | `drivers/android/binder.c`：`binder_transaction()` 内 target_proc 建立后按 reply 分派 reply/transaction 上报，oneway 且 async 空间不足再补 overflow 上报 |
| `0003-signal-frozen-kill-notify.patch` | `kernel/signal.c`：`do_send_sig_info()` 对 SIGKILL/SIGTERM/SIGABRT/SIGQUIT 上报 |

`drivers/Kconfig` 与 `drivers/Makefile` 的 `source`/`obj` 注入由 workflow 步骤
幂等完成：这两个文件会被 root 集成步骤改写，补丁 context 会漂移。

## 与上游形态的差异

- `rekernel.h`：上游直接读 `JOBCTL_TRAP_FREEZE`；pre-freezer-v2 树（4.9/4.14）
  无此位，改为 `#ifdef` 回退，冻结判定由 `frozen_task_group()` 的
  `cgroup_freezing()` 承担（4.19 有该位，可用上游原文）。
- `binder.c`：上游另插入 `TF_UPDATE_TXN`（`0x40`）的 async 事务合并块；三棵目标树
  都未命名该位，其中 MTK 4.14 的 `0x40` 是厂商 `TF_ASYNC_BOOST`，照搬本地定义会
  撞位，故本 port 只保留上报块。判定与证据见
  `localworkspace/pipelines/rekernel/FEASIBILITY.md`。

## 接线（workflow）

三版各接一条 `enable_rekernel`（默认 off）步骤：polaris 取 4.9、RMX2117 取 4.14、
alioth 取 4.19。步骤 apply 三件补丁 + `drivers/Kconfig`、`drivers/Makefile` 注入 +
`rekernel.config.fragment`（`CONFIG_REKERNEL=y`、`# CONFIG_REKERNEL_NETWORK is not set`）；
`merge-defconfig.sh` 断言 `CONFIG_REKERNEL` 且 `CONFIG_REKERNEL_NETWORK` 未开。
AK3 显示面在 `ENABLE_REKERNEL=true` 时于特性行加入 `REKERNEL`。

## 状态

- 4.9（polaris）：三件补丁对 `lineage-22.2` tip 洁净树真实 `git apply` 顺序通过，
  已接入 `build-polaris.yml`。
- 4.14（RMX2117）：三件补丁对 `f0c2afc4d` 洁净树真实 `git apply` 顺序通过，
  已接入 `build-RMX2117.yml`。
- 4.19（alioth）：三件补丁对 `71b13e6`（`lineage-23.2` tip）洁净树真实 `git apply`
  顺序通过，已接入 `build-alioth.yml`。
- 三版的接线步骤均在其目标树 worktree 上真实执行：`drivers/rekernel/` 四件落位、
  `drivers/Kconfig` 与 `drivers/Makefile` 各注入一行、fragment 为
  `CONFIG_REKERNEL=y` + `# CONFIG_REKERNEL_NETWORK is not set`、binder 与 signal
  上报钩子各一处。编译面待 CI 构建，运行面（netlink unit 对接、上报路径）待刷机实测。
