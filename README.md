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

## Packaging

```
tar --numeric-owner --owner=0 --group=0 -czf v2_punchcard_visualisation.spl v2_punchcard_visualisation
```
