# Decisions Log

## Format
- Date: YYYY-MM-DD
- Decision:
- Why:
- Impact:

---

## 2026-04-19
- Decision: Add 3-theme switching on home page and sync theme to exported dashboard.
- Why: User requested theme alternatives for Japanese customer visual preferences and consistent output style.
- Impact:
  - Added theme selector.
  - Added `modern-dark`, `japan-washi`, `japan-corporate`.
  - Exported HTML/ZIP now reflects selected theme.

## 2026-04-19
- Decision: Move `UI theme` selector to top-right area and rename labels to Japanese terms.
- Why: User requested higher visibility and simplified naming.
- Impact:
  - Selector moved from KPI settings card to top bar.
  - Labels changed to `暗色`, `柔和`, `明亮`.
  - Added responsive top-bar wrapping for smaller screens.

## 2026-04-19
- Decision: Introduce repository-level context sync protocol via `AGENTS.md`.
- Why: Reduce risk of memory loss in long sessions and keep handoff continuity.
- Impact:
  - Added trigger-based update cadence (manual trigger + turn/time checkpoints + milestone/end-session sync).
  - Standardized update responsibilities for `PROJECT_CONTEXT.md`, `DECISIONS.md`, and `HANDOFF.md`.
  - Added explicit manual trigger phrase: `sync ctx`.

## 2026-04-19
- Decision: Replace theme display labels with more natural Japanese wording.
- Why: Previous labels had Chinese-like phrasing and were not idiomatic for Japanese UI.
- Impact:
  - `暗色` -> `ダーク`
  - `柔和` -> `やわらか`
  - `明亮` -> `ライト`

## 2026-04-20
- Decision: Add dashboard snapshot import + incremental append workflow in `reporttool18TransS - 副本.html`.
- Why: Avoid re-entering container total and robot intervention values for previously processed orders.
- Impact:
  - Added `Dashboard読込` UI and snapshot restoration flow.
  - Exported dashboard now embeds `rt-dash-snapshot` JSON for future restore.
  - Query now supports incremental fetch from imported dashboard latest end time (+1 second).
  - Restored manual edits (`containerTotal`, `robotIntervention`, pallet fields) are reused automatically.

## 2026-04-20
- Decision: Switch incremental append behavior from automatic append to manual candidate selection.
- Why: User asked for selecting which newly fetched orders should be merged into previous dashboard.
- Impact:
  - Added `追加候補オーダー` section with checkbox selection.
  - Added `選択追加` and `全件追加` actions.
  - Incremental fetch now shows candidates first, then appends only chosen rows.

## 2026-04-20
- Decision: Extend dashboard snapshot restore to include summary text and anomaly records with media.
- Why: User needs imported dashboard to preserve photos/videos and text summary, not only order rows.
- Impact:
  - Snapshot schema upgraded to `version: 3`.
  - Added `summaryTitle`, `summaryBody`, `anomalyItems` into embedded snapshot.
  - Import now restores summary fields and anomaly list/media.

## 2026-04-20
- Decision: Harden pagination logic for palletize/order list API when server caps page size to 50 and omits total.
- Why: Existing logic stopped early because it assumed `items.length===requestedPageSize` for continuation.
- Impact:
  - `fetchAllPages` now supports multiple pagination metadata patterns (`total/count/has_next/total_page/next`).
  - When total is missing, it continues until empty page (with repeat-page safety guard).
  - Added duplicate-row protection and higher safety limits.

## 2026-04-21
- Decision: Hide `コンテナ種別` from palletize order lists on both the home page and exported dashboard.
- Why: User requested that palletize order views not show container-type information.
- Impact:
  - Main-page `オーダーデータ編集` table now omits the column in `palletize` mode.
  - Exported dashboard HTML/ZIP order detail table also omits the column in `palletize` mode.

## 2026-04-21
- Decision: Treat `active_run_time` as the effective runtime for PPH and work-time calculations, with `total_run_time` only as a fallback.
- Why: User clarified that effective runtime should come from `active_run_time`, not `total_run_time`.
- Impact:
  - Candidate list, main-page order aggregation, exported dashboard, and imported snapshot rows now use the same runtime source.
  - Legacy data remains compatible because `total_run_time` is still used when `active_run_time` is absent.

## 2026-04-21
- Decision: Show average robot intervention rate in the exported dashboard KPI card instead of average generic intervention rate.
- Why: User requested the dashboard's intervention KPI to reflect robot intervention rate specifically.
- Impact:
  - Dashboard HTML/ZIP KPI label changed to `平均ロボット介入率`.
  - KPI value and pass/fail judgment now use `avgRobotIR`.
  - Existing row-level intervention metrics and Excel export remain unchanged.

## 2026-04-21
- Decision: Add optional overwrite-save flow for dashboard HTML in `XYZNitoriReport.html` using the browser file-save picker.
- Why: User wants to keep appending orders from a previously imported dashboard and then update that same dashboard file instead of always downloading a new file.
- Impact:
  - Added `HTML保存/上書き` button alongside the existing HTML download export.
  - Users can choose the previous dashboard HTML file and overwrite it when the browser supports File System Access API.
  - Existing `HTMLエクスポート` download behavior remains unchanged as a fallback/new-file path.

## 2026-04-21
- Decision: Add robot-intervention time as a PPH calculation constraint in `XYZNitoriReport.html`.
- Why: User requested a configurable per-intervention time so PPH reflects robot intervention overhead but does not exceed `total_run_time`.
- Impact:
  - Added optional `干渉時間制限` toggle plus hidden `ロボット干渉時間（分/回）` input in the KPI/settings area.
  - When OFF, the page keeps the old PPH behavior and shows only `稼働時間`.
  - When ON, PPH runtime uses `min(active_run_time + robotInterventionCount * perInterventionMinutes, total_run_time)`, while the displayed `ロボット稼働時間` shows the uncapped `active_run_time + intervention compensation`.
  - Dashboard snapshot version bumped to `5` and now stores the feature toggle plus `robotInterventionTimeMin`, `displayRunSec`, `activeRunSec`, and `totalRunSec`.

## 2026-04-22
- Decision: Preserve SKU dimension text when importing dashboard snapshots in `XYZNitoriReport.html`.
- Why: Re-imported dashboards were losing SKU size text because `skuStructs` restoration only kept weight and name fields.
- Impact:
  - `buildImportedSkuList` now carries `dim` through snapshot import.
  - `formatSkuInfo` and `formatSkuStructured` now fall back to direct `dim` text when raw length/width/height fields are absent.
  - Previously exported dashboards can be re-imported without dropping visible SKU dimensions.

## 2026-04-22
- Decision: Let imported-dashboard append queries in `XYZNitoriReport.html` use arbitrary user-specified time ranges instead of always forcing only orders after the latest imported order.
- Why: User needs to append missed orders from older time windows, not only orders newer than the imported dashboard's last order.
- Impact:
  - If a start time is explicitly entered, query uses that range as-is.
  - If start time is blank after dashboard import, the old incremental default (`latest imported end + 1 second`) is still used for convenience.
  - Existing imported orders are automatically excluded from append candidates via content-based order matching.

## 2026-04-22
- Decision: Re-sort appended orders chronologically after merging append candidates into an imported dashboard in `XYZNitoriReport.html`.
- Why: Appending older orders was leaving them at the bottom of the edit table, making the timeline look incorrect and confusing row order for further edits/merges.
- Impact:
  - `allData` is now sorted by start time, then end time, after selected/all append operations.
  - Initial direct query load also uses the same chronological sort helper for consistency.
  - The order-edit table now reflects the real timeline more reliably after backfilling older orders.

## 2026-04-22
- Decision: Add a dedicated `日報モード` export format in `XYZNitoriReport.html`.
- Why: When the dashboard covers only a single day, multi-container trend/bar charts add noise and a daily-summary oriented layout is more useful.
- Impact:
  - Added a `ダッシュボード形式` selector with `通常レポート` and `日報モード`.
  - In `日報モード`, exported HTML/ZIP dashboards omit the trend chart section and simplify the top header to show only the target date.
  - Dashboard snapshots now store `reportMode` and use snapshot version `6`; importing older snapshots still falls back safely to the standard mode.

## 2026-04-22
- Decision: Add a new multi-daily import workflow that merges multiple daily dashboard HTML files into the normal report dataset instead of maintaining a separate weekly mode.
- Why: User wants to keep the original single-query and single-dashboard flows, while also being able to reuse previously generated daily dashboards to create a week-level dashboard without introducing a redundant mode alongside `通常レポート`.
- Impact:
  - Kept `日報一括読込` as a separate entry point that reads multiple exported dashboard HTML files, extracts their embedded snapshots, and merges rows/anomalies.
  - Removed the separate `週報モード`; merged week-level data now uses the existing `通常レポート` layout.
  - Weekly imports still require a consistent operation mode across selected files.

## 2026-04-22
- Decision: Reorganize the main action buttons in `XYZNitoriReport.html` into grouped toolbar sections.
- Why: The growing number of actions made the old single-row button layout feel crowded and harder to scan.
- Impact:
  - Grouped actions into `読込`, `操作`, and `出力`.
  - Kept all existing actions and labels, but reduced visual clutter by separating intent.
  - Export actions now sit in a compact 2-column block on desktop and stack on mobile.

## 2026-04-22
- Decision: Add main-page tri-language switching in `XYZNitoriReport.html`.
- Why: User wants the primary working UI itself, not only exported dashboards, to support Japanese, Chinese, and English.
- Impact:
  - Added a top-bar language switcher for `日本語`, `中文`, and `English`.
  - Localized the main page's static controls, key hints, option labels, and several dynamic table/section headings.
  - Main-page language preference is now stored in `reporttoolMainLangV1`.
  - Main-page font stacks now switch with the selected language so Japanese, Chinese, and English each use a more appropriate typeface family.
  - Runtime UI messages now also follow the selected language, including status banners and most alert/confirm/error prompts.

## 2026-04-27
- Decision: Add free-form `PM / FAE` info inputs to the home page and export them into Excel/dashboard snapshots.
- Why: User needs report-level personnel/contact information to be entered once on the main page and carried through both output formats.
- Impact:
  - Added a dedicated `PM / FAE` section with multiline text inputs on the main page.
  - Excel export now includes a `担当者情報` block when either field is filled.
  - Dashboard HTML now renders a contact-info section and embedded snapshots preserve `pmInfo` / `faeInfo` for re-import.
