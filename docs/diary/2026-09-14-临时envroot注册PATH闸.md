# 2026-09-14 临时 envroot 注册 PATH 闸

## 报障与根因

ohmycloud D46 收官回执知会：lan-win 用户 PATH（注册表 `HKCU\Environment\Path`）沉淀 ark 探针遗留的临时 envroot 段（Temp 下 zig 与 jq 多条）。

根因链（G003 五步定位）：`install.rs` 的 `bin_dir` 非官方分支把 `env_root` 拼进注册路径（`join_if_relative(env_root, expand_install_path(bin))`），而 `platform::add_user_path` 无 temp 闸——临时 envroot（探针或测试装）跑 install 时 Temp 段直写用户持久 PATH。POSIX 同构风险（真实 profile 塞 Temp 段）。

## 修复

- `platform.rs`：`add_user_path` 加 temp 闸（`is_temp_path` 纯函数：canonicalize 双向尽力、失败退字面比较；命中告警并返回 Ok(false)）。一处闸全调用面：install 的 `register_bin`、rustup 接管、selfdeploy 部署位（部署位在用户目录本就不中闸）。闸的是注册面不是安装面：Temp 下装得（staging 本就常用 Temp），持久 PATH 不进。
- `tests/linux_install.rs`：sandbox 挪出系统 temp（`tempfile::tempdir_in("target")`，gitignore 内自动清理）——profile 注册语义的两用例（jq 闭环、幂等）注册 dir 为 sandbox home 下 `.local/bin`，原 tempdir 布局会误中闸；挪 `target/` 保受控行为的测试价值。

## 验证

- cargo test --release --locked 11 组全绿（139 单测加集成面；含新增 temp 路径判定单测：temp 子路径命中闸、真实泊位不误伤）。
- md 门禁四件套绿（四件齐跑）。
- 提交推送 main；lan-win 注册表遗留段由 ohmycloud 手工清（Temp 前缀段全可删），引擎侧已防再犯。

## 附记

知会二（hst self update 无镜像回退，api.github.com 匿名 403 断流）转 hst 仓处理，ark 侧三层读序（`mirror_attempts`）可作实现参考。
