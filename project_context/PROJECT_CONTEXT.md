# Project Context

## Project
- Name: reporttool dashboard generator
- Main working file: `XYZNitoriReport.html`
- Purpose: query order/task data, edit/merge rows, record anomalies, export dashboard (HTML/ZIP/Excel)

## Scope
- Frontend single-file app (HTML/CSS/JS)
- No backend changes in this repo

## Current UI/Theming State
- Home page has a top-right `UI theme` selector.
- Home page now also has a top-bar language switcher for:
  - `日本語`
  - `中文`
  - `English`
- Main-page font family now switches with the selected language so Japanese, Chinese, and English each use a more appropriate typeface stack.
- Main-page runtime messages now also follow the selected language, including status banners and most alert/confirm/error prompts.
- Main action area is grouped into three sections:
  - `読込`
  - `操作`
  - `出力`
- Theme labels shown to users:
  - `ダーク`
  - `やわらか`
  - `ライト`
- Internal theme keys:
  - `modern-dark`
  - `japan-washi`
  - `japan-corporate`
- Current selected theme is persisted in `localStorage` key:
  - `reporttoolThemeModeV1`
- Current selected main-page language is persisted in `localStorage` key:
  - `reporttoolMainLangV1`

## Export Behavior
- Exported dashboard HTML/ZIP uses the same currently selected home-page theme.
- Dashboard export format now supports:
  - `通常レポート`
  - `日報モード`
- In `日報モード`, exported dashboard omits trend/bar chart sections and shows only the target day's date in the top header.
- `日報一括読込` can import multiple daily dashboard HTML files, merge their embedded snapshots, and prepare week-level data for the normal report layout.
- Theme CSS is generated through:
  - `getDashboardThemeCSS(theme, kpiCols)`
- Exported dashboard also embeds restore snapshot JSON:
  - script id: `rt-dash-snapshot`
  - snapshot includes order rows + summary + anomaly media/text + report mode
- In `palletize` mode, both the home-page order list and exported dashboard order list hide the `コンテナ種別` column.
- Dashboard KPI card for intervention now shows average robot intervention rate (`avgRobotIR`) instead of the generic intervention rate average.
- In `XYZNitoriReport.html`, HTML export now also supports an optional browser file-save flow that can overwrite a previously chosen dashboard HTML when the browser supports File System Access API.
- `XYZNitoriReport.html` now has an optional `干渉時間制限` toggle. When OFF, it keeps the old behavior and shows only `稼働時間`. When ON, PPH still calculates with `min(active_run_time + robotInterventionCount * minutesPerIntervention, total_run_time)`, but the displayed `ロボット稼働時間` column shows the raw `active_run_time + intervention compensation` value before that cap.
- Dashboard snapshot import in `XYZNitoriReport.html` preserves SKU `dim` text from `skuStructs` so re-imported dashboards do not lose visible size information.
- After importing a dashboard in `XYZNitoriReport.html`, append-candidate queries can now target any user-specified time range; only when start time is left blank does the app default to `latest imported end time + 1 second`. Existing imported orders are auto-excluded by content-based order matching.
- After appending candidates into an imported dashboard in `XYZNitoriReport.html`, `allData` is re-sorted chronologically by start/end time before the edit table is rebuilt, so older appended orders are inserted back into timeline order.

## Key Functions
- Theme init/apply:
  - `isValidTheme`
  - `getCurrentTheme`
  - `applyTheme`
  - `initTheme`
- Export core:
  - `buildDashboardHTML`
  - `_buildDashboardCore`
  - `exportDashboardHTML`
  - `exportZipPackage`
  - `saveDashboardHTMLToFile`
- Restore/incremental flow:
  - `triggerDashboardImport`
  - `triggerWeeklyDashboardImport`
  - `extractDashboardSnapshot`
  - `applyDashboardSnapshot`
  - `applyWeeklyDashboardSnapshots`
  - `cloneAnomalyItemsForSnapshot`
  - `normalizeImportedAnomalyItems`
  - `refreshRestoreMapFromCurrent`
  - `renderAppendCandidates`
  - `appendSelectedCandidates`
  - `appendAllCandidates`
  - `queryData` (incremental candidate fetch logic)

## Constraints
- Keep compatibility with existing data query/edit/export flow.
- Avoid breaking existing i18n/translation and anomaly media export logic.
- Preserve compatibility when importing older dashboard snapshots that do not yet include `reportMode`.
- Weekly import expects the selected dashboards to share the same operation mode (`unload` or `palletize`).

## Context Management
- Repository-level context sync protocol is defined in `AGENTS.md`.
- Manual sync trigger phrase: `sync ctx`.
- Primary memory files:
  - `project_context/PROJECT_CONTEXT.md`
  - `project_context/DECISIONS.md`
  - `project_context/HANDOFF.md`
