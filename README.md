# v2-Punchcard-visualisation

Splunk custom visualization app, migrated from the Splunkbase "Punchcard" package
(`punchcard-custom-visualization_150 (1).tgz`) into a new, independently-installable
app.

- App ID (directory / `app.conf` id / `app.manifest` id.name): `v2_punchcard_visualisation`
- Display label: `v2-Punchcard-visualisation`
- Source: [`v2_punchcard_visualisation/`](./v2_punchcard_visualisation)
- Original vendor package kept for reference: `punchcard-custom-visualization_150 (1).tgz`

## AppInspect compliance

Validated with Splunk AppInspect (`precert` mode, all tags): **0 errors, 0 failures**.

Fixes applied during migration:
- Renamed the app directory, `app.conf` `[package] id`, and `app.manifest` `info.id.name`
  from `punchcard_app` to `v2_punchcard_visualisation`, and updated every
  `display.visualizations.custom.<app>.punchcard.*` reference (savedsearches.conf,
  savedsearches.conf.spec, gallery.xml) to match.
- Added an `[id]` stanza (`name`/`version`) to `default/app.conf` to satisfy
  `check_for_valid_package_id` / `check_version_is_valid_semver`.
- Added the `sc_admin` role alongside `admin` in `metadata/default.meta` so the
  app-level write ACL is honored on Splunk Cloud (`check_kos_are_accessible`).
- Removed the stale `attribution_link` in `app.conf` (pointed at a mismatched,
  now-outdated Splunk Docs version).
- Updated `app.manifest` title/author and `app.conf` label/author for the new app name.

One warning is intentionally left as-is per requirement: `check_for_updates_disabled`
flags that `check_for_updates = true` is only recommended for apps published to
Splunkbase — `check_for_updates` must **not** be disabled, so `[package]
check_for_updates` stays `true`.

## Changelog

- **1.7.0** — Added a `threshold` color mode: colors circles by numeric
  boundaries on the `count` metric itself (the same value that drives circle
  size), so no separate color field is needed. Configured with
  `thresholdValues` (ascending, comma-separated numbers) and
  `thresholdColors` (comma-separated hex, one more entry than boundaries).
  A value is compared against each boundary in ascending order; hitting a
  boundary exactly puts it in the bucket *above* that boundary (e.g. with
  `thresholdValues=10,20`, a value of exactly 20 gets the third color, not
  the second). Missing/extra colors fall back to Splunk's default palette,
  same as `categoryColors`.
- **1.6.0** — Categorical color mode now accepts an optional
  `categoryColors` property: a comma-separated list of hex colors applied,
  in order, to the distinct values of the color field. Any category beyond
  the list, or all categories when the field is left blank, fall back to
  Splunk's built-in categorical palette (unchanged default behavior).
- **1.5.1** — Fixed an unbounded circle radius: low-cardinality data (few
  distinct x/y groups) could produce circles far larger than the panel,
  clipped at its edge. The radius scale is now capped at 40px.
- **1.5.0** — Initial migration from the Splunkbase "Punchcard" package.

## Packaging

```
tar --numeric-owner --owner=0 --group=0 -czf v2_punchcard_visualisation.spl v2_punchcard_visualisation
```
