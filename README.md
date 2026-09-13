# ark

**Ark（Agent Runtime Kit，命令 `ark`）**：本机跨平台（Windows / Linux / macOS）环境部署管理 CLI。管 47 个工具与 agent 运行时的版本解析、下载校验、PATH 注册、pin 锁定、更新与 doctor 诊断。

## 安装

需要 Rust 工具链（[rustup](https://rustup.rs)）。三平台同一套流程：克隆、构建、自部署。

Windows（PowerShell 7）：

```powershell
git clone https://github.com/raystyle/ark_rs
cd ark_rs
cargo build --release
.\target\release\ark init
```

Linux / WSL / macOS：

```bash
git clone https://github.com/raystyle/ark_rs
cd ark_rs
cargo build --release
./target/release/ark init
```

`ark init` 自部署：二进制进用户程序目录（Windows `%LOCALAPPDATA%\Programs\ark`，POSIX `~/.local/bin`）、同步 catalog、注册 PATH，幂等可重跑。重开终端后 `ark doctor` 验证。

- 被管理工具装在 EnvRoot：Windows 默认 `D:\ohmyenv`（无 D: 盘则 `C:\ohmyenv`），Linux / macOS 默认 `~/.local/share/ohmyenv`；可用 `--env-root` 或 `ARK_ROOT` 改（读回旧名 `OHMYENV_ROOT`）
- ark 自身装用户目录，与 EnvRoot 解耦；旧 `ome` 部署位与旧环境变量读回兼容，`ome` 以别名过渡可用
- 升级自身：`ark self update`（dev / stable / git 三通道）

## install 链

检测驱动，一条链走完：

1. `ark doctor` 体检：系统 / 依赖两层诊断加 check 节，列缺口与修复建议
2. `ark install [名]` 幂等安装：版本解析、下载、sha 校验、解压、PATH 注册与注册表 / 配置写入一次完成；已装且版本一致即跳过；省略名则全量
3. `ark verify` 部署域验收：逐维度 PASS/FAIL，FAIL 退出码非零，可进脚本
4. `ark heal` 自愈：PATH 修复、镜像源补写等，幂等；`--dry-run` 预览

配套查询与更新：

```powershell
ark status                # 锁定 / 已装 / PATH 三态对照
ark query ffmpeg --latest # 只解析最新版与资产，不下载
ark update [名]           # 拉云端最新安装（锁定归云端数据面，不回写）
ark pin [名]              # 查看 / 设置版本锁定（lock 为别名）
```

## catalog 四重门

清单数据权威在云端（ohmycloud catalog-seed 与镜像三件套），本仓持格式契约与消费逻辑，新增软件不用换二进制。拉取链过四重门，任一不过即拒收：

1. 边车 sha 锚：清单 sha256 与边车逐字对上
2. schema 解析：`Catalog::load` 解析不过不收
3. minisign 验签：公钥内嵌二进制，签名不过不收
4. seq 单调门：顶层 seq 低于已见拒收（防回滚重放旧但签名有效的清单对）

```powershell
ark catalog        # 看 catalog 与 manifest 两面：在位、锚、年龄、签名、同步态
ark catalog sync   # 立即从云端刷新（默认 TTL 24h 自动刷新；ARK_CATALOG_TTL 改，ARK_OFFLINE=1 关，旧名 OME_* 读回）
```

## 供给清单

50 个工具（清单权威在云端 seed 与镜像三件套），agent 四家二进制 PATH 在位即跳过、存量原地纳管：

| 类 | 工具 |
| --- | --- |
| 智能体（4） | claude、codex、grok、kimi |
| 自管与过渡（2） | ark（自管主条目，原 ome）、ome（更名过渡条目，存量端水位清零后退役） |
| 操作编排（1） | herdr（多 agent 并行会话宿主） |
| agent 配置与诊断（1） | hst（原 oma；Hooks、Statusline、Trace 与只读观测） |
| 云端控制台（1） | omc（云与内网控制台 CLI，npm-tgz 通道） |
| 运行时（7） | pwsh、wsl、docker、dotnet、bun、python、nushell |
| 运行时管理器（2） | fnm（node）、uv（python） |
| 编译器（4） | vsbuild（含 C 编译器）、rust、go、zig |
| 多路复用（1） | rmux |
| 远程服务（1） | openssh |
| 密钥安全（3） | age、sops、gitleaks |
| 命令工具（22） | git、gh、aria2、7z、gsudo、oscdimg、rg、jq、mq、yq、starship、just、ast-grep、rumdl、shellcheck、zoxide、sheldon、ffmpeg、rclone、reader、lightpanda、typst |
| 运行时衍生（1） | browser-harness（bin 名 bh） |

平台空态如实表达：sheldon 上游无 Windows 资产、shellcheck 仅 Linux 入册、ffmpeg 官方 mac 构建仅 Intel、lightpanda 上游无 Windows 构建。

## 镜像源

下载官方渠道失败自动回落 env.ohmygh.com 自建镜像；仅当有 sha 锚（catalog pin 或官方清单）才回落，校验不放松。

## 输出契约

数据走 stdout、提示走 stderr、错误为单行 JSON；`--format kv|json|jsonl` 可选。字段与退出码全量契约见 `docs\references\R013-Agent友好IO契约-输出格式退出码与冻结面.md`。
