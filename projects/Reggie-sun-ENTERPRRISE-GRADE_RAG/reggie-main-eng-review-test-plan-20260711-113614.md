# QA Test Plan

## Scope

本测试计划覆盖 `Production Technology Pilot Gate and Capability Cut` 的四个连续切片：

1. P1：stream finalized metadata parity。
2. P2：新增稳定 clarification reason `capability_disabled`。
3. P3：named overview capability fail-closed，且在 citation / no-citation 分叉前完成拦截。
4. P4：Production Technology 薄试点 gate 与 runtime acceptance。P4 在真实工程师 pre-plan observation、授权语料绑定和 expected business facts 完成前保持 blocked。

受影响入口：

- `POST /api/v1/chat/ask`
- `POST /api/v1/chat/ask/stream`
- `/portal`

## Critical Paths

### P1 Finalized Stream Metadata

- ordinary generation：最终 `meta` 与最终 response state 一致。
- deterministic route：最终 `meta` 不早于 route / clarification / citation state 定稿。
- retry / external fallback：最终 `meta` 反映最终模式、来源和 clarification，不泄漏中间态。
- fatal error：错误事件独立收口，不伪造成功态 finalized metadata。
- `clarification_request` 被稳定投影到 `ChatStreamMetaPayload`；正文 token、progress、reasoning 可继续提前流出。

### P2 Contract Extension

- backend enum 接受且只新增 `capability_disabled`。
- frontend type 与 backend schema 同步。
- `MAIN_CONTRACT_MATRIX.md` 登记稳定枚举语义。
- contract tests 冻结 sync / stream 的相同 reason 值；不得新增 response field。

### P3 Fail-Closed Capability Cut

- config key 缺失、读取异常、显式 false 均保持 capability disabled。
- config true 时保留现有 named overview 行为。
- 带 citations 的 sync / stream 请求返回 formal clarification，不调用 normal generation。
- 零 citations、无权限可见文档、空 corpus 的 sync / stream 请求同样被拦截，不可从 no-citation 分支绕过。
- clarification 必须满足：`mode=clarification`、`reason=capability_disabled`、有稳定 options、`citations=[]`、`external_sources=[]`、`related_assets=[]`。
- 现有两个“disabled 后继续 normal generation”的测试必须反转；它们是 CRITICAL regression tests。
- disabled turn 不得把旧 `document_id`、scope、citations、related assets 或回答正文写入 state / snapshot。
- 用户后续补充细节或 capability 重新启用时，正常路由可以恢复。
- 其他 deterministic route family 行为保持不变。

### P4 Pilot Gate And Runtime Acceptance

- same-document partial follow-up：验证 scope continuity 与具体业务事实，而非只看 citation 命中。
- stale-scope reset：切换到明确的新文档/主题后，不沿用旧 scope。
- ambiguity clarification：多候选且无法安全决定时返回 formal clarification。
- capability-disabled：验证 fail-closed 的 sync / stream parity。
- external fallback disabled：不得偷偷使用 external source。
- 四个本地测试账号分别覆盖账号、部门、权限与文档可见性边界。
- runtime identity：验证目标 API base、加载中的 revision/config checksum 与测试预期一致。
- `/portal`：验证 clarification 正文、选项、source cards、空 citations/related assets 和后续恢复交互。

P4 新 truth rows 与 pass 结论必须等待以下输入：

- 真实工程师 pre-plan observation record。
- 明确授权的 pilot corpus / document binding。
- 每个问题的 expected business facts 与 forbidden false-pass signals。

## Temporary Live Contract Probe

P4 的临时 Make wrapper 对稳定 API response 使用 `jq -e`（或等价只读断言），直接验证：

- `mode`
- `clarification_request.reason`
- `clarification_request.options`
- `citations` 数量
- `external_sources` 数量
- `related_assets` 数量
- sync / stream 最终状态一致性

不扩展 `scripts/eval_chat_memory.py` 字段语义，不新增 sample-governance schema。该 wrapper/probe 在正式 gate 被现有稳定 harness 吸收或试点结束后标记：

`[REMOVAL_CANDIDATE] Makefile::verify-production-tech-pilot + temporary live contract probe | 原因: 单一试点临时验证入口 | target owner: existing retrieval release harness | 当前真实 consumer: Production Technology pilot only | 删除或收敛触发条件: pilot exit 或稳定 harness 可表达同一合同 | last verification: implementation-time runtime acceptance`

## Edge Cases

- config 对象存在但属性缺失。
- config loader 抛异常或返回非预期对象。
- route 被选中但检索结果为空。
- citations 在 early stream 阶段存在、最终因权限或 rerank 被移除。
- stream 客户端中途断开；不得持久化未 finalized 的成功 metadata。
- clarification 后同 session 再次提问，旧 disabled 状态不得污染新 turn。
- 四账号中凭证缺失、登录失败或 runtime identity 不匹配时 fail fast，不降级成单账号 proof。
- provider 429 / timeout 与业务断言失败必须分开报告。

## Verification Layers

1. Focused unit tests：config、projection、top-level interception、state finalization。
2. Contract tests：backend schema、frontend type/build、sync/stream parity。
3. Existing behavioral gates：`verify-retrieval-focused`、`verify-chat-memory-behavioral-gate`、`verify-permission-visibility-gate`。
4. Fixed-ID eval rows：复用已存在的 same-doc / stale-reset / external-disabled evidence；新增 pilot rows 仅在 P4 blockers 清除后进入。
5. Active runtime replay：四账号、sync/stream、完整 visible answer body 审阅。
6. Browser smoke：`/portal` 登录态、clarification、来源区域与后续恢复。
7. Final release gate：只在前述 focused checks 通过且 runtime identity 已确认后运行，避免用全量 gate 掩盖局部失败。

## Exit Criteria

- P1-P3 focused tests、contract checks 和 active runtime replay 全部通过。
- 两个 CRITICAL fail-open regression tests 已反转并证明 generation 未被调用。
- citation / no-citation、sync / stream 四象限均 fail-closed。
- P4 blockers 有 machine-visible evidence；未满足时状态保持 blocked，不生成虚假 pass 或 truth。
- 四账号与 `/portal` proof 完整，visible answer body 符合 expected business facts。
- 临时 probe 的 owner、consumer 和删除条件已记录。
