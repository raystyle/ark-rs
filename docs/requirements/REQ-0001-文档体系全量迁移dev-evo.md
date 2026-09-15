---
id: REQ-0001
title: 文档体系全量迁移 dev-evo
status: draft
priority: must
trace: check.py 全 PASS 加四件套绿加断链零
---

# REQ-0001:文档体系全量迁移 dev-evo

## Scenario

本仓治理体系（根原语四件加 INDEX 加 docs 六目录）需统一到用户沉淀的 dev-evo 文档即代码体系，迁移中不丢任何在役规则语义、不断任何历史决策可查性。

## Criteria

- [ ] AGENTS.md 五节合同齐备（PE-01 PASS），现行四段规则语义无损重排进五节
- [ ] docs/adr 与 docs/requirements 骨架与索引在位（PE-02、PE-03 PASS）
- [ ] PRD D 表冻结头注记落位，D50 起走 ADR（ADR-0001 自证）
- [ ] TODO 与 PLAN 进行中面转 REQ，GOAL 定位句并入 AGENTS
- [ ] INDEX 职责拆解（AGENTS Read first 加各 README 索引），llms.txt 建成
- [ ] M 系列并入 ADR，全仓引用替换，断链回归零
- [ ] PE-11 禁字存量清零（Unicode 箭头、连接号），PE-12 断链清零
- [ ] check.py 全 PASS（requirements 空以外无 SKIP 障碍）加本仓四件套绿
