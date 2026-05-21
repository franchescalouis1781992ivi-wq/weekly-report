# PatPat Weekly Dashboard 交接入口

这个文件夹是 PatPat EDM 周报项目的集中交接包，优先从这里开始。

## 里面有什么

1. 最新周报 HTML
- patpat_edm_weekly_dashboard_2026-05-10_2026-05-16_v70_email_app_revenue_adjusted.html

2. 仓库维护说明
- PATPAT_EDM_WEEKLY_DASHBOARD_REPO_GUIDE.md

3. 周报更新 Skill（项目内版本）
- patpat-edm-weekly-updater_SKILL.md

4. 周报更新 Skill（本机 Codex 运行版本备份）
- patpat-edm-weekly-updater_SKILL_codex_runtime.md

## 推荐使用顺序

1. 先看最新周报 HTML，确认当前稳定基线。
2. 再看仓库维护说明，理解 append-only / frozen history 规则。
3. 再看 Skill，后续每周更新都按 Skill 执行。

## 每周更新时必须准备的 3 份数据

- Emarsys 手动邮件周报
- Emarsys 自动邮件周报
- Attribuly 归因数据

## 最重要的维护原则

- 不覆盖历史周
- 只追加新周
- 自动邮件 family 映射按当前 skill 固定规则执行
- 私域总数据的大盘 = Attribuly 全表，不是 Email + SMS
- 若用户给出 Email Revenue 手工修正值（如 App 端补量），必须同步重算私域合计和占比
