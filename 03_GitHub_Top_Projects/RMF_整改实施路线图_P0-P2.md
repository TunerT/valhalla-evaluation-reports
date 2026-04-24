# RMF 整改实施路线图 P0-P2（2026-04-15 状态版）

## P0：已完成的可信边界首轮整改

### 1. Sidecar 证据接口分级与鉴权
- 状态：已完成首轮
- 目标文件：RMF/packages/framework-ai/src/sidecar-runtime-server.ts、RMF/packages/framework-ai/src/sidecar-runtime-config.ts、RMF/sla_rmf_sidecar/config/sidecar.runtime.ros2-live.example.json
- 已落实
  - `/discovery`、`/assembly`、`/witness` 支持 hidden/summary/full 分级。
  - sidecar runtime 支持 bearer-token、`protectHealth` 与访问控制摘要。
  - live ROS 样例和 smoke 已验证 401 拒绝与 summary 输出。
- 当前验收结果
  - 未带 token 访问受保护端点会被拒绝。
  - summary 视图已能返回受限 witness / assembly / discovery 摘要。
- 剩余缺口
  - 尚未接 mTLS、OIDC/JWT 或更细粒度的角色/审计体系。

### 2. Artifact 服务端签发化
- 状态：已完成首轮
- 目标文件：RMF/apps/vite-demo/server.ts、RMF/packages/framework-ai/src/signing.ts、RMF/packages/framework-ai/src/web-artifact-integrity.ts、RMF/schemas/web-artifact.schema.json
- 已落实
  - persisted artifact 升级为 `rmf.web.artifact.v0.3`。
  - 服务端对 canonical payload 计算 digest 并签发 HMAC signature。
  - `/api/rmf/negotiate` 与 `/api/rmf/template-export` 只接受验签通过的 artifact。
  - 当前 signer 已新增 `command-bridge` 后端，允许把签发动作外送给 KMS/HSM wrapper。
  - 已新增实际 `scripts/aliyun-kms-signing-bridge.py`，支持阿里云 AsymmetricSign/Verify 与证书私钥 Sign/Verify。
- 当前验收结果
  - tampered artifact 在 negotiate 与 template-export 上均返回 409。
  - negotiated artifact 会被当前 key 重新签发。
- 剩余缺口
  - 真实 wrapper 已落地，但生产环境尚未完成真实 key rollout 与审批链接入。

### 3. Artifact 离线 re-sign 与 key rotation
- 状态：已完成首轮
- 目标文件：RMF/packages/framework-ai/src/artifact-store.ts、RMF/packages/framework-ai/src/cli.ts、RMF/scripts/trust-boundary-tamper-smoke.mjs
- 已落实
  - 新增 `rotate-web-artifact-signatures` CLI，可对现有 `artifacts/*.json` 做显式离线重签。
  - 支持 `RMF_ARTIFACT_SIGNING_TRUSTED_KEYS` 做 trusted legacy key 平滑轮换。
  - 新 smoke 已覆盖 legacy key 接受、offline re-sign、current signer 使用 `command-bridge` 和 tamper rejection。
  - bridge contract 已升级到 `rmf.command-bridge.v0.2`，command-bridge 模式下优先走 verify，不再依赖“重新签一遍”的伪验签。
- 当前验收结果
  - `offlineRotatedKeyId` 已验证回切到 current key。
  - 历史 artifact 不再只能依赖运行时 negotiate/export 才完成重签。
- 剩余缺口
  - 仍缺 rotation batch 审批记录、签发批次元信息和更正式的 key provenance。

### 4. Community plugin 真实性校验
- 状态：已完成首轮
- 目标文件：RMF/packages/framework-ai/src/market.ts、RMF/packages/framework-ai/src/protocol.ts、RMF/schemas/community-plugin.schema.json、RMF/schemas/generated-community-plugins.schema.json
- 已落实
  - plugin schema/catalog 已补 `trustLevel`、`signature`、`revokedAt`、`revocationReason`。
  - catalog 生成前执行 signature 校验并拒绝 revoked plugin。
  - 新增 `rotate-community-plugin-signatures` 支持显式 key rotation。
  - current plugin signer 已支持 `command-bridge`，并可通过实际 KMS/HSM wrapper 运行。
- 当前验收结果
  - tampered plugin manifest 会被拒绝。
  - legacy plugin 可在 trusted keyring 下加载，并可切回 current key。
- 剩余缺口
  - catalog 自身尚未签名，也没有正式 issuer allowlist / approval workflow。

## P1：当前剩余的企业化与交付边界修复

### 5. Formal signing 升级到企业级密钥治理
- 目标文件：RMF/packages/framework-ai/src/signing.ts、RMF/packages/framework-ai/src/web-artifact-integrity.ts、RMF/packages/framework-ai/src/market.ts
- 任务
  - 将现有实际 wrapper 部署到正式 KMS/HSM 或等价托管方案。
  - 在 signature / rotation 流程中保留 key version、issuer provenance、batch audit 信息。
- 验收标准
  - release-readiness 能识别 key version 与 signer provenance。
  - 历史 artifact / plugin 可按 key version 做明确审计与回放。

### 6. Sidecar 证据读取缓存化
- 目标文件：RMF/packages/framework-ai/src/sidecar-runtime-evidence.ts、RMF/packages/framework-ai/src/sidecar-runtime-server.ts
- 任务
  - 将同步读盘改为 preload snapshot。
  - 使用文件变更监听或显式 refresh 机制。
  - 对 `/witness` 输出统一读缓存对象。
- 验收标准
  - 高频请求 `/witness` 时不重复同步读 4 个 JSON 文件。
  - 现有 smoke 结果保持一致。

### 7. 交付产物去绝对路径化
- 目标文件：RMF/packages/framework-ai/src/release-readiness.ts、RMF/scripts/sidecar-runtime-startup-benchmark.mjs
- 任务
  - 去除 `workspaceRoot`、`projectRoot`、`configFile`、`frameworkAiResolvedCliPath` 等绝对路径的对外交付暴露。
  - 新增 `portable` 或 `redacted` 输出模式。
- 验收标准
  - release-readiness 对外模式只包含 workspace-relative path。
  - benchmark delivery 版产物不暴露用户本机目录结构。

### 8. Plugin catalog 审批链治理
- 目标文件：RMF/packages/framework-ai/src/market.ts、RMF/packages/framework-ai/src/release-readiness.ts
- 任务
  - 为 generated catalog 自身增加签名。
  - 增加 issuer allowlist、approval profile、trustLevel 与 export policy 的绑定规则。
- 验收标准
  - catalog 级篡改会被独立拒绝。
  - `verified-external` / `signed-internal` 不再只是展示元数据，而是能影响 formal export policy。

## P2：互操作与扩展

### 9. OTel 语义桥接
- 目标文件：RMF/packages/rmf-core-python/rmf_core/sdk.py、RMF/packages/framework-ai/src/protocol.ts
- 任务
  - 设计 `taskId`、`episodeId`、`spanId` 到 OTel 的映射。
  - 增加 exporter 或桥接 JSON。
- 验收标准
  - RMF episode 能映射到标准 trace/span 结构。
  - Grafana/Tempo/Jaeger 可消费最小链路样例。

### 10. 跨栈 runtime 标准化
- 目标文件：RMF/packages/framework-ai、RMF/packages/rmf-core-python
- 任务
  - 提炼与 Node/Vite 无关的 runtime contract。
  - 为 Python/Go/Java 预留一致的 artifact/seal 接口。
- 验收标准
  - 至少两种语言 runtime 能生成同构 artifact。

## 建议执行顺序
1. 先把 artifact/plugin signing 的 `command-bridge` 从 mock/helper 接到真实企业级密钥治理。
2. 再做 sidecar 证据缓存和交付产物去绝对路径化。
3. 最后做 plugin catalog 审批链、OTel 和跨栈标准化。