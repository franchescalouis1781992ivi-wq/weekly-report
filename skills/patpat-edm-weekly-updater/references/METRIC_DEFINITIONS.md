# PatPat EDM Weekly Metric Definitions (Current Baseline)

## Manual Email
- Send = Emarsys `Total sent` (weekly total row)
- Delivered = Emarsys `Total delivered`
- Opened = Emarsys `Total opened`
- Clicked = Emarsys `Clicked`
- Unsubscribed = Emarsys `Total unsubscribed`
- Open rate = Opened / Delivered
- CTR = Clicked / Delivered
- CTO = Clicked / Opened
- Unsubscribe rate = Unsubscribed / Delivered
- UV = Attribuly `Clicks(UV)` (manual scope)
- Order UV = Attribuly `Attribuly tracked conversions` (manual scope)
- Revenue = Attribuly `Attribuly tracked conversion value` (manual scope)
- CVR = Order UV / UV

Manual scope: `campaign_key` startswith `emanual` or `manual` (under current dashboard rules).

## Automation Email
- Send / Delivered / Opened / Clicked / Unsubscribed from auto Emarsys weekly file.
- UV / Order UV / Revenue from Attribuly auto whitelist scope.
- Keep current whitelist/family logic as baseline unless explicitly changed by user.

## Private Snapshot
- Sessions = Attribuly `Clicks(UV)`
- Revenue = Attribuly `Attribuly tracked conversion value`
- Rows include 大盘 / Email / SMS / 私域合计 with strict definitions:
  - 大盘 = Attribuly 全表汇总（all channels）
  - Email = Attribuly 渠道 Email 汇总（优先 `Channel=Email`；无 `Channel` 时回退 `Medium=email`）
  - SMS = Attribuly 渠道 SMS 汇总（优先 `Channel=SMS`；无 `Channel` 时回退 `Medium=sms`）
  - 私域合计 = Email + SMS
- Revenue share metrics:
  - Email Revenue占比 = Email Revenue / 大盘 Revenue
  - SMS Revenue占比 = SMS Revenue / 大盘 Revenue
  - 私域Revenue占比 = 私域 Revenue / 大盘 Revenue
- If user provides manual Email revenue override (e.g., include App orders), apply override and recompute 私域合计与占比.

## Critical Append Rules
- Never recalc and overwrite historical frozen weeks in baseline html.
- Only append new week data.
