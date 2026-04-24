# Valhalla 评测贡献指南 (Contributing Guide)

感谢您参与 **Valhalla Evaluation Reports** 开源共建！为了保持仓库的专业性、客观性和目录整洁，我们建立了一套标准化的内容提交流程。特别是对于高频更新的“每日一评”、“每周大盘”和“经典深度评测”，请务必使用本目录下的标准模板接口。

## 🎯 提交流程

1. **选择接口模板**：根据您要提交的评测类型，在 `docs/templates/` 目录下选择对应的 Markdown 模板。
2. **编写评测报告**：填写模板中的各类维度数据（如双轨架构指标、Benchmark 等）。
3. **放置目录规范**：
    * 每日/每周快报 👉 `04_Weekly_&_Daily_Briefs/`
    * AI Models / 智能体 👉 `01_AI_Models_&_Agents/`
    * 大厂底层生态 👉 `02_BigTech_Infrastructure/`
    * GitHub Top 经典 👉 `03_GitHub_Top_Projects/`
4. **提交 PR**：创建一个描述清晰的 Pull Request，包含评测的核心摘要。

## 📝 预留更新接口 (Templates)

我们预留了以下标准输入接口，方便自动化构建与社区用户的统一贡献：

### 1. 每日热评接口 (Daily Hot Review)
* **用途**：响应 GitHub 24小时内突发爆火开源项目的极速评测。
* **模板**：[daily_review_template.md](./templates/daily_review_template.md)

### 2. 每周最佳接口 (Weekly Best Review)
* **用途**：每周对前沿领域（如 AI, Web, DevOps）汇总排名的全面评估。
* **模板**：[weekly_review_template.md](./templates/weekly_review_template.md)

### 3. 经典/深度硬核接口 (Classic/Depth Review)
* **用途**：针对历史 Star Top 50 库、底层生态的核心基建长篇研究。
* **模板**：[classic_review_template.md](./templates/classic_review_template.md)

请务必在提交前移除文件中涉及内网和机密的任何数据（例如未公开的测试 IP、未脱敏的人员名单和商业数据估值）。
