# AI-DLC Companion

> **TH:** ตัวช่วยติดตั้งและตัวแสดงผล (installer + read-only viewer) สำหรับ AWS AI-DLC skill — ติดตั้ง skill ให้ถูกตำแหน่งใน `.kiro/skills/` และแสดงข้อมูล workflow จาก `.aidlc/` ในรูปแบบที่อ่านง่าย
>
> **EN:** An installer and read-only viewer for the AWS AI-DLC skill — it installs the skill into the correct `.kiro/skills/` location and visualizes the workflow artifacts under `.aidlc/`.

AI-DLC Companion is a VS Code / Kiro extension (written in TypeScript) that helps you work with the
[AWS AI-DLC (AI-Driven Development Life Cycle) skill](https://github.com/aws-samples/sample-aidlc-decisions-driven-skill).
It does two things:

1. **Installs / locates the AI-DLC skill.** It downloads the skill from the official
   `aws-samples/sample-aidlc-decisions-driven-skill` repository and installs the matching `skills/aidlc*`
   folders into `<workspace>/.kiro/skills/`, so Kiro can always find them. It also detects skills that were
   installed in the wrong place and offers to move them.
2. **Reads and displays `.aidlc/` artifacts.** It parses the `aidlc-manifest.yaml` and the generated
   markdown artifacts and presents them in a **Tree View** and a **Dashboard** (webview).

The extension is a companion only. Creating workflow artifacts and writing the `.md` files remains the job of
the AI-DLC skill (run through the Kiro agent). This extension **does not create or modify** your artifacts, with
one bounded exception: when you answer a decision question it writes your answer into that question's
`**Answer**:` field (see [Decisions](#decisions)). Otherwise its writes are limited to installing/relocating the
skill when you ask it to.

## Features

- **Install / Update Skill** — download the AI-DLC skill from the official repo and install it into
  `.kiro/skills/`. Existing installs are only overwritten after you confirm.
- **Check Installation** — report whether the skill is `installed`, `not installed`, `misplaced`, or `partial`,
  including the version when it can be read.
- **Fix Skill Location** — when a skill is found outside `.kiro/skills/`, move it to the correct location.
- **Workflow Tree View** — browse `feature → phase / unit → artifact` in the AI-DLC activity bar view, with a
  status icon per phase. Clicking an artifact opens the source `.md` / `.yaml` file.
- **Workflow Dashboard** — a webview that summarizes a feature: scope, mode, status, task progress, phase
  states, unit cards, and recorded decisions. Unit cards show rich detail parsed from `units.md` — purpose,
  priority/complexity/type and story badges, and collapsible responsibilities, dependencies and commands —
  which appears even when the feature has no manifest. Markdown artifacts can be rendered inline.
- **Workflow pipeline stepper** — the Dashboard opens with a fixed 7-step AI-DLC lifecycle at the top
  (Context → Requirements → Design → Tasks → Implement → Build & Test → Deploy). Each step is marked
  `completed` (✓), `current` (highlighted), `upcoming` (dimmed), or `unknown`, derived read-only from the
  manifest phase states and task progress. The current step of the main feature is also mirrored on the status
  bar.
- **Dependency graph & build plan** — a read-only dependency DAG of a feature's units, plus a topological build
  order, shown both as a **Dependency Graph** section in the Dashboard and in a dedicated graph panel. Nodes are
  units and edges are "depends on" relationships; the build plan groups units into waves and flags what is ready
  to work versus blocked. Both the graph and the plan can be exported as Mermaid, Graphviz DOT, or Markdown text.
- **Workflow timeline** — a read-only, chronological view of a feature's lifecycle, derived from the timestamps
  already in the model: the manifest's `created` / `updated`, each phase's `timestamp`, and, in incremental mode,
  the per-unit phase timestamps. It appears as a **Timeline** section in the Dashboard and in a dedicated panel,
  with a workspace-wide overview that lays out one lane per feature. Events with no timestamp are grouped as
  *undated* rather than dropped, and the whole timeline can be exported as Markdown text. When the target folder
  is a Git repository, the timeline also overlays **real commit activity as one lane per developer** on the same
  time axis (read-only, no fetch), so a team that spreads AI-DLC units across machines and hands off via Git can
  see who worked when.
- **Architecture diagram viewer** — render the architecture diagrams the AI-DLC skill authored under
  `.aidlc/diagrams/*.json` as an inline SVG, straight from the authored coordinates (no layout engine). Typed
  components, labeled connections, boundary regions, a view switcher, and explanatory cards, with pan / zoom /
  fit-to-view. A **Diagrams** group appears in the Tree View; open it with **AI-DLC: Open Architecture Diagram**.
- **Audit Trail viewer** — render the audit log the AI-DLC skill records at
  `.aidlc/workflow/{feature}/audit.md` as a read-only **vertical timeline** of events. Each entry shows its
  timestamp, a phase badge, the title, the action, its artifacts (clickable to open read-only), and an outcome
  badge (draft / approved / other), with a **client-side phase filter**. An **Audit Trail** node appears under
  each feature in the Tree View; open it with **AI-DLC: เปิด Audit Trail**.
- **Decisions Log** — a read-only aggregated view of *every* decision across all features and
  `decisions-*.md` files in one place. It shows summary counts (total / answered / pending), a **client-side
  filter** (status / phase / feature) and a **search box**, a **mismatch marker** when an answer differs from the
  recommended option, **click-through** to the source decision file, and **Markdown export**. Open it via
  **AI-DLC: เปิด Decisions Log** or the **Decisions Log** node in the Tree View.
- **Team Board** — a read-only view of the authored **Ownership Assignment (Dev A / B / C)** overlay in
  `.aidlc/specs/{feature}/units.md`: **per-dev cards** (owned units / directories / tasks), a **wave timeline
  grid**, and **coordination notes**, with a **client-side dev filter**. A **Team Board** node appears under each
  feature in the Tree View; open it via **AI-DLC: เปิด Team Board**. This is the *planned / authored* assignment —
  distinct from the git-derived ownership shown on unit cards (see [Workflow timeline](#workflow-timeline)).
- **Versions & Tech Stack** — a read-only view derived from a feature's `aidlc-manifest.yaml`: the **resolved
  tech-stack versions** (package → version, with the manifest's resolved-at / source), the **context summary**,
  and the manifest's **decision records** (the R6.5 summary the skill records in the manifest, distinct from the
  Decisions Log's Q&A), with a **client-side search** and **Markdown export**. A **Versions** node appears under
  each feature in the Tree View; open it via **AI-DLC: เปิด Versions & Tech Stack**.
- **Domain Model graph** — a read-only inline-SVG graph derived from each unit's **Domain Model**
  (aggregates / entities / value objects) and **Domain Events** (publish / subscribe) in
  `.aidlc/specs/{feature}/units.md` — one box per unit plus event-choreography edges
  (publisher → subscriber), with pan / zoom / fit-to-view. A **Domain Model** node appears under each feature in
  the Tree View; open it via **AI-DLC: เปิด Domain Model**. It complements the dependency graph (see
  [Dependency graph & build plan](#dependency-graph--build-plan)) by focusing on domain structure and event flow.
- **Project Overview / Command Center** — a single project-level landing page that summarizes *all* features at
  once: a **project totals bar** (active / completed features, task percent, pending decisions, blueprint /
  diagram counts) plus one **feature card** each showing progress, the **current lifecycle step** (a mini 7-step
  pipeline stepper), pending-decision count, unit count, and **quick-link buttons** to whichever views that
  feature has (Dashboard / Audit / Domain / Team / Versions / Graph / Timeline). It has a **client-side search /
  filter** (by name / status) and **Markdown export**. Open it via **AI-DLC: เปิด Project Overview** or the
  **Project Overview** node at the top of the Tree View (also the home button in the view title bar).
- **View decision questions** — the human-in-the-loop questions the AI-DLC skill writes under
  `.aidlc/workflow/{feature}/` show up in the Tree View (a **Decisions** group with a pending badge such as
  `2/5 pending`) and in the Dashboard, so you can spot unanswered questions without hunting through markdown.
- **Answer decisions in the Dashboard** — pick one of a question's options (the recommended one is marked) or
  type your own value for **Other**, and the extension writes it back into that question's `**Answer**:` field.
  It writes only the Answer field, atomically, and refuses the save if the file changed on disk since it was
  loaded (lost-update guard).
- **Auto-refresh on file changes** — the Tree View and Dashboard update automatically (debounced) when files
  under the AI-DLC root change. Can be disabled via settings.

## Getting started

1. Open the folder (workspace) where you want to use AI-DLC.
2. Open the **AI-DLC** view from the activity bar (the AI-DLC icon on the left).
3. If nothing is installed yet, use the welcome buttons or the Command Palette to run
   **AI-DLC: Install / Update Skill**. The skill is downloaded and placed in `.kiro/skills/`.
4. Once the AI-DLC skill has generated artifacts under `.aidlc/`, the Tree View lists your features. Open the
   **Dashboard** for a summary, or click any artifact to open it in the editor.

## Usage

### AI-DLC activity bar view

The extension contributes an **AI-DLC** view container to the activity bar with a **Workflow** tree. When no
data is found yet, the view shows a welcome panel with quick links to install the skill or check the
installation.

### Commands

All commands are available from the Command Palette under the **AI-DLC** category:

| Command | What it does |
|---|---|
| **AI-DLC: Install / Update Skill** | Download and install (or reinstall) the skill into `.kiro/skills/`. |
| **AI-DLC: Check Skill Installation** | Detect and report the current installation status. |
| **AI-DLC: Fix Skill Location** | Move a misplaced skill into the correct `.kiro/skills/` location. |
| **AI-DLC: Open Workflow Dashboard** | Open the Dashboard webview for the selected feature. |
| **AI-DLC: Open Dependency Graph** (`aidlc.openGraph`) | Open the dependency-graph panel for the selected feature. The graph also appears as a **Dependency Graph** section in the Dashboard. |
| **AI-DLC: Export Graph (Mermaid/DOT)** (`aidlc.exportGraph`) | Export the feature's dependency graph as Mermaid or Graphviz DOT text (clipboard + a new untitled document). |
| **AI-DLC: Export Build Plan (Markdown)** (`aidlc.exportBuildPlan`) | Export the feature's build plan (waves, ready/blocked, bottlenecks) as Markdown text (clipboard + a new untitled document). |
| **AI-DLC: Open Workflow Timeline** (`aidlc.openTimeline`) | Open the workflow-timeline panel (a per-feature timeline or a workspace-wide overview). The timeline also appears as a **Timeline** section in the Dashboard. |
| **AI-DLC: Export Timeline (Markdown)** (`aidlc.exportTimeline`) | Export the timeline as a chronological Markdown list (clipboard + a new untitled document). |
| **AI-DLC: รีเฟรช Timeline จาก Remote (git fetch)** (`aidlc.gitFetchRefresh`) | User-initiated `git fetch --all` followed by a refresh of the views. Updates only remote-tracking refs — it never touches the working tree or any file, and the extension never auto-fetches. Also available as a **รีเฟรชจาก remote** button in the Timeline panel. |
| **AI-DLC: Open Architecture Diagram** (`aidlc.openArchitecture`) | Render an architecture diagram authored under `.aidlc/diagrams/*.json` as an inline SVG (QuickPick when several exist). The diagrams also appear in a **Diagrams** group in the Tree View. |
| **AI-DLC: เปิด Audit Trail** (`aidlc.openAudit`) | Render the audit log at `.aidlc/workflow/{feature}/audit.md` as a read-only vertical timeline with a phase filter (QuickPick when several features have one). An **Audit Trail** node also appears under each feature in the Tree View. |
| **AI-DLC: เปิด Decisions Log** (`aidlc.openDecisionLog`) | Open the aggregated **Decisions Log** — a read-only view of every decision across all features/`decisions-*.md` files, with counts, a client-side filter (status/phase/feature) + search, a recommended-mismatch marker, and click-through to the source file. A **Decisions Log** node also appears in the Tree View. |
| **AI-DLC: Export Decisions Log (Markdown)** (`aidlc.exportDecisionLog`) | Export the aggregated decisions log (counts + entries grouped by feature/phase) as Markdown text (clipboard + a new untitled document). Never writes into the AIDLC root. |
| **AI-DLC: เปิด Team Board** (`aidlc.openTeamBoard`) | Render the authored **Ownership Assignment (Dev A/B/C)** section of `.aidlc/specs/{feature}/units.md` as a read-only board — per-dev cards (units/directories/tasks), a wave timeline grid, and coordination notes, with a client-side dev filter (QuickPick when several features have an assignment). A **Team Board** node also appears under each feature in the Tree View. |
| **AI-DLC: เปิด Versions & Tech Stack** (`aidlc.openVersions`) | Open the **Versions & Tech Stack** view for a feature — a read-only presentation of the resolved tech-stack versions (package → version, with resolved-at/source), the context summary, and the manifest's decision records (the R6.5 summary, distinct from the Decisions Log), with a client-side search and click-through to open the source `aidlc-manifest.yaml` (QuickPick when several features have a manifest). A **Versions** node also appears under each feature in the Tree View. |
| **AI-DLC: Export Versions (Markdown)** (`aidlc.exportVersions`) | Export the feature's versions, context summary, and manifest decision records as Markdown text (clipboard + a new untitled document). Never writes into the AIDLC root. |
| **AI-DLC: เปิด Domain Model** (`aidlc.openDomainGraph`) | Render a feature's **Domain Model graph** as an inline SVG — one box per unit (aggregates / entities / value objects) with event-choreography edges (publisher → subscriber) derived from the Domain Model / Domain Events sections of `.aidlc/specs/{feature}/units.md`, with pan / zoom / fit-to-view (QuickPick when several features have a domain model). A **Domain Model** node also appears under each feature in the Tree View. |
| **AI-DLC: เปิด Project Overview** (`aidlc.openProjectOverview`) | Open the **Project Overview / Command Center** — a single project-level landing page summarizing every feature (progress, current lifecycle step, pending decisions, unit count) with a project totals bar, a client-side search / filter (name / status), and quick-link buttons that jump to each feature's Dashboard / Audit / Domain / Team / Versions / Graph / Timeline. A **Project Overview** node also appears at the top of the Tree View, and it is the home button in the view title bar. |
| **AI-DLC: Export Project Overview (Markdown)** (`aidlc.exportProjectOverview`) | Export the project overview (totals + a per-feature summary) as Markdown text (clipboard + a new untitled document). Never writes into the AIDLC root. |
| **AI-DLC: Refresh** | Re-scan the AI-DLC root and refresh the views. |
| **AI-DLC: Select Target Folder** | Choose the target folder in a multi-root workspace. |

The Install, Dashboard, Dependency Graph, Timeline, and Refresh commands are also available as icons in the
Workflow view title bar.
(`AI-DLC: Open Artifact`, `AI-DLC: Open Decision`, and `AI-DLC: Answer Decision` are used internally when you
interact with tree items or the Dashboard and are not shown in the palette.)

### Status bar

The extension contributes two independent status bar items.

**Skill installation** reflects the install state and acts as a shortcut:

- `$(check) AI-DLC` — installed
- `$(warning) AI-DLC misplaced` — a skill was found in the wrong place
- `$(cloud-download) Install AI-DLC` — not installed

Clicking it runs **Check Skill Installation** or **Install / Update Skill** depending on the state.

**Workflow pipeline** shows the current lifecycle step of the main feature, for example
`$(git-commit) AI-DLC: Design`. The main feature is the first `active` feature (or the only feature when there
is just one). Clicking it opens that feature's Dashboard. When there is no workflow data — or several features
with none active — the item is hidden. It updates automatically as the manifest changes, so you do not need to
refresh. See [Workflow pipeline](#workflow-pipeline) for how each step's status is derived.

### Opening artifacts

Click any artifact node in the Tree View (or use the artifact links in the Dashboard) to open the underlying
`.md` / `.yaml` file in the editor. The extension never edits these files — it only reads them.

### Workflow pipeline

The Dashboard opens with a horizontal **pipeline stepper** that maps a feature onto the fixed 7-step AI-DLC
lifecycle:

Context → Requirements → Design → Tasks → Implement → Build & Test → Deploy

Each step shows one of four states, derived best-effort (read-only) from the manifest — the extension never
writes any of this back:

- **completed** (✓) — the phase artifact is `approved`, or, for **Implement**, every task is done.
- **current** (highlighted) — the phase is `in-progress` or `draft`, or **Implement** has partial task
  progress. When the feature is still `active` and nothing is explicitly in progress, the first
  not-yet-completed step becomes current. At most one step is current at a time.
- **upcoming** (dimmed) — a later step, a `not-started` phase, or a step with no artifact yet. **Build & Test**
  and **Deploy** usually have no direct artifact in the manifest, so they stay upcoming until inferred
  otherwise.
- **unknown** — now shown only for a *gap* in the manifest-less mode described below: a step whose artifact is
  missing even though the feature has clearly progressed past it. When a manifest is present, no step falls
  here.

**No readable manifest?** When a feature has no `aidlc-manifest.yaml` (missing, misnamed, or unparseable) the
stepper no longer shows every step as `unknown`. Instead it infers best-effort progress from which artifact
files exist under `specs/{feature}/` — plus the checkbox progress in `tasks.md` for **Implement** — treating
the furthest existing artifact as `current`, the earlier ones as `completed`, and any skipped-over step as an
`unknown` gap. The Dashboard shows a small note next to the stepper so this approximation, and its cause, are
clear; the detailed **Phases** list still reports each phase honestly as `unknown` in this mode.

The current step of the **main feature** — the first `active` feature, or the only feature when there is just
one — is mirrored on the [status bar](#status-bar). The stepper and the status bar item both update
automatically when files under the AI-DLC root change, so they always reflect the latest state without a
manual refresh.

### Dependency graph & build plan

The extension derives a **dependency graph** and a **build plan** from a feature's units — both read-only, both
computed from the scanned model. They show up in two places: a **Dependency Graph** section in the Dashboard, and
a dedicated panel opened with **AI-DLC: Open Dependency Graph** (`aidlc.openGraph`).

In the graph, each **node is a unit** and each **edge is a "depends on" relationship**. Dependencies are resolved
by unit name; a dependency that matches no unit is kept as an **external (dangling)** node rather than dropped,
and **cycles are detected and still render** — they never break or hang the view. Clicking a node reveals that
unit in `units.md`, and hovering a node highlights its upstream dependencies and downstream dependents. A
collapsible topological list is provided as a text fallback for accessibility.

The graph is laid out by the bundled **dagre** engine — a layered layout that reduces edge crossings and gives
clearer ranks, in place of the earlier in-house grid. That previous layered layout is kept as an automatic
**fallback**, so the view is always drawn and never crashes. Edges follow dagre's computed routes as
**polylines** with a direction **arrowhead** instead of straight lines, and every node carries a small
**per-kind shape/icon** — unit / feature / phase / story / command / actor / external — that matches the legend.
The layout stays **read-only** and **deterministic**, and remains **CSP-safe**: dagre is layout-only (it only
computes coordinates — no DOM, no `eval`) and the in-house SVG renderer is unchanged.

The dedicated graph panel adds more views and interactivity, all client-side. A **kind switcher** flips between
five read-only views of the same feature — **Dependencies** (the unit dependency DAG), **Feature Overview**
(feature → unit → phase), **Stories** (which units cover which stories), **Commands/Actors** (which actor
invokes which command), and **Build Plan** (the wave-by-wave build order, described below) — all precomputed, so
switching is instant. For the graph views you can **pan** by dragging the canvas, **zoom** with the mouse wheel
or the `＋ / − / ⟲` buttons, and use **พอดีจอ (fit-to-view, ⤢)** to scale and center the whole graph in one click
(the `⟲` reset now fits as well). The toolbar groups the kind switcher with the zoom controls, and the
**legend** names the node and edge kinds in view — including the per-kind node shapes.
**Bottleneck** units (high fan-in) and any nodes/edges caught in a **dependency cycle** are highlighted so they
stand out.

The **build plan** turns the same graph into a topological build order: it groups units into **waves** (each wave
can be worked in parallel), separates units that are **ready to work** from those still **blocked** by unfinished
dependencies, and highlights **bottlenecks** — the units many others depend on. The **Build Plan** view (the fifth
switcher option) lays the waves out in order and overlays each unit with a small badge showing its state —
**done** (✓), **ready**, **blocked**, or **bottleneck** (a unit can carry several, e.g. *ready* + *bottleneck*).
The same overlay is shown on the Dashboard's topological wave list, and the Markdown export annotates each unit
in its wave with those same tags (e.g. `- hub — ready, bottleneck`).

Everything here is read-only: it is derived from the model, and the extension never writes any of it back to the
AI-DLC root. Exporting is text-only as well — **AI-DLC: Export Graph (Mermaid/DOT)** (`aidlc.exportGraph`) and
**AI-DLC: Export Build Plan (Markdown)** (`aidlc.exportBuildPlan`) copy the result to the clipboard *and* open it
as an untitled editor document for you to save wherever you like. No files are written into the AI-DLC root.

### Workflow timeline

The extension derives a **workflow timeline** — a chronological view of a feature's lifecycle — from the
timestamps already present in the model: the manifest's `created` / `updated`, each phase's `timestamp` (with its
status), and, in incremental mode, the per-unit phase timestamps. It is read-only and computed from the scanned
model; the extension never writes any of it back to the AI-DLC root.

It shows up in two places: a **Timeline** section in the Dashboard, and a dedicated panel opened with
**AI-DLC: Open Workflow Timeline** (`aidlc.openTimeline`). The panel can show a **per-feature** timeline or a
**workspace-wide overview** that lays out one lane per feature on a shared time axis, so you can compare how
features progressed side by side. In incremental mode, each unit gets its own lane.

Each event is a point on the time axis, ordered oldest to newest. **Click an event** to open (or reveal) its
underlying artifact, and **hover** it for a tooltip naming the phase/unit, its status, and the time. A text
fallback lists every event chronologically — with a separate section for **undated** events (those with no
timestamp, which are grouped rather than dropped) — so the timeline stays usable without the SVG and remains
accessible. When a feature has no dated events, a friendly empty state is shown instead of a blank axis.

You can export the timeline with **AI-DLC: Export Timeline (Markdown)** (`aidlc.exportTimeline`): it produces a
chronological Markdown list, copies it to the clipboard *and* opens it as an untitled editor document for you to
save wherever you like. Nothing is written into the AI-DLC root. Like the tree, pipeline, and graph, the timeline
updates automatically — through the same refresh path — when files under the AI-DLC root change, so it always
reflects the latest state without a manual refresh, and it stays strictly read-only.

**Git-aware timeline (Phase 1).** When the target folder is a Git repository, the timeline enriches itself with
real commit activity. The commit history is read **bounded and read-only** through the `git` CLI — the extension
never fetches and never writes, it only reads what is already in the local repository — and the commits are
grouped into **one lane per developer (author)** alongside the manifest phase lanes on the same time axis.
Hovering a commit shows its author • short hash • date • subject; commits also appear in the text fallback (author
name only, never the email). This layer is **fail-soft**: if `git` is unavailable, the folder is not a repository,
or reading fails, the timeline quietly stays manifest-only and shows a small note that Git data is unavailable.
You can turn the layer off entirely with `aidlc.git.enabled` (see [Settings](#settings)).

**Unit attribution & ownership (Phase 2).** When commits can be linked to a Unit of Work, the extension shows each
unit's **owner** (the top committer) and its **contributors** (each with a commit count) on the Dashboard unit cards
and in the Build Plan, so you can see who owns and works each unit across a distributed team. The link is best-effort,
made through configurable signals — the commit's **branch / source ref**, a **`[unit:x]` token** in the commit
message, or a **story id** (e.g. `US-001`) mapped to a unit through that unit's stories — tried in that order, with
the first match winning and at most one unit per commit. Commits that match no signal are counted as *not linked to a
unit* (fail-soft) and surfaced as a small unmatched note rather than dropped. Like the rest of the timeline this layer
is read-only and shows the author name only (never the email); tune the three signals with `aidlc.git.branchPattern`,
`aidlc.git.unitTokenPattern`, and `aidlc.git.storyIdPattern` (see [Settings](#settings)).

**Handoffs, stale units, filtering & remote refresh (Phase 3).** A few more team-oriented signals build on the Git
layer above, all still read-only and derived by pure, deterministic functions:

- **Merge & handoff markers.** A commit with two or more parents (a **merge**) is drawn with its own marker instead
  of a plain commit dot, and its tooltip says it is a merge. When that merge is attributed to a unit (through the same
  signals as Phase 2), it is additionally flagged as a **handoff** for that unit — a unit's branch folding back into the
  main line — with a stronger marker and a tooltip that names the handoff.
- **Stale units.** Unfinished units that have gone quiet are badged as **stale**, with an age in days, on the Dashboard
  unit cards and in the Build Plan. A unit is stale when it is not yet finished *and* either has no commits linked to it
  or its most recent related commit is older than a threshold. The threshold is the **`aidlc.git.staleDays`** setting
  (default **14**); the check is deterministic — the current time is passed in rather than read inside the logic.
- **Timeline filter.** The standalone Timeline panel has a **client-side filter** — checkbox groups for **Developers**
  and **Units** — that shows or hides lanes and events instantly, entirely in the webview with no round-trip to the
  extension (CSP-safe, no inline handlers).
- **Refresh from remote.** A **รีเฟรชจาก remote** button in the Timeline panel — and the
  **AI-DLC: รีเฟรช Timeline จาก Remote (git fetch)** command (`aidlc.gitFetchRefresh`) — runs `git fetch --all` and then
  refreshes the views, so you can pull in teammates' commits from other machines before the timeline correlates them.
  This is the only network operation the extension makes, and it is **user-initiated only — the extension never
  auto-fetches**. `git fetch` updates only remote-tracking refs; it does **not** modify the working tree or any file,
  keeping this consistent with the read-only / fail-soft principles described for Phase 1 and Phase 2.

### Architecture diagram viewer

The AI-DLC skill can author **architecture diagrams** under `.aidlc/diagrams/*.json` — a JSON schema that
already carries each component's position and size. The extension picks these up (a **Diagrams** group appears in
the Tree View, and **AI-DLC: Open Architecture Diagram** / `aidlc.openArchitecture` opens one; a QuickPick lets
you choose when there is more than one) and draws the diagram as an **inline SVG straight from the authored
coordinates — no layout engine is involved**. You get:

- **Typed components** — each `type` (frontend / backend / cloud / external / security / …) is drawn with its own
  style using theme tokens, with its `label`, `sublabel`, and `tag`.
- **Labeled connections** — lines follow the authored `from` / `to` (and sides), with a label, a direction
  arrowhead, and a style per `variant` (e.g. emphasis / dashed / security).
- **Boundary regions** — drawn behind the components they wrap.
- **A view switcher** — the diagram's named views each focus (highlight) a subset of components and dim the rest,
  with the view's note shown alongside.
- **Explanatory cards** — the authored `cards` are shown next to the diagram.
- **Pan / zoom / fit-to-view** — drag to pan, wheel or the `＋ / − / ⟲` buttons to zoom, and fit-to-view to scale
  and center the whole diagram in one click, just like the graph panel.

You can also open the **source files**: the `.json` opens in the editor, and the authored `.html` opens
**externally in your browser**. The HTML is deliberately **not** loaded into the extension's webview — it pulls in
external fonts and scripts, so keeping it out preserves the strict CSP. Everything here is **read-only** (the
extension only reads `.aidlc/diagrams/`, never writes back) and **CSP-safe** (inline SVG built with
`createElementNS`, a nonce, and `localResourceRoots` — no `innerHTML` on diagram data, no external content). The
diagram location is configurable via the **`aidlc.diagramsGlob`** setting (see [Settings](#settings)), and the
view updates through the same refresh path as the rest of the extension when files under the AI-DLC root change.

### Audit Trail viewer

The AI-DLC skill records an **audit log** of a feature's lifecycle at `.aidlc/workflow/{feature}/audit.md`. The
extension picks it up (an **Audit Trail** node appears under each feature in the Tree View, and
**AI-DLC: เปิด Audit Trail** / `aidlc.openAudit` opens it; a QuickPick lets you choose when more than one feature
has an audit log) and renders it as a read-only **vertical timeline** of events, oldest to newest. You get:

- **One entry per event** — each shows its **timestamp**, a **phase badge**, the **title**, and the **action**.
  Events with no (or an unparseable) timestamp are kept as *undated* and grouped at the end rather than dropped.
- **Artifacts** — the artifacts recorded on an entry are listed and **clickable**; clicking one opens the
  underlying file in the editor (read-only, through `aidlc.openArtifact`).
- **An outcome badge** — each entry's outcome is shown as a badge coloured by kind: **draft**, **approved**, or
  **other**.
- **A client-side phase filter** — show or hide entries by phase instantly, entirely in the webview with no
  round-trip to the extension.

You can also open the **source file**: `audit.md` opens in the editor. Everything here is **read-only** (the
extension only reads `.aidlc/workflow/{feature}/audit.md`, never writes back) and **CSP-safe** (the timeline is
built with `createElementNS` / `textContent`, a nonce, and `localResourceRoots` — no `innerHTML` on audit data,
and no external HTML is loaded into the webview). The audit location is configurable via the
**`aidlc.auditGlob`** setting (see [Settings](#settings)), and the view updates through the same refresh path as
the rest of the extension when files under the AI-DLC root change.

### Decisions Log

The **Decisions Log** gathers *every* decision question — across all features and every `decisions-*.md` file —
into a single read-only view. It **aggregates the decisions already parsed** by the Decisions feature (R11):
nothing is parsed again, and there is **no new glob or setting** (it reuses `aidlc.decisionsGlob`). Open it with
**AI-DLC: เปิด Decisions Log** (`aidlc.openDecisionLog`) or the **Decisions Log** node in the Tree View. You get:

- **Summary counts** — the totals across the whole workspace (total / answered / pending), plus a per-feature
  breakdown.
- **Filter + search** — a **client-side filter** by **status**, **phase**, and **feature**, and a **search box**
  that matches the title / question / answer. Toggling a filter or typing runs entirely in the webview with no
  round-trip to the extension.
- **Mismatch marker** — when an answered question's value differs from the option marked *recommended*, the entry
  carries a small marker so divergences from the recommendation stand out.
- **Click-through** — clicking an entry opens its **source decision file at the answer line** (through
  `aidlc.openDecision`), the same navigation used by the Decisions tree group.
- **Markdown export** — **AI-DLC: Export Decisions Log (Markdown)** (`aidlc.exportDecisionLog`) produces a
  Markdown summary (counts + entries grouped by feature / phase), copies it to the clipboard *and* opens it as an
  untitled editor document. It **never writes into the AIDLC root**.

Everything here is **read-only** (the log is derived from the scanned model; the extension never writes it back)
and **CSP-safe** (the list is built with `createElementNS` / `textContent`, a nonce, and `localResourceRoots` —
no `innerHTML` on decision data, and no external HTML is loaded into the webview). The view updates through the
same refresh path as the rest of the extension when files under the AI-DLC root change.

### Team Board

The AI-DLC skill can author an **Ownership Assignment (Dev A / B / C)** overlay inside a feature's
`.aidlc/specs/{feature}/units.md`. The **Team Board** reads that section and presents it as a read-only view of
the *planned* team split. A **Team Board** node appears under each feature that has an assignment in the Tree
View, and **AI-DLC: เปิด Team Board** (`aidlc.openTeamBoard`) opens it (a QuickPick lets you choose when more than
one feature has an assignment). You get:

- **Per-dev cards** — the members table is parsed into one card per developer, each listing the **units**,
  **directories**, and **tasks** that developer owns.
- **A wave timeline grid** — the *Timeline by wave* is laid out as a grid (one row per wave, one column per
  developer) so you can see who does what in each wave at a glance.
- **Coordination notes** — the section's coordination notes are listed (markers trimmed) beneath the grid.
- **A client-side dev filter** — highlight or hide developers instantly, entirely in the webview with no
  round-trip to the extension.

You can also open the **source file**: `units.md` opens in the editor (read-only, through `aidlc.openArtifact`).
Everything here is **read-only** (the extension only reads `units.md`, never writes back) and **CSP-safe** (the
board is built with `createElementNS` / `textContent`, a nonce, and `localResourceRoots` — no `innerHTML` on the
board data, and no external HTML is loaded into the webview). The units location is configurable via the
**`aidlc.unitsGlob`** setting (see [Settings](#settings)), and the view updates through the same refresh path as
the rest of the extension when files under the AI-DLC root change. Note this reflects the **authored plan** — the
assignment the skill wrote into `units.md` — which is distinct from the **git-derived ownership** (top committer /
contributors) surfaced on the unit cards and Build Plan by the Git-aware timeline (R17).

### Versions & Tech Stack

The AI-DLC skill records the **resolved tech stack** and a **decisions summary** inside a feature's
`aidlc-manifest.yaml`. The **Versions & Tech Stack** view derives a read-only presentation of that data — nothing
is parsed anew beyond the already-scanned manifest, so it needs **no new setting**. A **Versions** node appears
under each feature that has a manifest in the Tree View, and **AI-DLC: เปิด Versions & Tech Stack**
(`aidlc.openVersions`) opens it (a QuickPick lets you choose when more than one feature has a manifest). You get:

- **A versions table** — the manifest's resolved packages laid out as **package → version**, sorted by package
  name, with the manifest's **resolved-at** timestamp and **source** shown as a heading.
- **A context summary** — the manifest's context fields presented as a card (key → value; arrays / objects are
  rendered readably).
- **Decision records** — the manifest's `decisions` summary grouped by phase (phase → key → value). Note these
  come **from the manifest** (the R6.5 summary the skill records) and are **distinct from the Decisions Log**
  (R22), which aggregates the human-in-the-loop Q&A from `decisions-*.md` files.
- **A client-side search** — a search box filters the versions, context, and decision records instantly, entirely
  in the webview with no round-trip to the extension.

You can also open the **source file**: the feature's `aidlc-manifest.yaml` opens in the editor (read-only,
through `aidlc.openArtifact`). Export with **AI-DLC: Export Versions (Markdown)** (`aidlc.exportVersions`): it
produces a Markdown summary (versions table + context summary + decision records), copies it to the clipboard
*and* opens it as an untitled editor document — it **never writes into the AIDLC root**. Everything here is
**read-only** (the view is derived from the scanned manifest; the extension never writes it back) and
**CSP-safe** (the tables and cards are built with `createElementNS` / `textContent`, a nonce, and
`localResourceRoots` — no `innerHTML` on manifest data, and no external HTML is loaded into the webview). The
view updates through the same refresh path as the rest of the extension when files under the AI-DLC root change.

### Domain Model graph

The AI-DLC skill can describe each unit's **Domain Model** and **Domain Events** inside a feature's
`.aidlc/specs/{feature}/units.md`. The **Domain Model graph** reads those sections and draws them as a read-only
**inline SVG** — a structural, event-flow view that **complements the dependency graph** (see
[Dependency graph & build plan](#dependency-graph--build-plan)): where that graph shows *depends-on*
relationships, this one focuses on **domain structure and event choreography**. A **Domain Model** node appears
under each feature that has a domain model in the Tree View, and **AI-DLC: เปิด Domain Model**
(`aidlc.openDomainGraph`) opens it (a QuickPick lets you choose when more than one feature has one). You get:

- **One box per unit** — each unit is drawn as a box titled with the unit name, listing its **aggregates** (with
  the aggregate **root** marked), **entities**, and **value objects** grouped separately.
- **Event edges** — an edge is drawn from a **publisher** unit to a **subscriber** unit whenever they share an
  event name (publisher → subscriber), labelled with the event name and carrying a direction arrowhead. A
  **dangling** event — one that is published with no subscriber, or subscribed with no publisher — is drawn
  distinctly rather than dropped.
- **Pan / zoom / fit-to-view** — drag to pan, wheel or the `＋ / − / ⟲` buttons to zoom, and fit-to-view to scale
  and center the whole graph in one click, just like the dependency and architecture graphs.

You can also open the **source file**: `units.md` opens in the editor (read-only, through `aidlc.openArtifact`).
This view **reuses the `aidlc.unitsGlob` setting** (see [Settings](#settings)) — the same glob that feeds the
Team Board — so there is **no new setting**. Everything here is **read-only** (the extension only reads
`units.md`, never writes back) and **CSP-safe** (the SVG is built with `createElementNS` / `textContent`, a
nonce, and `localResourceRoots` — no `innerHTML` on the domain data, and no external HTML is loaded into the
webview). The view updates through the same refresh path as the rest of the extension when files under the
AI-DLC root change.

### Project Overview / Command Center

Where every other view focuses on a single feature, the **Project Overview** is a project-level **landing page /
command center**: it aggregates *all* of your features into one screen so you can see the whole project at a
glance and jump straight to whatever needs attention. It is derived entirely from the **already-scanned** workflow
data — nothing is parsed anew — so it needs **no new setting**. A **Project Overview** node appears at the top of
the Tree View, and **AI-DLC: เปิด Project Overview** (`aidlc.openProjectOverview`) opens it (it is also the home
button placed first in the Workflow view title bar). You get:

- **A project totals bar** — across all features: how many are **active** vs **completed**, the overall **task
  percent**, the total **pending decisions**, and the **blueprint / diagram** counts.
- **One card per feature** — each card shows the feature name, status, scope, and mode; a **mini 7-step pipeline
  stepper** marking the **current lifecycle step**; a **progress bar**; a **decision badge** (`pending / total`);
  and the **unit count**.
- **Quick-link buttons** — each card carries buttons that jump directly to the views that feature actually has:
  **Dashboard** always, plus **Audit**, **Domain**, **Team**, **Versions**, **Graph**, and **Timeline** when
  available. Clicking a card opens its Dashboard.
- **A client-side search / filter** — filter the cards by **name** or **status** instantly, entirely in the
  webview with no round-trip to the extension.

Export with **AI-DLC: Export Project Overview (Markdown)** (`aidlc.exportProjectOverview`): it produces a Markdown
summary (project totals + a per-feature list), copies it to the clipboard *and* opens it as an untitled editor
document — it **never writes into the AIDLC root**. Everything here is **read-only** (the overview is derived from
the scanned model; the extension never writes anything back) and **CSP-safe** (the totals bar and cards are built
with `createElement` / `textContent`, a nonce, and `localResourceRoots` — no `innerHTML` on the overview data,
and no external HTML is loaded into the webview). When there are no features yet, the view shows an **empty
state**, and it updates through the same refresh path as the rest of the extension when files under the AI-DLC
root change.

### Decisions

The AI-DLC skill pauses to ask you questions (human-in-the-loop), writing them into decision files under
`.aidlc/workflow/{feature}/`. To work through them:

- Open the **Dashboard**, find a pending question, choose an option (or type a value for **Other**), and save.
  The answer is written back into that question's `**Answer**:` field.
- Or click a decision in the **Decisions** group of the Tree View to open the source file and jump to its
  answer line.

Answered questions show their current answer and can be edited after you confirm. Because the extension only
ever writes the `**Answer**:` field, the rest of the file is left untouched.

## Settings

Configure the extension under **Settings → Extensions → AI-DLC Companion** (or in `settings.json`). All paths
are relative to the target workspace folder.

| Setting | Type | Default | Meaning |
|---|---|---|---|
| `aidlc.rootPath` | string | `.aidlc` | Root folder that holds the AI-DLC artifacts. Change it to support layouts such as `.ai-dlc`. |
| `aidlc.skillsPath` | string | `.kiro/skills` | Location where the skill is installed. |
| `aidlc.repositoryUrl` | string | `https://github.com/aws-samples/sample-aidlc-decisions-driven-skill` | Source repository for the skill. |
| `aidlc.repositoryRef` | string | `main` | Branch or tag to download from the source repository. |
| `aidlc.autoRefresh` | boolean | `true` | Automatically refresh the views when files under the AI-DLC root change. |
| `aidlc.decisionsGlob` | string | `workflow/*/*decision*.md` | Glob (relative to the AI-DLC root) used to find decision files under `workflow/{feature}/`. Adjust it if the skill's decision filenames differ. |
| `aidlc.diagramsGlob` | string | `diagrams/*.json` | Glob (relative to the AI-DLC root) used to find the architecture diagram JSON files the skill authored. Adjust it if the skill writes diagrams elsewhere. |
| `aidlc.auditGlob` | string | `workflow/*/audit.md` | Glob (relative to the AI-DLC root) used to find the audit-trail markdown files the skill records under `workflow/{feature}/`. Adjust it if the skill writes the audit log elsewhere. |
| `aidlc.unitsGlob` | string | `specs/*/units.md` | Glob (relative to the AI-DLC root) used to find the `units.md` files whose **Ownership Assignment (Dev A/B/C)** section feeds the Team Board under `specs/{feature}/`. Adjust it if the skill writes units elsewhere. |
| `aidlc.git.enabled` | boolean | `true` | Read Git history to enrich the timeline with per-developer commit activity (read-only, no fetch). Turn it off for a manifest-only timeline. |
| `aidlc.git.maxCommits` | number | `500` | Upper bound on how many commits are read, so large repositories stay responsive. |
| `aidlc.git.branchPattern` | string | `(?:feat\|feature\|unit)[\/]([^\/]+)` | Regex matched against a commit's branch / source ref to capture a unit name (capture group 1) — the first signal of the unit-attribution resolver. |
| `aidlc.git.unitTokenPattern` | string | `\[unit:\s*([^\]]+)\]` | Regex matched against the commit message to capture a unit name from an inline token such as `[unit:x]` (capture group 1) — the second signal. |
| `aidlc.git.storyIdPattern` | string | `US-\d+` | Regex for story ids in commit messages (e.g. `US-001`), mapped to a unit through that unit's `units.md` stories — the third signal. |
| `aidlc.git.staleDays` | number | `14` | Number of days after which an **unfinished** unit with no newer linked commit is flagged as **stale** (with its age) on the Dashboard unit cards and the Build Plan. |

Changes take effect on the next operation — no reinstall or window reload required.

## Requirements

- **VS Code / Kiro `^1.85.0`.** The extension uses the standard VS Code Extension API and runs in both Kiro and
  VS Code.
- **Internet access** is required for the install/update command, which downloads the skill archive over HTTPS
  from the configured repository. Reading and displaying `.aidlc/` artifacts works offline.

## Development

```bash
# install dependencies
npm install

# build the extension bundle (dist/extension.js) with esbuild
npm run build

# rebuild on change
npm run watch

# production build (minified) — used before packaging
npm run package

# type-check without emitting
npm run typecheck
```

Press **F5** in VS Code to launch the **Extension Development Host** with the extension loaded.

### Tests

```bash
# unit tests (mocha)
npm run test:unit

# integration tests (@vscode/test-electron)
npm run test:integration
```

### Packaging

The runtime code and dependencies are bundled into `dist/extension.js` by esbuild, so packaging does not need
`node_modules`:

```bash
npx @vscode/vsce package --no-dependencies
```

This produces a `.vsix` file you can install via **Extensions: Install from VSIX…**. The `.vscodeignore` keeps
the package limited to the runtime bundle and assets (`dist/`, `media/`, `resources/`, `package.json`,
`README.md`).

## License

MIT License — Copyright (c) 2025 eCloudvalley Thailand.

Developed by Arthit Aunkeaw.
