# PatPat EDM Weekly Dashboard Repo Guide

这个仓库用于维护 PatPat EDM 周报项目，重点是保证每周周报 **append-only** 更新，避免历史周被重算漂移。

## 当前基线

- 当前参考周报基线：
  patpat_edm_weekly_dashboard_2026-05-10_2026-05-16_v70_email_app_revenue_adjusted.html

说明：
- 后续新增周报时，优先以“用户已确认的最新稳定版 HTML”为 baseline。
- 不要回滚或覆盖旧周数据。
- 新周数据只能追加到 `DATA.week_data` 中。

## 关键目录

- `output/`: 所有版本化周报 HTML、audit、changelog
- `skills/patpat-edm-weekly-updater/`: 仓库内交接用 skill
- `app.py` / `requirements.txt`: 这个仓库里原有的其他工具，不属于周报主线

## 周报维护规则

1. 历史周冻结
- 已确认周报只允许被引用，不允许被重算覆盖。

2. 每周追加
- 新周报统一输出为新文件名，不覆盖旧版本。

3. 口径优先级
- 手动邮件发送/送达/打开/点击：Emarsys 手动周报
- 自动邮件发送/送达/打开/点击：Emarsys 自动周报
- Revenue / UV / Order UV：Attribuly
- 私域总数据：
  - 大盘 = Attribuly 全表
  - Email / SMS = 渠道汇总
  - 私域合计 = Email + SMS

4. 自动化 family 规则
- 自动邮件 family 白名单、compare 规则、welcome 扩展项，以 skill 文件为准。

## 推荐操作方式

1. 先确认最新被业务认可的 baseline HTML。
2. 再提供三份新周数据：
- Emarsys 手动
- Emarsys 自动
- Attribuly
3. 使用仓库内 skill 更新周报。
4. 输出新 HTML + changelog，不覆盖旧文件。
