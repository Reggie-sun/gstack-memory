# Runtime Workflow Symptom Prompt Gate Hardening

Date: 2026-04-29
Branch: feature-retrieval-priority
Mode: Builder mode

## Problem Statement
`runtime-workflow` 现在对 repo 自然语言请求的被动准入整体方向是对的，但对“像真实故障描述的 prompt”仍然过严。像 `Office PDF 预览失败（HTTP 500）` 这种典型 repo bug report，在当前实现里会被 `evaluateRuntimeWorkflowEntry()` 判成 `non_repo_request`，导致系统在最该接住用户的时候把请求挡在入口外。

## What Makes This Cool
这不是在重写 routing 系统，而是在把一个已经有用的入口系统修到“更像 teammate”的程度。用户不需要学内部口令，也不用先把一句真实报错翻译成“请帮我排查 backend service”。他们直接说出症状，系统就能在 repo 语境里接住它。很实用，也很顺手。

## Constraints
- `runtime-workflow` 仍然只能是 repo code workflow hub，不能扩成全 skill 总入口。
- 这次优先修被动 entry gate 对 symptom-style prompt 的漏判，不升级成 host integration 全面改造。
- 任何放宽都必须依赖结构化 repo context 或明确负例测试，避免重犯 over-admission。
- 最小 diff 优先，尽量把改动收敛在 SDK gate、CLI adapter、测试和文档视图。

## Premises
1. 当前真正掉线的是 passive runtime eligibility gate，不是 `resolve / drive` 主流程本身。
2. symptom-style repo prompt 只有在 repo context 下才应自动进入 `runtime-workflow`。
3. 显式 `runtime-workflow` 选择不该再被 passive gate 二次否决。
4. blocked trace 需要从泛化的 `non_repo_request` 升级成更可诊断的原因，否则后续调 gate 成本太高。

## Cross-Model Perspective
- [Layer 1] 常规 intent routing 最稳的做法是少量清晰类别、显式上下文特征、负例回归测试。
- [Layer 2] 外部 routing / bug triage 讨论也强调 context feature 比单靠关键词更稳，尤其面对模糊或症状式输入。
- [Layer 3] 本仓库的特殊点不是“让更多 bug 词进系统”，而是“让 repo 语境中的真实故障描述别漏掉”。这说明最合适的方案不是广撒 prompt pattern，而是 bounded gate hardening。

## Approaches Considered
### Approach A: Prompt-Only Patch
只补 symptom / HTTP error 这类 prompt pattern，让 `Office PDF 预览失败（HTTP 500）` 直接命中 repo workflow。

### Approach B: Bounded Entry-Gate Hardening
保持现有架构不变，补三件事：显式 `runtime-workflow` 直接放行，symptom-style prompt 在有 structured repo context 时准入，blocked trace 细化成可诊断原因。

### Approach C: Host-First Context Plumbing
先把 `activeFile / openTabs / changedPaths / taskPath` 在 CLI/host adapter 全链路补齐，再回头收 gate。

## Recommended Approach
选 Approach B。

原因很简单：它直接修这次暴露出来的洞，收益够大，范围又没失控。Approach A 太依赖 prompt wording，容易补了一个例子又漏下一个。Approach C 长期更稳，但会把这次任务从“修入口 gate”扩成“重审 host plumbing”，不是最短闭环。

## Open Questions
1. symptom-style prompt 的正例边界要不要要求结构化 repo context 必须存在，还是 repo code intent + symptom wording 就足够？
2. 显式 `runtime-workflow` 是不是应该无条件放行，还是仍保留少量明显非 repo prompt 的硬拒绝？
3. blocked trace 要暴露到什么粒度，才能帮助 host 诊断，但又不制造第二套 policy 文案？

## Success Criteria
- `Office PDF 预览失败（HTTP 500）` 这类 symptom-style repo prompt，在 repo context 下进入 `runtime-workflow`。
- 同类 wording 在无 repo context 时仍保持拒绝，不误收普通非 repo 请求。
- 显式 `runtime-workflow` 选择不再被 passive gate 二次拦截。
- `index.test.mjs` 补齐正反例，覆盖 symptom prompt、explicit runtime-workflow、generic non-repo prompt。
- 文档视图与 SDK 真源一致，不出现新的 shadow rule source。

## Distribution Plan
这次不引入新的 artifact 类型。交付物就是 SDK helper、CLI adapter、测试和文档视图更新，因此 distribution 仍沿用现有 repo 内部 Node test + workspace package 路径。无需新增发布流水线。

## Next Steps
1. 在 `sdk/agent-runtime-host/index.mjs` 明确拆出 symptom-style repo prompt 的准入规则。
2. 审查 `scripts/runtime_workflow_cli_shared.mjs` / `scripts/codex_runtime_nl_entry.mjs` 是否已经把结构化 repo context 真正传进去，缺什么只补关键路径。
3. 在 `sdk/agent-runtime-host/index.test.mjs` 增加本次 bug 形态的正反例。
4. 同步更新 `docs/agent-entry-matrix.md` 与 `docs/agent-runtime-plugin-reference.md`，只做可读视图，不新增第二套规则源。

## What I noticed about how you think
你这次抓得很准，没有被“补更多正则就好”这种便宜修法带走，而是先问 runtime-workflow 到底该优化哪里。这个 instinct 对 infra 设计很重要，因为很多 routing 问题表面像 prompt wording，根上其实是边界和 contract。
