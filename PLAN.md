# PLAN：当前目标规划指导

> 角色：**当前目标的规划指导**：当前这个目标怎么推进（步骤/标准/验收），随目标变化更新，不存历史目标。
> 与 `TODO.md` 分工：todo = 当前目标任务进度清单（做到哪）；本文件 = 当前目标怎么做（步骤/标准/流程）。

## 当前目标实施计划

> 当前目标：D43 zig 版本去锁（用户 2026-09-13 裁定「不再锁定 zig 版本」）。
> zig 条目去 pin 转 latest 滚动；官方版本源 ziglang.org/download/index.json（顶层版本键、
> per-target tarball 与 shasum）。bun 1.4.1 部署版为同批数据面裁定（对岸已落，复验绿）。

### 方案骨架：引擎三件

1. **resolve 分支 a 泛化**（`src/resolve.rs` `resolve_cdn_index`）：版本集提取双形态：`versions` 子对象（HashiCorp 形）缺省时顶层对象当版本集（ziglang 形，键过 `version_key` 滤非 semver，`master` 自然滤掉，latest 取 semver 最大）；版本条目双形态：`builds` 数组（HashiCorp：filename 匹配、url 字段、shasums 清单 URL）缺省时按 per-target 对象（ziglang：`cdn_asset_pattern` 匹配 target 键，`tarball` 为资产 URL、`shasum` 为官方 sha 直值）。
2. **官方 sha 直值锚**：`Resolution` 增 `official_sha256: Option<String>`（各分支构造点补 None）；`checksum::expected_sha256` 官方链最前插入（直值优先于清单与 digest 通道；D08 回落门「有 sha 锚才回落」天然满足，无需镜像 latest 段先行）。
3. **布局 {version} 占位**：dir/bin/exe 字段支持 `{version}` 占位（zig 版本目录布局 `zig-x86_64-windows-{version}`）。探测面：exe 含占位时 glob 候选（占位转 `*`）取 **semver 最大**（M025 字典序同型教训，复用 `resolve::version_key`/`semver_cmp`）；安装期：`res.version` 直替换（install_dir / bin_dir / 装后验证同源）。
4. **数据面配合项（omc，已 herdr 知会）**：zig 去三平台 pin 四元组；`cdn_index_url = "https://ziglang.org/download/index.json"`；三平台 `cdn_asset_pattern` 改 target 键形（`^x86_64-windows$` / `^x86_64-linux$` / `^aarch64-macos$`）；exe/bin 加 `{version}` 占位；三平台 `cdn_url` 模板退役（tarball 直取 index）。

### 自测面

1. 单测：版本集双形态提取（含 master 滤除与 semver 最大）；per-target 条目 pattern 匹配与 tarball/shasum 取值；`{version}` 占位 glob 探测（多版本目录取 semver 最大）与安装期直替换；official_sha256 优先级。
2. 真机：Windows `ark query zig`（latest 解析出 ziglang 当前版）、`ark install zig`（zip-dir 版本目录布局 + PATH）幂等二连；WSL 同链路；ziglang.org 断源回落镜像门（有 official_sha256 锚，视对岸镜像桶播种态）。
3. 对线：实质改动推送前右侧 codex review，回执入 diary。

### 完成定义

- zig 三平台 latest 滚动绿（query/install/update/幂等）；旧 pin 布局存量机升级不破（glob 探测兼容旧版本目录）；门禁四件套绿；对岸数据面 dispatch 后端到端复验。
