# Valhalla 深度客观评测与整改报告：RMF (Runtime Meaning Fabric)
> 评估定位：企业级语义运行时与 Episode 证据底座

## 一、评估说明与证据来源

本版报告基于 RMF 仓库真实代码、样例产物和仓库自带校验链生成，不再沿用上一版中偏推断式结论。

### 1. 评估方法
- Matrix-Enterprise：检查 sidecar 启动链、runtime assembly、artifact/export 路径、I/O 模式、基准结果与回归 smoke。
- Valhalla-Pro：检查治理门禁、artifact 持久化可信边界、runtime 证据接口暴露面、community plugin 供应链真实性。

### 2. 本次实际核对的证据
- sidecar 与 runtime：RMF/sla_rmf_sidecar/README.md、RMF/packages/framework-ai/src/sidecar-runtime-server.ts、RMF/packages/framework-ai/src/sidecar-runtime-evidence.ts、RMF/packages/framework-ai/src/sidecar-runtime-assembly.ts
- 治理与门禁：RMF/packages/framework-ai/src/governance-profile.ts、RMF/apps/vite-demo/src/lib/governance-gate.ts、RMF/profiles/operator-governance.profile.json
- artifact 完整性与轮换：RMF/packages/framework-ai/src/signing.ts、RMF/packages/framework-ai/src/web-artifact-integrity.ts、RMF/packages/framework-ai/src/artifact-store.ts、RMF/apps/vite-demo/server.ts
- community plugin 真实性：RMF/packages/framework-ai/src/market.ts、RMF/packages/framework-ai/src/protocol.ts、RMF/schemas/community-plugin.schema.json、RMF/schemas/generated-community-plugins.schema.json、RMF/apps/vite-demo/community-plugins/default-human-review.json
- 回归与交付证据：RMF/scripts/trust-boundary-tamper-smoke.mjs、RMF/packages/framework-ai/src/verify-template-export.ts、RMF/open-source-kit/release-readiness/release-summary.json、RMF/artifacts/live-ros-sidecar-startup-benchmark-dry-run.json

### 3. 本次实际执行的仓库校验
- `npm run typecheck`：通过，`@rmf/framework-ai` 与 `@rmf/vite-demo` 类型检查均通过。
- `npm run validate-runtime-contracts`：通过，当前 manifest 为 1 route、1 plugin、12 artifacts，artifact 与 plugin 的真实性校验已纳入 contract validation。
- `npm run smoke:trust-boundary-tamper`：通过，验证了 legacy key 在未信任时被拒绝、trusted legacy key 生效、artifact 离线 re-sign 切回当前 key、artifact/plugin 当前 signer 可切到 `command-bridge` 后端、tampered negotiate/template-export 返回 409、plugin 重签生效、tampered plugin manifest 被拒绝。
- `npm run verify-template-export`：通过，template export 不仅校验 checksums，也会继续验证导出的 server-signed artifact。
- `npm run smoke:sidecar-runtime-live-ros-sample`：此前同轮整改已通过，证明 sidecar bearer-token、summary 视图和 witness 摘要路径可实际工作。

### 4. 评分区间说明
- 90-100：代码、产物、smoke 三者均支撑，且无明显上线阻断项。
- 75-89：主能力成熟，存在工程债务，但不阻断小范围落地。
- 60-74：能力成立，但存在企业化部署边界问题。
- 0-59：存在安全、可信或交付级硬阻断，不建议直接扩大接入。

---

## 二、校正说明

上一版有一条判断需要明确修正：

- 当前 RMF 的 community plugin 机制，本质上是“带 seal 元数据的 manifest/catalog”，不是“在宿主内执行第三方代码”的插件 VM。
- 因此，当前的首要问题不是插件代码执行沙箱，而是 plugin manifest 的真实性校验不足，以及服务端对 artifact 语义的可信边界偏弱。

---

## 三、Valhalla 双轨雷达图（证据版，2026-04-15更新）

```text
【🔵 Matrix-Enterprise】                  【🔴 Valhalla-Pro】
治理闭环完整度：■■■■■ 93.0               治理规则强度：■■■■■ 92.0
Sidecar 启动基线：■■■■■ 90.0             证据接口暴露控制：■■■■□ 84.0
运行时可移植性：■■■□□ 68.0               Artifact 可信边界：■■■■□ 86.0
高并发演进空间：■■■□□ 72.0               Plugin 供应链真实性：■■■■□ 84.0
【🟢 Adoption / Delivery】
交付证据体系：■■■■■ 94.0
跨栈扩展能力：■■■□□ 66.0
```

### 分值依据
- 治理闭环完整度 93.0：治理 profile、profile pack、runtime gate、release-readiness、template export 已形成闭环，并且样例 smoke、template export verification 与 tamper smoke 相互印证。
- Sidecar 启动基线 90.0：release-readiness 已记录 synthetic docker benchmark 推荐 budget 为 0，推荐 median startup 为 961.07ms，低于 3000ms 目标。
- 运行时可移植性 68.0：当前 benchmark/report 仍写入绝对路径，产物跨机器复用与对外交付不够干净。
- 高并发演进空间 72.0：当前 sidecar 与 assembly/evidence 读取大量依赖同步文件 I/O，当前样例规模可用，但对长期高频 runtime 请求不够友好。
- 证据接口暴露控制 84.0：sidecar 已支持 bearer-token、health 可选保护以及 discovery/assembly/witness 的 hidden/summary/full 分级，live ROS smoke 已覆盖 401 与 summary 返回；剩余缺口在企业级身份体系和更细粒度角色控制。
- Artifact 可信边界 88.0：artifact 已升级到 server-signed v0.3，`/api/rmf/negotiate` 和 `/api/rmf/template-export` 只接受验签通过的 persisted artifact，并新增离线 re-sign 工具、trusted key rotation 和 `command-bridge` KMS/HSM 抽象层；剩余缺口在真实 KMS/HSM 对接、密钥治理和不可抵赖链路。
- Plugin 供应链真实性 86.0：plugin manifest/catalog 已补齐 trustLevel、signature、revocation 字段，catalog 生成前执行真实性校验，并支持 key rotation 与 `command-bridge` 当前 signer；剩余缺口在 catalog 自身签名与审批链治理。
- 交付证据体系 94.0：open-source-kit、release-readiness、template export、checksums 与 exported artifact verification 已形成更完整的交付闭环。
- 跨栈扩展能力 66.0：已有 Python SDK 与 SLA adapter，但主 runtime/control plane 仍以 Node/Vite 为中心，OTel 等行业标准接口尚未接入。

---

## 四、已经验证成立的强项

### 1. 治理不是口号，已经进入代码执行门禁
- `validate-governance-profile-sample` 实测通过，说明治理 profile 不是展示文案，而是可执行验证对象。
- `evaluateGovernanceGate` 已对 confirm、exit、translation、non-participant review 做真实阻断判定。
- `smoke:sidecar-runtime-regressions` 与 `smoke:sidecar-runtime-live-ros-sample` 证明 sidecar discovery、adapter 依赖和 access policy 并非纸面设计。

### 2. RMF 的交付证据链明显强于常见 AI demo 工程
- release summary 已归档 governance profile、profile pack、latest artifact、template export、sidecar runtime smoke、synthetic evidence bundle。
- template export 会同时带出 artifact、export manifest、release-readiness snapshot 和 checksums，这对交付、审计、尽调都非常有利。

### 3. Artifact 已从“客户端送语义”升级到“服务端签发 + 离线可轮换”
- vite-demo sidecar 现在会把 persisted artifact 升级为 `rmf.web.artifact.v0.3`，服务端先 canonicalize 再签发。
- `/api/rmf/negotiate` 与 `/api/rmf/template-export` 只接受验签通过的 server-signed artifact。
- 新增 `rotate-web-artifact-signatures` 离线工具，可在 key rotation 后对历史 artifact 直接重签，不必等下一次 runtime 路径重新签发。
- `smoke:trust-boundary-tamper` 已证明 legacy key 可在 trusted keyring 下平滑接受，offline re-sign 后 artifact 会切回当前 key，篡改则被 409 拒绝。

### 4. Community plugin 已补真实性、信任等级与重签能力
- plugin manifest/catalog 已升级到带 `trustLevel` 与 `signature` 的 schema v0.2。
- catalog 生成前会执行 signature 验证，并拒绝 revoked plugin。
- 已新增 `rotate-community-plugin-signatures`，可以在 key rotation 时批量重签现有 manifest。
- `smoke:trust-boundary-tamper` 已证明 tampered plugin manifest 会被直接拒绝。

### 5. Sidecar 冷启动基线并不差
- 当前仓内已有 benchmark 文档和 summary/decision 链。
- 从 release summary 看，synthetic docker benchmark 推荐 budget 为 0，median startup 为 961.07ms，说明“Node 写 sidecar 一定启动很慢”这一判断，在当前样例规模下并不成立。

---

## 五、完成首轮整改后的剩余问题

### 问题 1：sidecar 证据接口已完成首轮硬化，但身份体系仍偏轻

#### 证据
- `RMF/packages/framework-ai/src/sidecar-runtime-server.ts` 已支持 bearer-token、`protectHealth` 和 discovery/assembly/witness 的 hidden/summary/full 分级。
- live ROS 样例配置已切到 bearer-token + summary，并且 `smoke:sidecar-runtime-live-ros-sample` 覆盖了未授权 401、summary 输出与 witness 摘要路径。
- 当前 access policy 仍以本地 env token 为主，没有接入 mTLS、OIDC/JWT 或更细粒度角色体系。

#### 风险判断
- 在本地 sandbox 与样例 smoke 下已可控。
- 但在企业部署场景里，单一 bearer token 仍不够，应继续提升到更明确的身份与审计模型。

#### 评级
- Valhalla-Pro：P1

#### 整改建议
1. 在 bearer-token 之上补 mTLS 或 OIDC/JWT 模式，而不是只停留在 env token。
2. 将 `summary` 进一步细分为 operator / auditor / debug 视图。
3. 对敏感 witness 字段补访问审计日志，而不是只做静态隐藏。

---

### 问题 2：artifact 可信边界已补强，并已进入 KMS/HSM 抽象层，但真实签名设施尚未接通

#### 证据
- `RMF/apps/vite-demo/server.ts` 现在会对 persisted artifact 做 canonicalize + HMAC 签发，并在 negotiate/template-export 前强制验签。
- `RMF/packages/framework-ai/src/artifact-store.ts` 新增了离线 `rotate-web-artifact-signatures` 工具，用于显式重签历史 artifact。
- `RMF/packages/framework-ai/src/signing.ts` 已把 `command-bridge` 升级为正式 `sign` / `verify` contract，并允许 envelope 记录 `keyVersionId` / `requestId`。
- `RMF/scripts/aliyun-kms-signing-bridge.py` 已作为实际 wrapper 落地，可直接接阿里云 `AsymmetricSign` / `AsymmetricVerify` 或证书私钥 `CertificatePrivateKeySign` / `CertificatePublicKeyVerify`。
- `smoke:trust-boundary-tamper` 已验证：artifact 当前 signer 可切到 `command-bridge`，legacy artifact 在 trusted keyring 下可被接受，offline re-sign 后会切回当前 key，而修改核心字段后会被 negotiate/template-export 拒绝。

#### 风险判断
- 当前已经不再是“客户端提交什么就持久化什么”的裸链路。
- 同时，签名实现也已不再被硬编码绑定在 env-key HMAC 上，当前已经具备可外接 KMS/HSM 的 `command-bridge` 抽象层，并已落地实际的阿里云 KMS/HSM wrapper。
- 当前剩余缺口不再是“没有真实 wrapper”，而是“生产环境尚未完成真实 key rollout、审批链和 release-readiness 的可观测性展示”。

#### 评级
- Valhalla-Pro：P1

#### 整改建议
1. 将现有 `scripts/aliyun-kms-signing-bridge.py` 部署到真实企业密钥治理环境，并完成 artifact current signer 的 production rollout。
2. 为 artifact 增加更明确的 formal / dev signing policy，而不是只靠当前 schema version 隐式区分。
3. 为离线 re-sign 流程补审批记录、批次信息和 rotation audit trail，并把 `keyVersionId` / `requestId` 纳入 release-readiness 展示。

---

### 问题 3：community plugin 已补真实性校验，并具备 KMS/HSM 抽象层，但 catalog 级治理仍未完成

#### 证据
- `RMF/packages/framework-ai/src/market.ts` 已在 catalog 生成前执行 manifest signature 校验，并拒绝 revoked plugin。
- plugin schema/catalog 已补 `trustLevel`、`signature`、`revokedAt`、`revocationReason`，同时支持显式 key rotation。
- community plugin current signer 也已支持切到 `command-bridge`，并可经 `scripts/aliyun-kms-signing-bridge.py` 走真实 KMS/HSM wrapper；`smoke:trust-boundary-tamper` 已证明 tampered manifest 会被拒绝，legacy plugin 在 trusted keyring 下可平滑过渡到 current key。

#### 风险判断
- 当前 plugin 不是代码执行 VM，这一点没有变化。
- 现阶段首要剩余问题已从“manifest 可伪造”下降为“catalog 自身未签名、issuer allowlist 与审批流仍偏本地化”。

#### 评级
- Valhalla-Pro：P1

#### 整改建议
1. 为 generated catalog 自身再补一层签名，而不是只校验单个 manifest。
2. 将 current plugin signer 的实际 wrapper 部署到正式 KMS/HSM，并建立 issuer allowlist / approval profile。
3. 将 trustLevel 与 release-readiness / export policy 进一步绑定，避免只做展示字段。

---

### 问题 4：sidecar 运行时路径存在较多同步文件 I/O，当前可用，但不利于长期放大

#### 证据
- `sidecar-runtime-evidence.ts` 使用 `readFileSync` 读取 witness、metrics、seal、manifest。
- `sidecar-runtime-assembly.ts`、`sidecar-topic-discovery.ts`、`governance-profile.ts` 等大量使用 `readFileSync` / `writeFileSync`。
- 当前样例 benchmark 说明启动基线不错，但这不等于长期高频请求下的 runtime path 就足够稳健。

#### 风险判断
- 对 bootstrap、验证、样例链问题不大。
- 对长期驻留 sidecar、频繁轮询 `/witness`、或未来引入更大规模 artifact 时，会放大尾延迟与阻塞问题。

#### 评级
- Matrix-Enterprise：P1

#### 整改建议
1. 将 sidecar runtime 的证据读取改为启动期 preload + 文件变更监听，而不是每次请求同步读盘。
2. 将 discovery、assembly、evidence 三类输出拆成 cache snapshot，对外只读缓存对象。
3. 后续如果要承接更高频事件流，再考虑把热路径从 Node 文件读写迁移到更强的 runtime 实现；但根据当前 benchmark，不建议现在就把“Rust 重写”作为 P0。

---

### 问题 5：交付产物仍携带绝对路径，不够便携，也可能泄露本地环境结构

#### 证据
- `RMF/artifacts/live-ros-sidecar-startup-benchmark-dry-run.json` 中仍包含绝对 `configFile`、`outputFile`、`frameworkAiResolvedCliPath`。
- `RMF/open-source-kit/release-readiness/release-summary.json` 顶层包含 `workspaceRoot` 与 `projectRoot` 绝对路径。

#### 风险判断
- 这会降低证据包的可重放性，也会把本机目录结构、用户工作路径泄露到交付材料中。

#### 评级
- Matrix-Enterprise / Valhalla-Pro：P1

#### 整改建议
1. 对外部交付模式默认只输出 workspace-relative path。
2. 增加 `portable` 或 `redacted` 输出模式，自动抹除 home 目录、用户名和本机绝对路径。
3. 将 benchmark report 分成 `debug` 与 `delivery` 两种产物格式。

---

### 问题 6：OTel 兼容层尚未形成正式出口

#### 证据
- 仓内已有 `spanId` 概念，Python SDK 也支持 `span_id`。
- 但未发现 OpenTelemetry / OTel / traceparent 的正式接入点或 exporter。

#### 风险判断
- RMF 目前已经有自己的 task/episode/span 语义模型。
- 如果不尽快与 OTel 对齐，企业现有可观测平台难以直接吸收 RMF 的运行时资产。

#### 评级
- Matrix-Enterprise：P2

#### 整改建议
1. 建立 `taskId -> traceId`、`episodeId -> root span attribute`、`spanId -> child span id` 的映射规范。
2. 输出 OTel-friendly exporter 或 JSON bridge，让 Grafana/Tempo/Jaeger 能消费 RMF 事件。

---

## 六、优先级重排后的整改路线

### P0：已完成的可信边界首轮整改
1. sidecar runtime 已补 bearer-token、health 可选保护和 hidden/summary/full 输出分级。
2. artifact 已升级为 server-signed v0.3，`/api/rmf/negotiate` 与 `/api/rmf/template-export` 已收紧为已验签输入。
3. community plugin 已补 trustLevel、signature、revocation-aware 验证，并新增 key rotation。
4. artifact 已新增显式离线 re-sign 工具，不再只依赖运行时新签发。

### P1：继续补企业化与交付边界
1. 把当前已存在的 `command-bridge` signer 接到真实 KMS/HSM 或等价的企业托管方案。
2. 为 plugin catalog 自身增加签名、issuer allowlist 与审批链。
3. sidecar runtime 引入 preload/cache，降低同步读盘比例。
4. release-readiness / benchmark 产物去绝对路径化。

### P2：做行业互通
1. OTel 适配。
2. 跨语言 runtime 进一步标准化。

---

## 七、结论

RMF 现在最值得肯定的，不是“概念大”，而是它已经把治理、artifact、release-readiness、template export、sidecar smoke 和这轮新增的 signing rotation/tamper smoke 真正写进了仓库。这一点比绝大多数 AI Runtime 或 Agent Demo 都更像产品底座。

本轮之后，RMF 最关键的三条可信边界已经不是空白项：
- sidecar 证据接口已经有 access policy 与视图分级。
- artifact 已经转入服务端签发、离线可重签、tamper 可拒绝的状态。
- plugin 元信息已经有真实性校验、trustLevel 和 rotation 路径。

接下来真正的主战场已经收敛到两件事：
- 把当前已落地的 `command-bridge` 抽象层真正接到企业级密钥治理与审批链。
- 把 runtime hot path 与对外交付格式做得更轻、更稳、更可移植。

因此，RMF 现在已经不该再被定义成“一个治理意识很强的开发沙箱”，而应被定义为“已经跨过首轮可信边界整改、但仍待企业化收口的语义运行时底座”。
