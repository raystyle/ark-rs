# 2026-09-14 D46 Windows 构建切 gnu 交叉编译

## 决策与背景

用户裁定（2026-09-14）：Windows 构建切 gnu 交叉编译，摆脱 VC。ohmycloud 侧先行实证（不必重验）：WSL 加 rustup target x86_64-pc-windows-gnu 加 apt mingw-w64 后 cargo build 一次过（含 ring 汇编与 xz2 的 C）；产物部署 lan-win 实跑 --version 与 query（cloud manifest 拉取、minisign 验签、seq 校验）全过；CRT 静态零 DLL 依赖；尺寸 11.8MB 与 msvc 同量级。

## 落地

- build.yml：windows-latest msvc 岗整体换 ubuntu-latest 交叉岗（rust-toolchain targets 挂 x86_64-pc-windows-gnu、apt 装 mingw-w64）；资产名 ark-x86_64-pc-windows-gnu.exe；三岗统一带 --target 与 target 路径归位；交叉岗跳过测试与文档门禁（PE 不可在 linux 跑，测试面由 linux/mac 双岗覆盖）。
- selfupdate.rs：platform_triple windows 臂切 gnu；新增 asset_msvc_fallback（仅 Windows 有此层）；official_asset_meta 与 mirror_attempts/mirror_fallback_meta 签名扩 msvc 回退参数，读序三层（gnu 主名先、ark/ 段 msvc 回退名次、ome/ 段兼容名殿后）；fallback_seg 注释明确 msvc 名属 ark- 族走主段（逻辑天然覆盖）。
- 单测：镜像段读序尝试表三层顺序断言（gnu 先、msvc 次、ome 殿后）加缺层跳过；段内回落裁定补 msvc 名断言。
- mirror_fallback.rs 补收昨日漏网：ome/ 段删桶后锚链 404 的两个 gated 用例——「兼容段边车锚一致」整删、「双段同内容」对照断言退役（保留主段主名腿）。

## 语义边界

- 新 gnu 二进制对旧源（stable 段与历史 release 仅剩 msvc 资产的窗口期）：主名 miss 后 msvc 回退名命中即保供，下个 stable 封 gnu 版后回退不再触。
- 旧 msvc 二进制对新 gnu 源：读序无 gnu 名全 miss，无回退（有意），升级走 omc catalog 通道重装。
- 版本号不动、不推 tag（stable 封版另裁）；推 main 后 dev 滚动源自动出 gnu 资产。

## 验证与对线

- 本机交叉复验：cargo build --release --locked --target x86_64-pc-windows-gnu 一次过，PE32+ 12.4MB。
- cargo test --release --locked 全绿（137 单测加集成面）。
- 文档门禁四件套绿。
- 对线：herdr 发 ohmycloud 会话复核 diff，回执后推 main，盯 CI 绿与 dev release 资产名列表回报。
