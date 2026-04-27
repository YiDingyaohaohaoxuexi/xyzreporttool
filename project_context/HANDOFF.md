# Handoff

## Latest Status
- Date: 2026-04-22
- Branch/Repo state: local workspace files (no git repo detected in this folder)
- Main file touched: `XYZNitoriReport.html`

## Completed
- Added theme system with 3 options.
- Moved theme selector to top-right (`top-bar`).
- Renamed theme labels to Japanese:
  - `ダーク`
  - `やわらか`
  - `ライト`
- Ensured export dashboard uses current theme via dynamic CSS generator.
- Added mobile-friendly top-bar behavior for theme control.
- Added `AGENTS.md` with context sync protocol:
  - Manual trigger: `sync ctx`
  - Periodic checkpoints: every 12 turns or ~35 minutes
  - Milestone and end-session mandatory sync guidance
- Added dashboard restore + incremental append feature in `reporttool18TransS - 副本.html`:
  - `Dashboard読込` button + hidden file input.
  - Reads embedded snapshot (`rt-dash-snapshot`) from exported dashboard HTML.
  - Restores previous rows and manual edits (container total / robot intervention / pallet fields).
  - On next query, fetches only orders after latest restored end time and shows them as selectable candidates.
  - Supports `選択追加` or `全件追加` into existing data.
  - Exported dashboard now embeds restore snapshot JSON for next reuse.
  - Snapshot restore now also includes `summaryTitle` / `summaryBody` and `anomalyItems` (images/videos/text).
- Fixed pagination robustness for palletize/order APIs:
  - Handles server-side page-size caps (e.g., actual 50 rows even if requesting 100).
  - Continues pagination without `total` metadata using empty-page termination + repeat-page guard.
- Adjusted palletize order-list display:
  - Main-page `オーダーデータ編集` table no longer shows `コンテナ種別`.
  - Exported dashboard HTML/ZIP order detail table also no longer shows `コンテナ種別`.
- Adjusted effective runtime source:
  - PPH and displayed work-time calculations now prefer `cycle_data.active_run_time`.
  - `cycle_data.total_run_time` is kept only as a fallback for older data / imported snapshots.
- Adjusted exported dashboard KPI:
  - The intervention KPI card now shows `平均ロボット介入率` using average `robotInterventionRate`.
  - Row-level metrics/charts were left intact; Excel export still uses the previous average intervention KPI.
- Added optional overwrite-save flow in `XYZNitoriReport.html`:
  - New `HTML保存/上書き` button uses the browser file-save picker when supported.
  - User can choose the previously imported/exported dashboard HTML file and overwrite it in-place.
  - Standard `HTMLエクスポート` download remains available for new-file export and browser fallback.
- Added robot-intervention time based PPH adjustment in `XYZNitoriReport.html`:
  - New optional `干渉時間制限` toggle controls whether the adjustment is active.
  - When OFF, the app hides the per-intervention input, keeps the old PPH behavior, and shows only `稼働時間`.
  - When ON, `ロボット干渉時間（分/回）` adjusts PPH runtime with `ロボット介入回数`; PPH itself is still capped by `total_run_time`, but the displayed `ロボット稼働時間` shows the raw compensated time before that cap.
  - Snapshot export/import now preserves the feature toggle and runtime fields.
  - KPI/settings UI was refined so enabling the feature no longer awkwardly pushes the new input into a separate broken row; desktop stays aligned and mobile stacks cleanly.
- Fixed dashboard re-import SKU dimension loss in `XYZNitoriReport.html`:
  - Snapshot import now preserves `skuStructs[].dim`.
  - Re-imported dashboards should keep visible SKU size text on page and in re-exported dashboard/Excel output.
- Adjusted imported-dashboard append query behavior in `XYZNitoriReport.html`:
  - User-specified time ranges are now honored even when a dashboard has already been imported.
  - If start time is left blank, the app still defaults to `latest imported end + 1 second`.
  - Existing imported orders are auto-excluded from append candidates so older ranges can be searched without duplicating rows.
- Adjusted imported-dashboard append ordering in `XYZNitoriReport.html`:
  - After candidates are appended, the combined order list is re-sorted chronologically before rebuilding the edit table.
  - Older backfilled orders should no longer stay stuck at the bottom of the table.
- Adjusted single-SKU display wrapping in `XYZNitoriReport.html`:
  - Long single-SKU names now wrap within the `SKU情報` cell instead of visually running into the next column.
- Added `日報モード` export format in `XYZNitoriReport.html`:
  - New `ダッシュボード形式` selector supports `通常レポート` and `日報モード`.
  - `日報モード` keeps the KPI/time-summary style blocks but omits the trend/bar chart section.
  - The top header in `日報モード` now shows only the target date.
  - Exported dashboard snapshots now include `reportMode` and use snapshot version `6`.
  - Importing an older dashboard without `reportMode` falls back to `通常レポート`.
- Added a new multi-daily import workflow in `XYZNitoriReport.html`:
  - New `日報一括読込` button reads multiple exported dashboard HTML files at once.
  - The app merges embedded snapshot rows, anomaly records, and daily summary text into a week-level working dataset.
  - Mixed `荷降ろし` / `パレタイズ` files are rejected during multi-daily import.
  - The merged result now uses the existing `通常レポート` layout instead of a separate weekly mode.
- Reorganized the main action toolbar in `XYZNitoriReport.html`:
  - Actions are now grouped into `読込`, `操作`, and `出力`.
  - The old crowded flat button row was replaced with grouped panels.
  - Export buttons now display in a compact grid on desktop and stack on mobile.
- Added main-page tri-language switching in `XYZNitoriReport.html`:
  - New top-bar language switcher supports `日本語`, `中文`, and `English`.
  - Static UI labels, major hints, report-mode options, and several dynamic section/table headings now react to the selected language.
  - Main-page language is persisted in `localStorage` via `reporttoolMainLangV1`.
  - Main-page font stacks also switch with the selected language.
  - Runtime status banners and most alert/confirm/error prompts now also localize with the selected language.

## Not Completed / Open Checks
- Manual visual QA in browser (desktop + mobile viewport) still recommended.
- Manual QA for restore flow is needed:
  - Export dashboard once with new version.
  - Re-import that dashboard.
  - Verify candidate selection append and edit reuse behavior.
  - Verify summary and anomaly media/text are restored after re-import.
- Manual QA for the new daily-report export mode is needed:
  - Generate one dashboard in `通常レポート` and confirm trend charts still appear.
  - Generate one dashboard in `日報モード` and confirm the chart section is omitted.
  - Confirm the top header shows only a single date and no period range/right-side total count.
  - Re-import the `日報モード` dashboard and verify the selector restores as `日報モード`.
- Manual QA for the new multi-daily import workflow is needed:
  - Export at least two daily dashboards in the same operation mode.
  - Use `日報一括読込` to import them together.
  - Confirm the merged result stays in `通常レポート`.
  - Confirm merged rows are sorted chronologically and anomalies are combined.
  - Confirm a mixed-mode selection (`荷降ろし` + `パレタイズ`) is blocked with an error.
- Manual visual QA for the regrouped action toolbar is needed:
  - Confirm the grouped layout is readable on desktop.
  - Confirm the export block collapses cleanly on mobile.
  - Confirm disabled export buttons still look intentionally unavailable rather than broken.
- Manual QA for the new main-page tri-language switch is needed:
  - Toggle between Japanese, Chinese, and English on desktop.
  - Confirm top-bar controls, form labels, action groups, and summary/anomaly headings update correctly.
  - Confirm append/order-edit/anomaly sections still render correctly after switching language with data already loaded.
  - Confirm the typeface visibly changes to a more suitable Japanese/Chinese/English font stack after switching language.
  - Confirm status/error prompts (import failures, fetch failures, delete confirmations, ZIP export prompts) also change language correctly.
- Optional: tune per-theme typography/colors further based on stakeholder feedback.

## Suggested Next Steps
1. Open `reporttool18TransS - 副本.html` in browser.
2. Query and fill manual values, then export `HTML`.
3. Click `Dashboard読込` and import exported dashboard.
4. Query again and confirm new orders appear in `追加候補オーダー`.
5. Use `選択追加` or `全件追加`, confirm previous manual fields stay restored.
6. Test `HTML保存/上書き` in a Chromium-based browser:
   - choose the previously imported dashboard HTML,
   - confirm overwrite succeeds,
   - confirm a second save can overwrite the same file again in the current session.
7. Test the new PPH rule:
   - verify `干渉時間制限` OFF hides the per-intervention input, shows only `稼働時間`, and matches old PPH behavior,
   - verify `干渉時間制限` ON shows the input and extra time column,
   - set `ロボット干渉時間（分/回)` to a non-zero value,
   - enter `ロボット介入回数` in the order list,
   - confirm `PPH計算時間` increases but never exceeds `total_run_time`,
   - confirm PPH updates accordingly in page/export.
8. If needed, tune palette/spacing per client preference.
9. Test arbitrary-range append after dashboard import:
   - import an older dashboard,
   - set a time range earlier than the imported dashboard's latest order,
   - confirm missed orders in that older range appear as append candidates,
   - confirm already imported orders are auto-excluded,
   - confirm appended older orders are inserted back into chronological order in the edit table.
10. Test the report-format switch:
   - export once in `通常レポート`,
   - export once in `日報モード`,
   - compare that only the standard export includes trend charts,
   - confirm daily export keeps KPI/time/anomaly/summary sections intact.
11. Test week-level generation from daily dashboards:
   - prepare multiple daily dashboard HTML files,
   - click `日報一括読込`,
   - verify merged data appears in the edit table,
   - export the merged result in `通常レポート`,
   - confirm the merged dashboard preserves the full chart-based layout.

## Resume Prompt (copy-paste for next session)
Read `AGENTS.md`, `project_context/PROJECT_CONTEXT.md`, `project_context/DECISIONS.md`, and `project_context/HANDOFF.md`, then continue work on `XYZNitoriReport.html` from the latest state, especially the `日報一括読込` workflow and its browser QA.
