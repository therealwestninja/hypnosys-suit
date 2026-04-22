# Stillness Stats Visualization Pack

This pack converts the sample analytics object into a set of charts and one flow diagram.

## Recommended primary UI surfaces
1. **Sidebar KPI cards** for total sessions, completion rate, mood improvement, average duration, peak hour, and peak day.
2. **Insights tab** for ranked bar charts: personas, methods, intentions, soundscapes, and visual modes.
3. **Patterns tab** for time-of-day and day-of-week activity.
4. **Wellbeing tab** for mood shift and eventual sparkline from real history records.
5. **Settings profile** for average speech, pitch, narration volume, and drone volume.
6. **Feature adoption** as a compact yes/no bar or checklist generated from the feature bitmask.
7. **Flow/help panel** showing where data comes from and which store powers each visual.

## What the sample data says
- 25 total sessions with 84% normal completion.
- Average mood shifts from 5.6 to 7.8, a gain of +2.2.
- Peak practice time is 21:00, with Wednesday slightly leading weekly usage.
- Favorite combo leans toward driver, highway, rain, and medium.

## Perchance backend guidance
- Keep `stillness_analytics_v1` as the fast O(1) dashboard store.
- Use `stillness_history_v1` only for visuals that need chronology, like heatmaps and sparklines.
- Derive display-only metrics at render time instead of storing redundant values.
- For AI-heavy views in Perchance, preload `aiTextPlugin`, stream responses when useful, and keep a token buffer so context and summaries stay efficient.
- Use Dexie-backed IndexedDB tables and small mirrored settings/profile values for fast replay and prompt injection.

## Files
- 01_persona_usage.png
- 02_method_usage.png
- 03_intention_usage.png
- 04_hourly_pattern.png
- 05_weekday_pattern.png
- 06_mood_shift.png
- 07_completion_outcomes.png
- 08_settings_profile.png
- 09_feature_adoption.png
- 10_dashboard_summary.png
- 11_data_flow_diagram.png
