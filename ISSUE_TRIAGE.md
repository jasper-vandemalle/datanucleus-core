# DataNucleus-Core — Open Issue Triage

> **Date**: 2026-03-25
> **Source**: https://github.com/datanucleus/datanucleus-core/issues?q=sort%3Aupdated-desc+is%3Aissue+is%3Aopen
> **Total open issues**: 40

## Overview

All 40 open issues classified by impact (correctness/usability) and estimated effort.
This serves as a reference for tackling issues incrementally across multiple PRs.

---

## 🔴 HIGH IMPACT — Bugs affecting correctness (8 issues)

These are actual bugs where DataNucleus produces incorrect results, throws unexpected
exceptions, or violates JPA/JDO spec behavior.

### Quick wins

| Issue | Title | Effort | Area | Details |
|-------|-------|--------|------|---------|
| [#36](https://github.com/datanucleus/datanucleus-core/issues/36) | `dnNewObjectIdInstance()` wrong for compound PK + persistent properties | Small (~20-30 LOC) | Enhancer | Isolated fix in `NewObjectIdInstance2`. When using JPA properties with compound PK, the enhanced method incorrectly tries `new CompoundPK2((String) key)` instead of extracting individual PK field values via property getters. |
| [#20](https://github.com/datanucleus/datanucleus-core/issues/20) | `CopyOnAttach=false` doesn't handle collection element removal | Small-Med (~50-80 LOC) | FieldManager | `AttachFieldManager` only handles added elements when `copy=false`. Deleted elements in detached collections are silently lost. The code has a TODO acknowledging this gap. |

### Medium effort

| Issue | Title | Effort | Area | Details |
|-------|-------|--------|------|---------|
| [#526](https://github.com/datanucleus/datanucleus-core/issues/526) | Detached Collection reorder + `cache.collections=false` → IndexOutOfBound | Medium (~40-60 LOC) | SCO/Attach | Recent (Jul 2025), has reproducible test case. Fix needed in `SCOUtils.updateListWithListElements()` — index calculations fail when reordering with caching disabled. |
| [#16](https://github.com/datanucleus/datanucleus-core/issues/16) | Embedded PK: incomplete `dnCopyKeyFieldsXXX` methods | Medium (~40-70 LOC) | Enhancer | `@Embeddable` PK classes get empty `jdoCopyKeyFieldsXXX()` method bodies. Breaks DELETE queries that depend on PK extraction. Fix in `CopyKeyFieldsFromObjectId2` and `CopyKeyFieldsToObjectId2`. |
| [#38](https://github.com/datanucleus/datanucleus-core/issues/38) | 1-N bidir: moving elements by setting collection fails | Medium (~50-80 LOC) | Relationships | FK/Join table stores don't properly update when elements are moved to a new owner by setting the collection. Spans `RelationshipManager` + store implementations. |
| [#50](https://github.com/datanucleus/datanucleus-core/issues/50) | Optimistic txn queued operations: cross-collection ordering broken | Medium | Flush | Operations are queued per-collection, not globally. This breaks ordering guarantees across different collections in the same transaction. |

### Larger effort

| Issue | Title | Effort | Area | Details |
|-------|-------|--------|------|---------|
| [#501](https://github.com/datanucleus/datanucleus-core/issues/501) | Multiple FETCH JOINs → "Symbol left already exists" | Med-Large (~80-120 LOC) | JPQL Compiler | JPA spec 4.4.5.3 says FETCH JOINs shouldn't require identification variables. Parser mandates aliases for all JOINs, causing duplicate symbol registration. Touches `JPQLCompiler`, `JPQLSingleStringParser`, `SymbolTable`. |
| [#42](https://github.com/datanucleus/datanucleus-core/issues/42) | Detach graph with Map + overridden `hashcode()` in key fails | Med-Large (~60-100 LOC) | Detach/Maps | When map key has lazy fields and overridden hashcode, detachment fails because `hashcode()` is called before the key is fully detached. Needs two-pass or deferred insertion in `DetachFieldManager.processMapContainer()`. |

---

## 🟡 MEDIUM IMPACT — Enhancements improving real-world usability (11 issues)

### Quick wins

| Issue | Title | Effort | Area | Details |
|-------|-------|--------|------|---------|
| [#52](https://github.com/datanucleus/datanucleus-core/issues/52) | Class loading should use classloader of current class | Small (~1-5 LOC) | Enhancement | Change `Class.forName(className)` to use `getClass().getClassLoader()` in `loadClass` method for `PersistenceCapable` classes. |
| [#143](https://github.com/datanucleus/datanucleus-core/issues/143) | `findObject` should have arg to skip inheritance check | Small | EC | Add boolean parameter to `ExecutionContextImpl.findObject()` for cases where we know the exact class (e.g. relation loading). |
| [#144](https://github.com/datanucleus/datanucleus-core/issues/144) | Metadata extension to ignore class during schema creation | Small | Metadata | Add a class-level metadata extension + check in schema generation to skip specific classes. |
| [#257](https://github.com/datanucleus/datanucleus-core/issues/257) | Check for overriding only getter or setter but not both | Small | Enhancer/Meta | Add validation in metadata processing to catch when a persistent property overrides only the getter or setter, which causes incorrect `dnCopyField`/`dnReplaceField` calls. |

### Medium-to-large effort

| Issue | Title | Effort | Area | Details |
|-------|-------|--------|------|---------|
| [#515](https://github.com/datanucleus/datanucleus-core/issues/515) | L2 cache `maxSize` should evict idle entries | Small-Med | Cache | Currently maxSize just stops adding new entries. Should evict least-recently-used entries instead. |
| [#483](https://github.com/datanucleus/datanucleus-core/issues/483) | Class-level type converters to override superclass field converters | Medium | Metadata/Types | Support JPA `@Convert(attributeName=...)` at class level to override a superclass field's converter. |
| [#466](https://github.com/datanucleus/datanucleus-core/issues/466) | Only wrap mutable fields when accessed | Medium | SCO/State | Performance optimization: reduce unnecessary SCO wrapping when fields are accessed internally (not handed to user). |
| [#455](https://github.com/datanucleus/datanucleus-core/issues/455) | Upgrade `javax.*` to `jakarta.*` (transaction, validation, CDI) | Large | Dependencies | Namespace migration for 3 dependencies. Broad impact across codebase. |
| [#480](https://github.com/datanucleus/datanucleus-core/issues/480) | Drop/isolate multithreaded PM support | Large | Core/EC | Partially implemented feature (only some EC methods handle it, no SCO methods). Either remove or further isolate. JDO spec doesn't mark as optional. |
| [#155](https://github.com/datanucleus/datanucleus-core/issues/155) | Revise enhancement contract for derived identity without identity class | Large | Enhancer | Fundamental change to how `org.datanucleus.identity.IdentityReference` is used. |
| [#94](https://github.com/datanucleus/datanucleus-core/issues/94) | Allow detached objects to have a StateManager | Large | Core/Detach | Would enable lazy-load of unloaded fields while detached (if PMF/EMF still open). Fundamental design change. 13 comments of discussion. |

---

## 🟢 LOW IMPACT — Long-term / niche enhancements (21 issues)

These are nice-to-haves, internal cleanups, or features for uncommon use cases.
Most are labeled "unresourced" by the maintainer.

### Smaller effort

| Issue | Title | Effort | Area |
|-------|-------|--------|------|
| [#44](https://github.com/datanucleus/datanucleus-core/issues/44) | Enable StateManager pooling when multithread is reliable | Small | State |
| [#32](https://github.com/datanucleus/datanucleus-core/issues/32) | List wrapper SCOs: efficient initialise for `setXXXField` | Small-Med | SCO |
| [#26](https://github.com/datanucleus/datanucleus-core/issues/26) | Simple SCO wrappers: delay cascade-delete until flush | Small-Med | SCO/Flush |

### Medium effort

| Issue | Title | Effort | Area |
|-------|-------|--------|------|
| [#376](https://github.com/datanucleus/datanucleus-core/issues/376) | `DNStateManager`: use `Persistable`, drop generics | Medium | State |
| [#429](https://github.com/datanucleus/datanucleus-core/issues/429) | Optimistic Lock Groups | Medium | Transactions |
| [#34](https://github.com/datanucleus/datanucleus-core/issues/34) | FetchPlan: support `field.field`, `field#element.field` syntax | Medium | FetchPlan |
| [#30](https://github.com/datanucleus/datanucleus-core/issues/30) | `CompleteClassTable`: column names for embedded collection elements | Medium | Schema |
| [#35](https://github.com/datanucleus/datanucleus-core/issues/35) | Enhancement: serialisation of `detachedState` with overridden `writeObject` | Medium | Enhancer |
| [#49](https://github.com/datanucleus/datanucleus-core/issues/49) | 1-N subclass-table + compound identity (NPE) | Medium | Metadata/Store |
| [#37](https://github.com/datanucleus/datanucleus-core/issues/37) | Runtime enhancement: cater for relations in persistability | Medium | Enhancer |
| [#40](https://github.com/datanucleus/datanucleus-core/issues/40) | Composite PK auto-generation (compound identity) | Medium | Enhancer |
| [#21](https://github.com/datanucleus/datanucleus-core/issues/21) | Detach wrappers for SCO classes (change tracking while detached) | Medium | SCO/Detach |
| [#17](https://github.com/datanucleus/datanucleus-core/issues/17) | `RelationshipManager`: extend to cover more relation change scenarios | Medium | Relationships |
| [#28](https://github.com/datanucleus/datanucleus-core/issues/28) | Lifecycle transitions for non-transactional datastores | Medium | Lifecycle |

### Large effort

| Issue | Title | Effort | Area |
|-------|-------|--------|------|
| [#23](https://github.com/datanucleus/datanucleus-core/issues/23) | Lazy loading of individual collection elements | Large | SCO/Query |
| [#171](https://github.com/datanucleus/datanucleus-core/issues/171) | Embedded objects with collections of non-embedded persistables | Large | Store/Embedded |
| [#209](https://github.com/datanucleus/datanucleus-core/issues/209) | Enhancement: single PC object per StateManager | Large | Enhancer/State |
| [#33](https://github.com/datanucleus/datanucleus-core/issues/33) | In-memory query evaluation: support variables | Large | Query/InMem |
| [#22](https://github.com/datanucleus/datanucleus-core/issues/22) | In-memory query evaluation: support correlated subqueries | Large | Query/InMem |
| [#31](https://github.com/datanucleus/datanucleus-core/issues/31) | DataFederation: composite persistence-unit spec | Large | Federation |
| [#18](https://github.com/datanucleus/datanucleus-core/issues/18) | DataFederation: definition of federated persistence/retrieval | Large | Federation |

---

## Recommended Approach

Work through issues in phases, one PR per issue (or small group of related issues):

**Phase 1 — Quick bug fixes** (small, isolated, high-value):
`#36`, `#52`, `#20`, `#257`, `#143`, `#144`

**Phase 2 — Medium bug fixes** (need careful testing):
`#526`, `#16`, `#38`, `#515`, `#32`

**Phase 3 — Larger correctness fixes** (cross-cutting):
`#501`, `#42`, `#50`

**Phase 4 — Strategic enhancements** (design decisions required):
`#455`, `#480`, `#466`, `#94`, `#155`

**Backlog** — Low-impact items remain open for opportunistic work.
