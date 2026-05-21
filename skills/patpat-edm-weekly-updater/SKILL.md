---
name: patpat-edm-weekly-updater
description: Use this skill when updating PatPat EDM weekly dashboard data (manual/automation/attribuly) by appending a new week to the existing approved HTML baseline without rewriting historical weeks. It preserves frozen historical data, applies the established metric definitions, and updates weekly trend + period selector + automation family attribution map.
---

# PatPat EDM Weekly Updater

## When To Use
Use this skill when the user asks to:
- add a new week (e.g. `3.29-4.4`) into the existing weekly dashboard,
- keep prior weeks unchanged,
- preserve existing metric logic for manual/automation/private snapshot,
- regenerate a new versioned HTML.

## Core Rules (Do Not Break)
1. **Append-only**: add new week to `DATA.week_data` as `wN`.
2. **Frozen history**: do not overwrite old weeks already in approved baseline.
3. **Single source of truth**:
- Manual send/engagement = Emarsys weekly total row (`Total sent`, `Total delivered`, ...)
- Attribution metrics = Attribuly weekly file
3.1 **Private snapshot rule (must follow)**:
- `大盘` = Attribuly 全表汇总（all channels, all campaigns），不是 Email+SMS。
- `Email` / `SMS` 从 Attribuly 渠道维度汇总（优先 `Channel` 字段；若无则回退 `Medium`）。
- `私域合计` = Email + SMS。
- `Revenue占比` 一律以 `大盘 Revenue` 为分母。
3.3 **Private share card compare fallback (must follow)**:
- 私域卡片中的 `Email Revenue占比（占大盘）` / `SMS Revenue占比（占大盘）` / `私域Revenue占比（占大盘）` 必须显示上周与WoW。
- 若当前周 `private_rows` 缺少 `对比周期 Revenue%` 或 `Revenue% 环比`，必须自动回填：
  - 上周占比 = 上一周同维度 `当前周期 Revenue%`
  - WoW = `(本周占比 - 上周占比) / 上周占比`
- 禁止将占比卡片上周/WoW写死为 `null`。
3.2 **Manual override support**:
- 若用户明确给出 Email Revenue 手工修正值（如含 App 端补量），必须覆盖当前周 Email Revenue，
- 并同步更新 `私域合计 Revenue = Email + SMS`、私域卡片、以及相关占比字段。
4. **Automation attribution scope (must follow)**:
- 自动邮件归因默认只统计 **Email channel**（`Channel=email`，若无则 `Medium=email`）。
- SMS 的 automation campaign（如 `subscribe-welcome_*` 且 `Channel=SMS`）**不能计入自动邮件**。
- 自动邮件 Revenue 校验口径必须可回溯到 Attribuly campaign 明细。
4.1 **Automation family whitelist (hardened rule, must follow)**:
- 自动邮件 family 层固定使用以下 5 类（用于自动模块总览与 By Flow）：
  - `订阅欢迎`
  - `浏览未购`
  - `加购未购`
  - `reactivation`
  - `首购后入会`
- Attribuly family 映射白名单（大小写不敏感）：
  - `订阅欢迎` = `subscribe-welcome_2025_09` + `subscribe-welcome_2025_11` + `eautomation_Sale_welcome_us-1_Miki_2026_2509`
  - `浏览未购` = `abandoned-browse_2025_10`
  - `加购未购` = `abandoned-cart_2025_09`
  - `reactivation` = `engagement_2025_11`
  - `首购后入会` = `membership_2025_10`
- 禁止将 `sale_* / gift-card_* / order-confirmation_* / order-status-update_* / order-delivery_*` 误纳入自动 family 销售。
4.2 **Automation top-vs-flow consistency (hardened rule, must follow)**:
- 自动模块上方 KPI（Revenue/UV/Order UV/CVR/AOV/RPR）与下方 By Flow 的 family 汇总必须来自同一套 `AUTO_FAMILY_ATTR_BY_RANGE` 映射。
- 禁止出现“上方总览一套口径、下方By Flow另一套口径”。
- 如发现差异，优先修 family 映射，不允许只改展示层。
4.3 **Automation family revenue extension (hard-locked)**:
- `订阅欢迎` family 的 Attribuly 白名单必须强制包含：
  - `subscribe-welcome_2025_09`
  - `subscribe-welcome_2025_11`
  - `eautomation_Sale_welcome_us-1_Miki_2026_2509`
- 任何周如果这条 `eautomation_Sale_welcome_us-1_Miki_2026_2509` 出现在 Attribuly 且 Channel=Email，必须计入自动化与订阅欢迎 family。

4.4 **Automation send totals and compare source (hard-locked)**:
- 自动邮件总览 `Send/Delivered/Opened/Clicked/Unsubscribed` 只能来自“当周 Emarsys 自动邮件源数据总和”（有效 campaign 行求和）。
- 禁止从 Flow/Node allocation 反推自动总览。
- By Flow 的“上期值”必须来自“上一周同一 family 的 By Flow 汇总”，不能跨错周或取其他快照。

4.5 **Automation family coverage guardrail (hard-locked)**:
- `reactivation` 与 `首购后入会`（membership）在 Emarsys 中有发送时，By Flow 不允许显示全 0。
- 若出现 0，必须先判定是解析漏识别；修映射后再交付。

5. **Theme merge consistency (must follow)**:
- `theme_attr` 与 `theme_em_detail` 必须使用同一套业务主题键（如 `sale/holiday/ip/kids`）。
- 需要先做发送主题到业务主题的映射（例如 `vacation->holiday`, `stitch->ip`, `happy/member/mommy/flash/new->sale`）。
- 前端按 `日期 + Theme` 合并时，交集必须覆盖 100%，否则视为阻断错误（会导致 Send/Delivered/Opened 全 0）。
6. **Automation flow consistency**:
- keep existing parse + unresolved logic
- update `AUTO_FAMILY_ATTR_BY_RANGE` for the new week to avoid auto revenue showing as 0.
7. **Automation compare consistency (mandatory)**:
- keep Auto Flow tables in **current vs previous** compare mode (WoW enabled),
- preserve two-line cell pattern in Auto Flow tables:
  - line1: current value
  - line2: previous value (light gray) + WoW (up/down soft color),
- never collapse back to single-line crowded rendering.
8. **UI compatibility**:
- keep existing IDs and layout order
- only update data block / period options / default week.

## Manual Email Analysis Enhancement Rules (v40+)
These rules are mandatory whenever user asks to enhance/fix manual analysis compare/trend:

1. **Three-week assembly must be built from adjacent week blocks**
- If current key is `w7`:
  - 本周 = `w7.manual_rows[*].本周`
  - 上周 = `w6.manual_rows[*].本周`
  - 上上周 = `w5.manual_rows[*].本周`
- Do **not** rely on `w7.manual_rows[*].上周` for multi-week backfill.

2. **Manual top cards must be re-constructed (not direct reuse of `manual_cards`)**
- Required cards: `Revenue`, `Send`, `Delivered`, `Open rate`, `CTR`, `Order UV`, `AOV`.
- Card main value = 本周；sub value = 上周；WoW = `(本周-上周)/上周`.
- `AOV = Revenue / Order UV`, and should work for 本周/上周/上上周.
- If denominator is 0, render `—`.

3. **CTR precision in manual module**
- CTR must display with **3 decimals** in manual scope:
  - manual cards
  - manual compare table
  - manual weekly trend table
- Example: `0.0526% -> 0.053%`.

4. **Manual compare table columns (fixed)**
- `指标 | 本周 | 上周 | 上上周 | WoW | vs 上上周`
- `WoW = (本周-上周)/上周`
- `vs 上上周 = (本周-上上周)/上上周`
- Keep up/down color classes consistent.

5. **Manual weekly trend table (N-week switch)**
- Add/maintain selector: `3周 / 4周 / 6周`.
- Build trend from ordered `week_data` keys, not current-week compare fields.
- Required metrics in trend table:
  - `Send`, `Delivered`, `Open rate`, `CTR`, `Order UV`, `Revenue`, `AOV`, `CVR`.

6. **Scope isolation**
- Only change manual module behavior when implementing this enhancement.
- Do not modify private/auto/business logic unless explicitly requested.

## Auto Email Trend Enhancement Rules (v45+)
These rules are mandatory whenever user asks to add/fix automatic email weekly trend:

1. **Insertion position and scope (strict)**
- Insert `自动Email周度趋势` only inside `section.core-auto.hidden`.
- Place it after `autoCards + autoTbl + auto note`, and before `自动邮件 Flow 看板`.
- Do not change manual/private/appendix/global structure.

2. **Required DOM IDs**
- week selector: `autoTrendWeeks` (`3周` / `6周`, default `3周`)
- trend table: `autoTrendTbl`

3. **Trend data assembly (must follow current period)**
- End week = current `periodMode` selected week key.
- Backtrack last N weeks from ordered keys (`3` or `6`).
- If available weeks < N, render all available weeks without error.

4. **Data source and metric consistency (must follow)**
- Reuse auto overview source via existing weekly aggregator (e.g. `buildAutoOverviewFromWeek(week_data[w])`).
- Never back-calculate from Flow/Site/Node breakdown rows.
- Trend metrics fixed to:
  - `Send`, `Delivered`, `Open rate`, `CTR`, `Order UV`, `Revenue`, `AOV`, `CVR`

5. **Cell rendering style (align with manual trend)**
- first row: main value + subline `—`
- later rows: main value + WoW vs previous displayed week
- color classes follow existing up/down/na logic
- `CTR` shows 3-decimal percentage (`fmtRate3`)

6. **Interaction**
- changing `periodMode` refreshes `autoTrendTbl`
- changing `autoTrendWeeks` refreshes only auto trend table
- must not affect `manualTrendWeeks` / `manualTrendTbl`

7. **Minimal-change rule**
- no refactor of existing auto flow logic
- no new business metric definitions
- no changes to existing IDs/classes/functions unless strictly required

## Inputs Required Each Week
- 1) Manual Emarsys weekly file (`.csv`)
- 2) Auto Emarsys weekly file (`.csv` or `.xlsx`)
- 3) Attribuly weekly file (`.csv` or `.xlsx`)
- 4) Current approved baseline HTML (latest stable version)

Checklist is in [references/DATA_INPUT_CHECKLIST.md](references/DATA_INPUT_CHECKLIST.md).

## Standard Workflow
1. Read baseline HTML and parse `const DATA = {...}`.
2. Detect latest week key and append next key (`w5`, `w6`, ...).
3. Compute new week objects with existing logic:
- `private_rows`
- `overall_email_rows`
- `manual_rows` + `manual_cards`
- `theme_attr` + `theme_em_detail`
- `carry_rows` + `carry_totals`
- `raw_manual_rows` + `raw_auto_rows`
- `auto_rows` + `auto_cards`
- `auto_flow_data` + `auto_validation_rows`
4.1 Ensure automation overview includes derived KPI:
- `RPR = Revenue / Delivered` (Revenue per Recipient)
4.2 Ensure automation compare block remains available:
- show current / previous / WoW for auto cards and auto table
4.3 Ensure flow drilldown compare remains available:
- flow/site/node table cells keep second-line previous + WoW display
4.4 If manual enhancement is requested, apply `Manual Email Analysis Enhancement Rules (v40+)`.
4.5 If auto trend enhancement is requested, apply `Auto Email Trend Enhancement Rules (v45+)`.
5. Update:
- `current_range`, `previous_range`, `older_range`
- `default_week`
- `weekly_trend_rows` append new week
- period selector options in HTML (`<select id="periodMode">`)
- `AUTO_FAMILY_ATTR_BY_RANGE` append current week block
- 自动邮件归因口径固定为 Email channel only（除非用户明确要求改口径）
- `private_rows` 中补齐并校验 Revenue 占比字段：
  - `Email Revenue占比（占大盘）`
  - `SMS Revenue占比（占大盘）`
  - `私域Revenue占比（占大盘）`
6. Save as a **new versioned html** in `output/`.

Detailed formula and guardrails: [references/METRIC_DEFINITIONS.md](references/METRIC_DEFINITIONS.md)

## Parsing Guardrails
- Emarsys campaign date token must support both:
- `YYMMDD` (6 digits)
- `YYYYMMDD` (8 digits)
- If auto source is xlsx, convert to csv before loading.
- If parse fails, keep unresolved rows visible; do not silently drop.

## Validation (Minimum)
After writing new HTML, always print/check:
- new week manual send
- new week manual revenue
- new week auto send
- new week auto revenue
- new week auto RPR
- auto revenue(email only) 与 Attribuly 自动 campaign（Email channel）求和一致
- auto family revenue check（must pass）:
  - `订阅欢迎` 是否等于 `subscribe-welcome_2025_09 + subscribe-welcome_2025_11 + eautomation_Sale_welcome_us-1_Miki_2026_2509`
  - auto flow family revenue total 是否等于白名单 family revenue 求和
- auto top-vs-flow check（must pass）:
  - 自动模块上方 Revenue/UV/Order UV 是否与 By Flow family 汇总一致
- auto flow previous-week check（must pass）:
  - By Flow 每个 family 的“上期”必须等于上一周同 family 的当周值（禁止错配到更早周）
- auto family non-zero coverage check（must pass when source has data）:
  - 若 Emarsys 本周 `reactivation` 或 `membership` 有发送量/送达量，则 By Flow 对应行不可为 0
- auto welcome whitelist check（must pass）:
  - `订阅欢迎 Revenue` 必须等于 `subscribe-welcome_2025_09 + subscribe-welcome_2025_11 + eautomation_Sale_welcome_us-1_Miki_2026_2509`（Email channel）
- auto compare block shows previous + WoW (not missing)
- auto flow summary/site/node tables still render second-line previous + WoW
- `theme_attr` vs `theme_em_detail` 的 `日期+Theme` key 交集覆盖率 = 100%
- 若 key 未对齐，禁止交付（否则主题汇总发送数据会全部变 0）
- private snapshot check:
  - 大盘 Revenue 是否等于 Attribuly 全表 Revenue
  - 私域 Revenue 是否等于 Email + SMS
  - Email/SMS/私域 Revenue% 是否按“大盘 Revenue”为分母
  - 私域卡片三张占比（Email/SMS/私域）必须有上周值；缺失时需由上一周当前占比回填
  - 私域卡片三张占比 WoW 必须可计算或明确为 `—`（仅在上周占比为0/缺失时）
- unresolved count
- week selector includes new range
- `DATA.week_data` contains old + new weeks

### Additional Validation For Manual Enhancement (v40+)
- Manual cards display non-empty `上周` when previous week exists.
- Manual compare table has 6 columns (`本周/上周/上上周/WoW/vs 上上周`).
- `CTR` renders with 3 decimals in manual cards/table/trend.
- `AOV` exists in both manual cards and manual table.
- Trend selector (`3/4/6周`) switches data correctly.
- For current `wN`, values match `wN / wN-1 / wN-2` join by metric name.

### Additional Validation For Auto Trend Enhancement (v45+)
- `autoTrendWeeks` selector exists with only `3周/6周`.
- `autoTrendTbl` renders rows oldest→newest for selected N weeks.
- `periodMode` switch refreshes auto trend correctly.
- `autoTrendWeeks` switch refreshes only auto trend block.
- Trend metrics match auto overview definitions (no flow/node reverse aggregation).
- CTR in auto trend renders with 3-decimal percentage.

## Output Convention
Create:
- new HTML: `patpat_edm_weekly_dashboard_<new_range>_vXX.html`
- short changelog txt in same folder.

## References
- [references/METRIC_DEFINITIONS.md](references/METRIC_DEFINITIONS.md)
- [references/DATA_INPUT_CHECKLIST.md](references/DATA_INPUT_CHECKLIST.md)
- [references/WEEKLY_UPDATE_PROMPT_TEMPLATE.md](references/WEEKLY_UPDATE_PROMPT_TEMPLATE.md)
