# CLAUDE.md — summon

Shared concepts live in the [root CLAUDE.md](../../CLAUDE.md) and [docs/vocabulary.md](../../docs/vocabulary.md); this file only states what is specific to the summon chart.

## 1. Role

Summon is the default, CRD-free way to deploy containers in runik. It ships as the `defaultTrinket` and renders only core Kubernetes resources: Deployment / StatefulSet / Job / CronJob / DaemonSet, plus Service, ServiceAccount, ConfigMap, Secret, PV / PVC, HPA, and RBAC. When a spell needs CRD-based workload primitives (Argo Rollouts, PodMonitors, and similar) the [microspell](../trinkets/microspell/) trinket extends summon with those — summon itself stays CRD-free by design.

## 2. Field reference

Every spell field summon accepts is documented in [docs/usage/summon.md](../../docs/usage/summon.md). This CLAUDE.md intentionally does not duplicate that reference.

## 3. Internal glyph dispatcher

- **Entry**: `charts/summon/templates/summon.yaml` — the `range $root.Subcharts` loop.
- **Trigger**: a top-level spell key matches a bundled subchart name AND the value under it is a map whose entries each have a `.type` field.
- **Effect**: summon includes the template named `<subchart>.<type>`, passing `(list $root $glyphWithName)` — the same calling convention kaster uses, so one glyph template works in both dispatchers.
- Unknown top-level keys (no matching subchart) are silently ignored.

## 4. Bundled subcharts

`charts/summon/charts/` is a git worktree of `glyphs.git`; contents match `charts/kaster/charts/` exactly.

Dispatchable subcharts (14):

```
argo-events · aws · certManager · crossplane · external-secrets ·
freeForm · gcp · istio · keycloak · pinniped · postgresql · s3 · vault ·
workflow
```

Plus helpers that are not dispatched: `common`, `runic-system`, `summon`.

Edit rule: edit only in the canonical path `charts/glyphs/`, then bump the submodule reference in `charts/summon/`. Per-glyph fields: [docs/usage/glyphs.md](../../docs/usage/glyphs.md).

## 5. The contentType system

Single convention summon uses for `configMaps` and `secrets` to decide how the data reaches the pod.

| Value | Effect |
|-------|--------|
| `env` | Keys surfaced via `envFrom` |
| `yaml` / `json` / `toml` | Serialized single file |
| `file` | Raw content as a single file (default) |

Implementation: `charts/summon/charts/summon/templates/storage/{configMaps,secrets,_checksums}.tpl`.

## 6. Relationship to microspell

`microspell/` bundles summon as a subchart and calls summon's templates (`summon.workload.*`, `summon.configMap`, ...) from its own `templates/base.yaml`, then layers native CRD resources on top in `templates/microservice.yaml`. If a workload needs those CRDs, switch the spell to microspell instead of summon — fields that summon already understands flow through unchanged. Details: [trinkets/microspell/CLAUDE.md](../trinkets/microspell/CLAUDE.md).

## 7. Testing

Examples: `charts/summon/examples/` (one YAML per pattern or edge case). Run via Make:

```
make render   summon [file]
make snapshot summon [file]
make test     summon [file]
```

## 8. Related docs

- [Root CLAUDE.md](../../CLAUDE.md) — framework-wide concepts.
- [librarian/CLAUDE.md](../../librarian/CLAUDE.md) — what reaches summon.
- [docs/usage/summon.md](../../docs/usage/summon.md) — field reference.
- [docs/usage/glyphs.md](../../docs/usage/glyphs.md) — per-glyph fields.
- [docs/design/summon-internals.md](../../docs/design/summon-internals.md) — code walkthrough.
