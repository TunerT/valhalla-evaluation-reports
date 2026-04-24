> [!NOTE]
> **🌍 Bilingual Notice:** This is the Chinese original report. The English version is currently being translated. We welcome community PRs!

# GitHub 外部项目双轨详细评测（ClawPanel 独立版本）

## 项目识别

- 说明：本稿已保留为 qingchencloud/clawpanel 的独立评测版本，用于归档首轮候选仓库分析结果。
- 原始触发背景：首轮根据“Hermes claw”语义匹配，默认命中 qingchencloud/clawpanel。
- 当前定位：它不再作为 NousResearch/hermes-agent 的最终正稿，仅作为 clawpanel 独立版本继续保留。
- 评测时间：2026-04-16

## 评测依据

- GitHub API 元数据
- 本地源码快照：GitHub上/clawpanel-eval/clawpanel-main
- Project-Valhalla-Pro 自动评测输出：Project-Valhalla-Pro/reports/qingchencloud-clawpanel-pro-auto.json 与 Project-Valhalla-Pro/reports/qingchencloud-clawpanel-pro-auto.md
- 基于 Valhalla-Matrix-Enterprise 方法论的人工架构/产品化评审

## 一、项目概览

| 项目项 | 结果 |
| --- | --- |
| 仓库 | qingchencloud/clawpanel |
| 描述 | OpenClaw & Hermes Agent 多引擎 AI 管理面板，内置 AI 助手，多模态与工具调用 |
| Star | 2370 |
| Fork | 304 |
| Open Issues | 119 |
| 默认分支 | main |
| 最近推送 | 2026-04-13T07:27:10Z |
| 主要语言 | JavaScript 1,854,309 / Rust 766,323 / CSS 180,729 |
| 许可证 | GitHub API 显示 NOASSERTION，但仓库 LICENSE、package.json、src-tauri/Cargo.toml 均声明 AGPL-3.0 |

### 核心定位

从 README 与代码结构看，ClawPanel 不是单点聊天客户端，而是一个桌面优先、兼容 Web 模式的本地 AI Agent 控制台。它把 OpenClaw、Hermes Agent、插件管理、技能管理、渠道配置、日志、定时任务、记忆和安装升级整合进一个跨平台壳层里。

这类产品的价值不在“模型能力”，而在“统一控制面”。它更像一个面向个人开发者、小团队运维、灰度实验环境的 AI Agent 运维工作台。

## 二、执行摘要

### 双轨结论

| 轨道 | 结论 |
| --- | --- |
| Valhalla-Matrix-Enterprise | 产品完成度高，交付和部署灵活，属于“强产品化、多入口、多引擎”的高可用控制台 |
| Project-Valhalla-Pro | 自动评测结果为 REJECTED，在零层免疫门即被拒绝，未进入后续加权估值阶段 |

### 总判断

ClawPanel 的优点非常明确：产品面完整、跨平台交付成熟、对 OpenClaw/Hermes 用户上手门槛低、Web/桌面双态兼容、CI 和热更新链路都已经成形。

但从企业安全和治理角度看，这个项目的默认权限面偏宽，主机控制能力偏强，且在应用源码中直接保留了 pipe-to-shell 安装路径，导致 Project-Valhalla-Pro 自动准入被直接拒绝。换句话说，它是一个“能力强、运维友好，但默认安全边界不够收紧”的产品。

最终建议：

- 适合个人开发者、内测团队、隔离实验环境、熟悉本地运维的操作者。
- 不适合未经加固就直接作为企业标准桌面终端或受监管环境的默认控制台。

## 三、Valhalla-Matrix-Enterprise 评测

### 1. 架构定位

ClawPanel 的产品方向相当清晰：

- 以 Tauri v2 作为桌面壳，前端为 Vite + 原生 JavaScript，后端为 Rust。
- 不是单一引擎，而是同时支持 OpenClaw 与 Hermes Agent 双引擎。
- 同时提供桌面模式与纯 Web 模式，兼容 ARM64 开发板、Linux 服务器和 Docker。

这说明它在 Matrix 视角下不是“算法项目”也不是“单一 SDK”，而是一个偏运营控制台、偏交付层的复合型资产。

### 2. 产品完成度

从 README 和代码组织看，它已经覆盖较完整的控制面能力：

- 双引擎切换与管理
- 内置 AI 助手
- Hermes 聊天与工具调用可视化
- 服务管理、日志、配置、记忆、路由图、定时任务
- 插件中心、渠道配置、技能安装与依赖安装
- 桌面版、Web 版、Docker、ARM64 场景

这类完整度在开源“AI 面板”项目里不低，明显不是 demo 或单页 UI 包装。

### 3. 交付与运维成熟度

工程侧的正面信号比较多：

- CI 在 macOS、Linux、Windows 三平台跑 npm ci、cargo fmt、cargo check、cargo clippy 和前端构建验证。
- Release 工作流覆盖多平台安装包，并额外处理 Windows 完整包与前端热更新包。
- 前端热更新链路带 SHA-256 校验和最低兼容版本判断，不是裸下载替换。
- 同时存在 package-lock.json 和 src-tauri/Cargo.lock，说明 JS/Rust 两侧都有锁文件。
- 提供 SECURITY.md，并给出私下漏洞披露路径和响应时间。

这些特征让它在 Matrix 视角下具备“可持续交付”的基本盘。

### 4. 部署灵活性

它的部署灵活性比一般 Tauri 工具更强：

- 桌面原生打包
- Linux 服务器 Web 模式
- Docker 模式
- ARM64 开发板模式

这使它很适合社区传播和异构环境落地，也解释了为什么项目增长快、覆盖面广。

### 5. Matrix 侧主要短板

Matrix 视角下，它的问题不是“太弱”，而是“太宽”：

- 产品边界宽，功能面从安装器、CLI 代理、插件系统、技能系统、渠道连接一路铺到运行时控制，复杂度不低。
- 桌面版与 Web 版并存，带来更多环境差异和运维分叉。
- 强绑定 OpenClaw/Hermes 生态，通用性不如中性控制平面。
- README 已明确提醒 Web 版默认监听所有网卡且需注意公网暴露，这说明运维门槛不是零。

### 6. Matrix 人工量化结论

以下为基于 Valhalla-Matrix-Enterprise 方法论的人工量化，不是自动 CLI 输出：

| 维度 | 说明 | 分数 |
| --- | --- | ---: |
| 架构设计 | Tauri + Rust + JS 双栈整合合理，但系统边界较宽 | 88 |
| 产品完成度 | 双引擎、插件、技能、日志、配置、部署模式都很完整 | 92 |
| 交付成熟度 | 多平台 CI、Release、热更新链路齐全 | 89 |
| 部署灵活性 | 桌面/Web/Docker/ARM64 覆盖面强 | 91 |
| 稳定与可维护性 | 有工程纪律，但复杂度上升明显 | 82 |
| 企业适配度 | 原生安全边界偏宽，默认不宜直接企业准入 | 76 |
| 综合 Matrix 分 | 强产品化、多引擎控制台 | 86 |

### 7. Matrix 结论

如果仅从产品价值、交付完整度和生态穿透力看，ClawPanel 在同类开源项目里是偏强的。它最适合作为“Agent 运维控制台”而不是“企业安全底座”。

## 四、Project-Valhalla-Pro 评测

### 1. 自动评测结果

本次已实际运行 Project-Valhalla-Pro，结果如下：

| 项目 | 结果 |
| --- | --- |
| 评测模式 | pro |
| 自动状态 | REJECTED |
| 拒绝原因 | Zero-layer immune rejection was triggered |
| 扫描文件数 | 117 |
| reject 级命中 | 2 |
| review-only 命中 | 3 |

也就是说，这个项目并不是“低分通过”，而是压根没有进入后续供应链估值合成阶段，直接被零层免疫门拦截。

### 2. 触发 REJECTED 的直接原因

Pro 自动报告给出的两个硬拒绝点都不是外围 shell 脚本，而是应用源码内部保留了 pipe-to-shell 路径：

1. src/lib/openclaw-kb.js

- 内置知识库直接包含 `curl -fsSL https://openclaw.ai/install.sh | bash`。
- 由于它进入了应用内部知识/提示源，而不是单独的外部运维文档，Pro 将其判为 application_source 级别的 pipe-to-shell 命中。

1. src/lib/mirror-urls.js

- deployCommand() 直接返回 GitHub 与 Gitee 两条 `deploy.sh | bash` 命令字符串。
- 这同样属于应用源码中的可执行安装路径表达，而不是离线文档说明。

这两个点叠加，足以让 Pro 在“零层免疫”里直接拒绝。

### 3. 被降级为 review-only 的命中

另外 3 个命中被认定为上下文可解释，因此只降级为人工复核：

- deploy.sh
- scripts/docker-deploy.sh
- src/pages/setup.js

这说明 Project-Valhalla-Pro 并不是机械地把所有 curl|bash 都打死，而是确实区分了运维脚本上下文和应用源码上下文。

### 4. 需要肯定的安全控制

虽然自动准入被拒，但项目并不是完全没有安全意识，几个正面点仍然成立：

- assistant 页面内置关键危险命令模式识别，明确把 curl|bash、wget|bash、rm -rf、git push --force 等列为 critical patterns。
- 前端热更新使用 SHA-256 校验，并检查 minAppVersion。
- OpenClaw 升级链路也有 SHA-256 校验逻辑。
- README 明确提示 Web 版公网暴露风险，并建议保护 ~/.openclaw 目录权限。
- SECURITY.md 完整给出了漏洞私报方式和响应时限。

所以更准确的表述是：项目有安全防护意识，但默认能力面仍然超过了企业直接准入的可接受范围。

### 5. 手工 Pro 风险拆解

#### 5.1 主机控制面过宽

ClawPanel 的本地 AI 助手并非只读助手，而是带宿主机操作能力：

- assistant_exec 直接以 `sh -c` 或 `cmd /c` 执行命令。
- assistant_read_file 直接按用户给定路径读取文件。
- assistant_write_file 直接按用户给定路径写文件，并自动创建父目录。
- assistant_list_dir 也没有路径沙箱或 allowlist。

这里只有审计日志，没有目录白名单、工作区边界、只读模式或强制逐次授权。对于个人工具可以接受，但对企业桌面管控来说太宽。

#### 5.2 安装与扩展面较大

项目支持多种安装/扩展路径：

- install_plugin / install_channel_plugin 通过 OpenClaw CLI 安装插件。
- skills_install_dep 可执行 brew、npm -g、go install、uv tool install。
- skillhub_install 会从远程 SkillHub 下载 zip 并解压安装。
- install_clawapp 直接通过官方 `install.sh | bash` 安装。

这使产品非常易用，但也把软件供应链、远程脚本、第三方镜像和本机环境修改纳入默认能力面。

#### 5.3 桌面权限与持久化能力偏强

Tauri 配置与 capability 暴露了较宽的桌面权限：

- src-tauri/tauri.conf.json 中 `csp` 为 null。
- default capability 允许 `shell:allow-open`。
- default capability 允许 autostart enable/disable/is-enabled。

这不等于项目一定危险，但意味着如果再叠加本地命令执行、插件安装和远程资源拉取，整体攻击面会明显放大。

#### 5.4 默认鉴权姿态不够保守

Hermes 配置逻辑中，ClawPanel 在写入 `.env` 时默认加入：

- `GATEWAY_ALLOW_ALL_USERS=true`
- `API_SERVER_KEY=clawpanel-local`

随后 hermes_api_proxy 会读取 API_SERVER_KEY 并自动注入 Authorization 头。这种设计确实降低了上手门槛，但默认值太固定、也太乐观，不符合企业“默认拒绝、按需放开”的准入习惯。

#### 5.5 供应链和更新链路是“有控制，但不彻底”

正面：

- package-lock.json 与 Cargo.lock 都在。
- Release 工作流会为前端热更新包生成 SHA-256 并写入 latest.json。
- 客户端下载热更新时会校验哈希。

不足：

- Release 工作流里 Apple/Windows 签名变量是注释态，说明签名不是默认生效。
- README 直接告诉 macOS 用户通过 `xattr -rd com.apple.quarantine` 绕过系统拦截，说明发布物默认仍可能未签名或未公证。
- SkillHub 与技能安装链路引入了额外远程资产源。
- 项目大面积使用安装脚本与镜像切换策略，虽然提高可达性，但会增加溯源和采购法务的审核成本。

#### 5.6 遥测与隐私结论

本次针对 `sentry`、`posthog`、`analytics`、`telemetry`、`umami`、`plausible` 等关键词做了定向检索，未发现明显的第三方埋点 SDK 或常见 SaaS 分析器接入。

这意味着它不是那种“默认把用户行为持续上报到分析平台”的产品。

但仍然存在多条外联面：

- `claw.qt.cool` 更新清单
- SkillHub COS 与 `lightmake.site` API
- 用户自行配置的模型 API
- GitHub / Gitee 镜像
- 项目文档与生态页面中的 `gpt.qt.cool`

因此它更适合被归类为“无显式第三方埋点，但存在多外部服务依赖”的中等隐私风险项目。

#### 5.7 许可证与法务姿态

许可证是一个必须单独提醒的点：

- 根 LICENSE 为 AGPL-3.0，并附加商业授权和品牌保护条款。
- package.json 与 Cargo.toml 也都声明 AGPL-3.0。
- 但 GitHub API 返回的是 NOASSERTION，说明 GitHub 未能把其 SPDX 识别为标准、无附加条件的单一许可证态。

这对个人开源使用不是大问题，但对企业采购、闭源集成、SaaS 再包装和品牌使用来说，需要法务先介入。

### 6. Pro 人工分项结论

| 维度 | 判断 |
| --- | --- |
| 自动准入 | REJECTED |
| 攻击面 | 高 |
| 默认权限边界 | 高风险 |
| 供应链风险 | 中高 |
| 遥测/外联风险 | 中 |
| 许可证/法务复杂度 | 中高 |
| 企业直接准入建议 | 不建议 |

## 五、双轨合并判断

ClawPanel 属于非常典型的“产品能力强于治理边界”的开源项目。

从 Matrix 角度看：

- 它是一个完成度高、交付强、生态穿透力强的多引擎 AI 管理面板。

从 Pro 角度看：

- 它把安装、升级、插件、技能、文件系统、命令执行、自动启动、远程更新和多外部服务连接都放进了统一控制面。
- 这对于个人效率是加分项，但对企业准入是减分项。
- 更关键的是，Project-Valhalla-Pro 自动链已经实际给出了 REJECTED，而不是“谨慎通过”。

因此，本项目最准确的定位不是“企业即插即用资产”，而是“强大的社区控制台，需要企业二次硬化后才有可能进入准入名单”。

## 六、适用场景判断

### 更适合

- 个人开发者
- Homelab / 自托管玩家
- 小团队内测环境
- 熟悉本机权限边界与脚本安装风险的高级操作者
- 用作 OpenClaw / Hermes 的统一运维控制台

### 不适合直接使用的场景

- 受监管企业终端
- 大规模员工桌面标准化分发
- 多租户共享主机
- 无法接受本地命令执行和任意文件读写的环境
- 需要严格签名、公证、SBOM、来源固定和默认最小权限的采购体系

## 七、如果要进入企业准入名单，最少需要补的改造

1. 把应用源码中的 pipe-to-shell 安装路径全部移出。
2. 为 AI 助手增加目录白名单、只读模式、命令 allowlist 和逐次确认机制。
3. 把 `GATEWAY_ALLOW_ALL_USERS=true` 和固定 `API_SERVER_KEY=clawpanel-local` 改成默认拒绝与随机密钥。
4. 默认关闭或策略化控制 autostart、插件安装、技能安装、外部镜像切换。
5. 开启正式签名与公证流程，不再依赖用户手工移除 macOS quarantine。
6. 为 SkillHub、插件源、安装脚本增加来源白名单、哈希钉住和更强的供应链证明。
7. 在 CI 中补上 SBOM、依赖审计、许可清单与发布物 provenance。

## 八、最终结论

如果你问的是“这个项目值不值得看”，答案是值得，它在开源 AI Agent 管理面板里明显高于平均线。

如果你问的是“能不能直接进企业标准名单”，答案是否定的。Project-Valhalla-Pro 的自动评测已经给出 REJECTED，理由成立且可解释。它更适合作为产品参考、灰度工具或隔离环境运维面板，而不是未经改造就直接纳入企业终端基线。

## 九、关键证据文件

- GitHub上/clawpanel-eval/clawpanel-main/README.md
- GitHub上/clawpanel-eval/clawpanel-main/src/lib/mirror-urls.js
- GitHub上/clawpanel-eval/clawpanel-main/src/lib/openclaw-kb.js
- GitHub上/clawpanel-eval/clawpanel-main/src/pages/assistant.js
- GitHub上/clawpanel-eval/clawpanel-main/src-tauri/tauri.conf.json
- GitHub上/clawpanel-eval/clawpanel-main/src-tauri/capabilities/default.json
- GitHub上/clawpanel-eval/clawpanel-main/src-tauri/src/commands/assistant.rs
- GitHub上/clawpanel-eval/clawpanel-main/src-tauri/src/commands/hermes.rs
- GitHub上/clawpanel-eval/clawpanel-main/src-tauri/src/commands/skills.rs
- GitHub上/clawpanel-eval/clawpanel-main/src-tauri/src/commands/messaging.rs
- GitHub上/clawpanel-eval/clawpanel-main/src-tauri/src/commands/update.rs
- GitHub上/clawpanel-eval/clawpanel-main/src-tauri/src/commands/skillhub.rs
- GitHub上/clawpanel-eval/clawpanel-main/.github/workflows/ci.yml
- GitHub上/clawpanel-eval/clawpanel-main/.github/workflows/release.yml
- GitHub上/clawpanel-eval/clawpanel-main/LICENSE
- Project-Valhalla-Pro/reports/qingchencloud-clawpanel-pro-auto.json
- Project-Valhalla-Pro/reports/qingchencloud-clawpanel-pro-auto.md
