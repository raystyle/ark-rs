# PLAN：当前目标规划指导

> 角色：**当前目标的规划指导**：当前这个目标怎么推进（步骤/标准/验收），随目标变化更新，不存历史目标。
> 与 `TODO.md` 分工：todo = 当前目标任务进度清单（做到哪）；本文件 = 当前目标怎么做（步骤/标准/流程）。

## 当前目标实施计划

> 当前目标：D42 运行时源中国镜像统一落 manifest（用户 2026-09-13 重投裁定 + 补充裁定）。
> 口径：源配置收进统一 manifest 部署配置（manifest DSL 扩 mirror 节，post_install 按节落源），
> 不散在各端脚本；全集对齐 ohmypwsh 遗产 set-mirror.ps1 / P0017 / P0019；
> omc bootstrap 已按同口径兜底，ark 到位后撤。验收面 wsl 总台 verify A13 四件断言。

### 裁定口径

| 面 | 定夺 |
| --- | --- |
| fnm / node 面 | `FNM_NODE_DIST_MIRROR=https://npmmirror.com/mirrors/node/`（win 用户环境变量、POSIX shell rc）；npm registry=npmmirror（`~/.npmrc` 行级 upsert） |
| uv / python 面 | `~/.config/uv/uv.toml` 清华 TUNA `[[index]]` default；pip.conf 同 TUNA；`UV_PYTHON_INSTALL_MIRROR` NJU（env 面） |
| bun 面 | `~/.bunfig.toml` registry=npmmirror（与 heal_bunfig 同语义） |
| rust 面 | `RUSTUP_DIST_SERVER`/`RUSTUP_UPDATE_ROOT`=rsproxy；装走 rsproxy rustup-init；cargo config rsproxy 全量形态（win=EnvRoot 重定位位、POSIX=`~/.cargo`）；rust 接管扩 POSIX（装法 + 持久 env + config + PATH） |
| 权威与形态 | 镜像值一律 manifest 数据面声明（mirror 节）；落点与合并语义引擎按类型实现一次；rust 引导期常量与通道耦合留在 rustup 模块（收口过渡同 2026-09-11 撤内建模式，omc 数据就绪后再撤） |
| 幂等与存量 | 各键内容一致零重写；install 幂等分支与 update 同样落源（共识③同款），存量端升级即得 |
| 测试隔离 | mirror 落源整面受 `ARK_TEST_NO_PATH_REG` 闸门（用户环境写入面第五面） |

### 方案骨架

1. **manifest.rs（DSL 扩展）**：`ToolManifest` 增 `mirror` 节（键：`env`/`env_unset`/`npm_registry`/`bunfig_registry`/`uv_index`/`pip_index`/`cargo_config`，全可选）；schema_version 不动（serde 容忍未知字段，旧引擎前向兼容）；`apply_mirror` 按 kind 落源（env 走 set_user_env_var 双通道、npmrc 行级 upsert 保认证行、bunfig/uv/pip/cargo 内容比对整写）；env 先行（post_install 子进程继承，FNM_NODE_DIST_MIRROR 即时生效）。
2. **platform.rs**：增 `remove_user_env_var`（win 注册表删值、POSIX env 块摘行；块空收口）；对应纯函数入测。
3. **install.rs**：`apply_manifest_primitives` 接 `apply_mirror`（env_set 后、post_install 前；幂等分支与早退通道共用同点）。
4. **rustup.rs（POSIX 接管）**：POSIX 分支落码：rsproxy rustup-init 下载（evergreen 边车锚、chmod +x）引导 stable；系统标准位 `~/.rustup`/`~/.cargo`（R010，不重定位）；持久 `RUSTUP_DIST_SERVER`/`RUSTUP_UPDATE_ROOT`；`~/.cargo/config.toml` rsproxy 全量；PATH `~/.cargo/bin`；Windows 行为零变化。
5. **verify / heal / lint**：verify 增 POSIX `dev-rust` 维度（`~/.cargo/bin/rustc` 加 `~/.rustup/toolchains`）；heal `dev-rust` 键开 POSIX；catalog_lint manifest 面增 mirror 节校验（全空应省略）；夹具 manifest.toml 增 mirror 形态样例。
6. **文档与回执**：R016 增 mirror 节规范；README 行为基线与 CHANGELOG；INDEX 同步；diary 当天篇；回执含 manifest 节形态说明与 omc 侧待办（manifest mirror 节数据、tools.toml rust POSIX 字段、镜像桶 rustup-init POSIX 资产、bootstrap 撤点）。

### 自测面

1. 单测：mirror 解析全键形态；npmrc upsert 纯函数（无文件新增行、异值行原位替换、认证行不动、带空白键形）；bunfig/uv/pip/cargo 内容比对幂等（注入 home/config dir）；env 块摘行纯函数（POSIX）；rustup POSIX 参数与位路径；lint mirror 全空报错。
2. 集成：install 沙盒（闸门开）mirror 跳过行；`cargo test` 全量加 clippy 干净（Windows 本机）；POSIX 面单测 cfg 门控下沉（M016 纪律），WSL 真机面留验收。
3. 真机与总台：Windows 本机 `ark install uv`/`fnm` 幂等二连零重写；WSL `ark install fnm uv rust` 后 A13 四件断言绿（omc 数据就绪后跑）；对线（右侧 codex）结论与修复回执入 diary。

### 完成定义

- mirror 节 DSL 落码加单测绿；rust POSIX 接管落码（Windows 零回归）；R016/README/CHANGELOG/INDEX/diary 齐；codex 对线过；提交号与版本回执。
- omc 侧待办随回执知会（herdr 通道）：manifest mirror 节数据（fnm/uv/bun）、tools.toml rust linux/mac 字段、镜像桶 `rustup-init` POSIX 资产名、bootstrap 兜底撤点。
