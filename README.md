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

## 更新原则
1. append-only，只追加新周
2. 历史周冻结，不回写重算
3. 当前稳定基线由业务确认的版本决定
4. 更新时优先遵循 `skills/patpat-edm-weekly-updater/SKILL.md`

## 初始化说明
本仓库已是独立 git 工程，可直接推送到 GitHub。
