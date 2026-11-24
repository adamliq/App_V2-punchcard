# v2-Punchcard-visualisation

Splunk custom visualization app, migrated from the Splunkbase "Punchcard" package
(`punchcard-custom-visualization_150 (1).tgz`) into a new, independently-installable
app.

Splunk now ships a native built-in "Punchcard" visualization in Dashboard Studio.
This app remains useful alongside it specifically for its four configurable color
modes — categorical with custom hex colors, a sequential gradient, numeric
threshold buckets on the count itself, and field-value colors computed entirely
in SPL — none of which the built-in visualization offers. It also works inside
Dashboard Studio (confirmed by adding it to a Studio dashboard, searching real
data, and configuring every color mode through Studio's native Configuration
panel) as well as Classic (Simple XML) dashboards, so you're not limited to one
dashboard type.

- App ID (directory / `app.conf` id / `app.manifest` id.name): `v2_punchcard_visualisation`
- Display label: `v2-Punchcard-visualisation`
- Source: [`v2_punchcard_visualisation/`](./v2_punchcard_visualisation)
- Original vendor package kept for reference: `punchcard-custom-visualization_150 (1).tgz`
- Examples: [`gallery.xml`](./v2_punchcard_visualisation/default/data/ui/views/gallery.xml)
  (Classic) and [`studio_gallery.xml`](./v2_punchcard_visualisation/default/data/ui/views/studio_gallery.xml)
  (Dashboard Studio), both built on the bundled `bikeshare.csv` lookup

## Color modes

Set `display.visualizations.custom.v2_punchcard_visualisation.punchcard.colorMode`
(or the "Color mode" field in the Format panel) to one of the following. All
examples below also work as `<option>` tags in dashboard XML or
`display.visualizations.custom.v2_punchcard_visualisation.punchcard.*` keys in
`savedsearches.conf`. See `gallery.xml` (Classic) and `studio_gallery.xml`
(Dashboard Studio) for complete, runnable panels using the bundled
`bikeshare.csv` lookup.

| colorMode | Needs a 4th "by" field? | What colors it | Extra properties |
|---|---|---|---|
| `categorical` (default) | Yes, any type | One solid color per distinct field value | `categoryColors` (optional) |
| `sequential` | Yes, numeric | Smooth gradient across the field's numeric range | `minColor`, `maxColor`, `numOfBins` |
| `threshold` | No — colors `count` itself | Discrete buckets by numeric boundary | `thresholdValues`, `thresholdColors` |
| `field` | Yes, a color string | Uses the field's own value as-is | none — compute the color in SPL |

Hex colors accept a leading `#` or not (`65778A` and `#65778A` both work) in
`categoryColors`, `thresholdColors`, and `field` mode.

**Categorical with custom colors** — one color per distinct value, in the
order the values are first encountered; extra categories fall back to
Splunk's default palette:
```xml
<option name="v2_punchcard_visualisation.punchcard.useColors">true</option>
<option name="v2_punchcard_visualisation.punchcard.colorMode">categorical</option>
<option name="v2_punchcard_visualisation.punchcard.categoryColors">1f77b4,ff7f0e,2ca02c</option>
```

**Threshold** — colors `count` (the same value driving circle size) by
ascending numeric boundary; a value equal to a boundary falls into the
bucket *above* it:
```xml
<option name="v2_punchcard_visualisation.punchcard.useColors">true</option>
<option name="v2_punchcard_visualisation.punchcard.colorMode">threshold</option>
<option name="v2_punchcard_visualisation.punchcard.thresholdValues">10,20</option>
<option name="v2_punchcard_visualisation.punchcard.thresholdColors">65778A,D41F1F,118832</option>
```

**Field value** — the color is whatever your search computes, e.g.:
```spl
... | stats count by x y
| eval colour=case(count<10,"65778A", count<=20,"D41F1F", count>20,"118832")
```
```xml
<option name="v2_punchcard_visualisation.punchcard.useColors">true</option>
<option name="v2_punchcard_visualisation.punchcard.colorMode">field</option>
```
`colour` must be the 4th "by"/table column (same position as the color field
in `categorical`/`sequential`). Accepts any valid CSS color, not just hex —
`rgb()`, `hsl()`, and named colors all work. A blank value falls back to the
default blue.

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

A second warning, `check_for_legacy_custom_visualizations`, is also accepted:
newer Splunk versions recommend migrating custom visualizations to the
Studio-native `studio_visualization` framework (a full rewrite in Splunk's
React/TypeScript SDK). This app keeps the proven legacy
`SplunkVisualizationBase` framework from the original vendor package — and,
per the 1.9.0 testing below, it already works correctly inside Dashboard
Studio dashboards despite being "legacy", so there's no functional gap to
justify that rewrite right now.

## Changelog

- **1.9.0** — Live-tested the whole app in a real Splunk Enterprise 10.6.2.5
  Docker instance (classic dashboards, Dashboard Studio, the Format panel,
  and dark theme) and acted on what that testing surfaced:
  - Format panel now only shows the settings/help text for the currently
    selected `colorMode`, instead of all four modes' sections at once.
    Implemented with a small, self-contained `MutationObserver` in
    `visualization.js` (no Splunk core files touched) that reacts to the
    `colorMode` control's value; falls back gracefully to "show everything"
    if Splunk ever changes that control's structure.
  - X-axis labels no longer overlap illegibly on dense (e.g. 24-hour)
    categorical axes in the default horizontal label mode — now
    auto-switches to angled rendering above 12 categories, regardless of
    the `labelRotation` setting.
  - Added `maxRadius`: the circle-size ceiling (previously hardcoded at
    40px since 1.5.1) is now a formatter property, so it can be tuned per
    panel instead of fixed in code.
  - Added `studio_gallery.xml`: a Dashboard Studio example dashboard
    (confirmed working live) alongside the existing Classic `gallery.xml`,
    showing all four color modes.
  - Clarified in the app description/README that this app's value, now that
    Splunk ships a native built-in Punchcard visualization in Dashboard
    Studio, is specifically its custom color modes.
- **1.8.1** — Docs and examples pass, no behavior change: added a "Color
  modes" reference section to this README, added `gallery.xml` example
  panels for `categoryColors`, `threshold`, and `field` modes (using the
  bundled `bikeshare.csv` lookup, runnable out of the box), and polished the
  formatter panel's help text for all four color modes for clarity and
  consistency.
- **1.8.0** — Added a `field` color mode: the 4th "by" column in the search
  results is used directly, as-is, as each circle's CSS color — compute it
  however you like in SPL (e.g.
  `| eval colour=case(count<10,"#65778A",count<=20,"#D41F1F",count>20,"#118832")`)
  rather than being limited to the fixed `categoryColors`/`thresholdValues`
  config set at the panel level. Blank values fall back to the default blue.
  Also hardened `categoryColors`, `thresholdColors`, and `field` mode to
  accept bare hex codes without a leading `#` (e.g. `65778A`), auto-prepending
  it — matches CSS's own requirement that hex colors start with `#`, which a
  bare code silently fails.
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
