# Handoff: Subhex Editor Plan

Date: 2026-06-17  
Merged from branch: `subhex-detail-20260610`  
Workspace: `F:\Codex\campaign-codex`

## Current State

The renderer/performance pass is in a good place again.

Shipped direction now includes:

- normal map zoom stops extended through subhex detail: `0.16`, `0.25`, `0.5`, `0.85`, `1.25`, `2`, `3`, `4`, `5`
- parent editor modes still capped at `1.25`
- subhex detail remains read-only in normal map mode
- subhex entry hitch was reduced mainly by yielding background precache and making high-zoom wheel stepping more responsive
- subhex POI badge sizing in detail view was tuned down from parent-scale presentation
- farm overlays now appear across owned subhexes instead of disappearing in detail view
- map-edge subhex rendering is clipped back to the parent-map boundary instead of spilling onto parchment

Important: the current codebase also contains a prototype subhex editor shell. Treat it as scaffolding, not final product direction.

## Product Direction

The desired subhex editor should **not** feel like a detached popup tool.

Preferred interaction:

1. user enters subhex editing from the existing editor flow
2. selected parent hex becomes the active editing subject
3. map view zooms/focuses into that hex
4. surrounding map remains faintly visible behind a strong veil
5. left editor pane stays in the familiar editor layout language

The user specifically preferred this over a self-contained modal because it keeps the experience tied to the actual map and avoids feeling like a separate toy viewer.

## Entry Points

Planned entry paths:

- inspect popup inside editor gets an `Edit` button for subhex editing
- optionally later, a full `Sub-Hex` mode can sit alongside Surveyor / Cartographer if that proves cleaner

Current preference: start from the inspect-popup editor entry first.

## Editor Presentation

Desired presentation rules:

- selected parent hex is centered in the usable viewport area, biased to account for the left pane
- veil should be heavy, around `85%` opaque gray, with neighboring hexes still faintly visible
- only the chosen parent hex is editable
- experience should feel like an editor state layered over the real map, not a separate document

## Editing Scope

V1 subhex editor should stay lightweight and staged.

Editable in first pass:

- owned subhex terrain
- owned subhex features
- POI local anchor placement

Not full freeform in first pass:

- no unrestricted redraw of roads/rivers/paths
- no deleting route continuity at edges
- no editing neighboring parent hexes from the same session

## Ownership Rule

Owned editable subhexes are the cells whose centers fall inside the parent hex boundary.

Context / partial border cells:

- should not be editable in v1
- do not need to be shown as editable cells
- should not be relied on for persistence

This means continuity with neighbors must be handled through shared boundary anchors/portals, not by letting the editor paint half-cells outside the parent.

## Route Plan

Routes should stay parent-owned records.

V1 route behavior:

- preserve parent route continuity at the boundary
- allow future internal adjustment points inside the parent
- keep boundary enter/exit anchors locked

For mountain handling, preferred direction is:

- stop relying on mountain-pass permutation visuals
- in mountain-heavy parents, roads can degrade to path-like routing
- local routing should try to avoid mountain/cliff subhexes where possible

Crossing rule that still matters:

- POIs tagged as crossings should resolve onto the actual river-crossing subhex
- route projection should visibly respect that shared crossing anchor

## POI Plan

POIs remain tied to the parent hex as their world record.

Subhex behavior:

- every POI should resolve to an effective subhex anchor even if the parent is still procedural
- generated default anchor should be deterministic and center-biased
- multiple POIs in one parent should spread deterministically around the center
- manual anchor should override generated anchor

Rendering rule:

- preserve existing stable parent-view POI behavior at normal map zooms
- subhex POI rendering is its own presentation path and should not destabilize far-zoom behavior again

## Persistence Direction

Preferred persistence remains the hybrid model from `subhex-map-spec.md`:

- parent hex stays authoritative
- no separate `32 rows per parent hex` table for v1
- add nullable `subhex_snapshot jsonb` on the parent hex row for owned terrain/feature materialization
- later add local POI anchor data on POI records
- later add route adjustment data on overlay records

Key state model:

- `Generated`: fully procedural
- `Anchored`: procedural terrain/features, but saved POI / route-local placement
- `Materialized`: owned subhex terrain/features explicitly saved

## Current Code Notes

Things already present that can be reused carefully:

- multi-stop zoom path into subhex detail
- subhex detail tile caching and route/POI projection groundwork
- popup `Edit` plumbing in `js/map-ui.js`
- prototype subhex editor shell styling / renderer hooks

But:

- do not assume the current shell is the final UI
- do not deepen the modal-shell approach without revisiting the in-map editor presentation

## Recommended Next Build Order

1. Convert the current prototype from popup-shell thinking toward an in-map subhex editor state.
2. Reuse the inspect-popup `Edit` entry.
3. Focus/center the selected parent hex with left-pane-aware framing.
4. Apply the strong veil and isolate only owned editable cells.
5. Implement staged terrain/feature painting for owned subhexes.
6. Add POI anchor editing.
7. Add locked boundary route portal display.
8. Add internal route adjustment only after the terrain/feature/POI flow feels solid.

## Guardrails

- Preserve current stable normal-map performance first.
- Do not reintroduce parent-view POI regressions while working on subhex editor rendering.
- Do not let subhex editor work leak into ordinary map navigation cost.
- Do not treat the prototype modal shell as architecture that must be preserved.
- Keep the editor staged, like Cartographer, rather than immediate live-write behavior.

## Related Docs

- `subhex-map-spec.md`
- `log/handoff-subhex-performance-20260605.md`
