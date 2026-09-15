# PLAN：当前目标规划指导

> **冻结声明（ADR-0001，2026-09-15）**：本文件为历史规划档案（命令语义全表现由 `README.md` 承接，AGENTS Commands 为摘要层）；只读保留，不再流转。

> 角色：**当前目标的规划指导**：当前这个目标怎么推进（步骤/标准/验收），随目标变化更新，不存历史目标。
> 与 `TODO.md` 分工：todo = 当前目标任务进度清单（做到哪）；本文件 = 当前目标怎么做（步骤/标准/流程）。

## 当前目标实施计划

> 当前目标：D44 下载链反转镜像优先（用户 2026-09-13 裁定「ark 安装默认走 ohmygh，
> 其他官方渠道是兜底」，令发 1.1.1）；顺领 go `goproxy` 语义键（omc go.mirror 数据先行）。
> D43 zig 去锁挂起待办（TODO 在案，数据面配合项已知会对岸）。

### 方案骨架

1. **download.rs 两条链反转**：主资产链与 evergreen 边车链改为镜像（env.ohmygh.com）**单次快速首试**（不退避不 curl：镜像未播该版本属常态，须秒级回落）后走官方完整链兜底（ureq 三次退避加 curl）；缓存三分支提取 `cache_reuse` 共用（渠道无关先查缓存）；镜像段失败形态=未命中 404 / 网络错 / 锚不符（CF 陈旧对象被锚拦下即换道）。
2. **锚语义不变**：expected_sha256（pin / 边车 / digest）照常校验；「有锚才回落」红线改写为「镜像为主、官方兜底、有锚必校验」；错误文案镜像在前官方在后。
3. **goproxy 语义键**：Mirror 增键，行级 upsert 落 GOENV 文件（win `%APPDATA%\go\env`、POSIX `~/.config/go/env` 即 `go env -w` 持久位，直写不依赖 go 二进制在位），配套 GOSUMDB=sum.golang.google.cn，用户键逐字保留；lint 值校验白名单与 fixtures 补样例。
4. **文档口径反转**：AGENTS 边界、README 镜像源节、SKILL 语义段与命令行、R015 消费侧、R016 mirror 键表；测试 mirror_fallback 用例改名「镜像优先」语义（行为断言兼容）。

### 自测面

1. 单测：镜像单次失败回落官方双断报错（镜像在前）主链与 evergreen 链各一；go env upsert 纯函数（用户键保留、配套 sumdb、幂等）与落盘内容比对；gated 真网（ARK_TEST_MIRROR）镜像优先命中加 sha 锚一致（原回落用例语义兼容改写）。
2. 真机：WSL `install omc`（pin 锚镜像直装已实证）与 `install bun`（镜像桶命中）走镜像优先链；缓存命中复用不触网回归。
3. 对线：右侧 codex review（实质改动：download 跨链反转），回执入 diary。

### 完成定义

- 全量测试绿加 clippy 干净加门禁四件套绿；v1.1.1 tag 推送滚 stable；WSL `self update --stable` 到 1.1.1 后 install 复验走镜像优先；对岸回执。
