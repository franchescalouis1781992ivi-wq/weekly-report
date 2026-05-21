# PatPat EDM Weekly Dashboard

这个仓库是 PatPat EDM 周报项目的干净交接版 git 工程。

## 仓库用途
- 保存多期周报 HTML / audit / changelog
- 保存 weekly updater skill 与口径说明
- 作为后续 GitHub 部署 / 同事协作的稳定基础

## 目录结构
- `output/`: 多期周报产物
- `skills/patpat-edm-weekly-updater/`: 每周更新周报使用的 skill 与 references
- `README_交接入口.md`: 给同事看的快速入口
- `PATPAT_EDM_WEEKLY_DASHBOARD_REPO_GUIDE.md`: 仓库维护说明

## 每周更新怎么提交
同事每周更新周报时，至少准备下面 4 项：

1. 上一版已确认周报 HTML（baseline）
2. Emarsys 手动邮件周报
3. Emarsys 自动邮件周报
4. Attribuly 归因数据

示例文件名：
- `patpat_edm_weekly_dashboard_2026-05-10_2026-05-16_v70_email_app_revenue_adjusted.html`
- `emarsys手动邮件-TrendReporting5.17-5.23.csv`
- `emarsys自动邮件-TrendReporting5.17-5.23.csv`
- `attribuly归因数据表5.17-5.23.xlsx`

如果自动邮件拿到的是 `.xlsx`，也可以直接使用；Attribuly 也可以是 `.csv`。

## 发给 Codex 的最简模板
```text
请按 patpat-edm-weekly-updater skill 更新周报。

本周周期：2026-05-17 ~ 2026-05-23
基线周报：patpat_edm_weekly_dashboard_2026-05-10_2026-05-16_v70_email_app_revenue_adjusted.html
新增文件：
1. emarsys手动邮件-TrendReporting5.17-5.23.csv
2. emarsys自动邮件-TrendReporting5.17-5.23.csv
3. attribuly归因数据表5.17-5.23.xlsx

要求：append-only，不覆盖历史周，输出新版本 HTML + changelog。
```

## 必须额外说明的口径变动
如果本周有下面任一变化，请在提需求时一起写明：

- Email Revenue 需要补 App 端出单
- 自动邮件白名单新增/删除 campaign
- 手动邮件 theme 需要拆分展示
- 周期修正（例如之前给错日期范围）

这些会直接影响：
- Email Revenue / 私域 Revenue
- Revenue 占比
- 自动邮件 family 汇总
- 手动 theme 展示和历史对比

## 详细样例文档
完整的交接样例见：

- `每周更新提交样例.md`

## 更新原则
1. append-only，只追加新周
2. 历史周冻结，不回写重算
3. 当前稳定基线由业务确认的版本决定
4. 更新时优先遵循 `skills/patpat-edm-weekly-updater/SKILL.md`

## 初始化说明
本仓库已是独立 git 工程，可直接推送到 GitHub。
