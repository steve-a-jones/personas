# Tech Spec System Standard

Purpose: define a consistent, reusable structure so Avery and other personas can locate and understand any spec at any time.

## Spec Home
Default spec root is `src/specs/` at repo top level. If a repo uses a different root, declare it at the top of `src/specs/INDEX.md` and keep the same internal layout.

## Directory Layout (per spec)
Each spec lives in its own folder named after the spec ID:

src/specs/
  INDEX.md
  <SPEC-ID>/
    spec.md
    progress.md
    journal/
      YYYYMMDD-HHMM-<slug>.md
    reviews/
      project-manager-v<version>.md
      product-experimentation-architect-v<version>.md
      tech-spec-director-v<version>.md
      software-engineer-v<version>.md
      staff-ui-ux-designer-v<version>.md
      staff-engineer-in-test-v<version>.md
    artifacts/ (optional)

## New Spec Setup (Required Scaffolding)
Create a new spec as an isolated folder with standard files. Do not reuse another spec's directory.
1. Add a row to `src/specs/INDEX.md` with the new Spec ID and path.
2. Create `src/specs/<SPEC-ID>/spec.md` from the tech spec template.
3. Create `src/specs/<SPEC-ID>/progress.md` from the progress template.
4. Create `src/specs/<SPEC-ID>/journal/` and add the first journal entry.
5. Create `src/specs/<SPEC-ID>/reviews/` for persona review outputs.
6. Add `src/specs/<SPEC-ID>/artifacts/` only if the spec needs unique assets or diagrams.

## File Roles
- `spec.md`: the current spec draft. This is always the authoritative, latest spec text.
- `progress.md`: the resume snapshot. It tells anyone where the spec stands right now and what to do next.
- `journal/`: chronological change log entries; one file per work session.
- `reviews/`: persona review reports tied to the spec version being reviewed.
- `artifacts/`: optional supporting docs, mockups, or diagrams.

## Isolation Protocol (Always Enforced)
- Work on one spec at a time; confirm the active Spec ID before loading files.
- Load only the active spec's files unless explicit permission is given to open other specs.
- If a comparison is requested, ask for the exact Spec IDs to load before proceeding.

## Lifecycle Over Time
1. **Draft** (v0.1+) - outline, initial goals, early assumptions.
2. **Review Round(s)** (v0.x) - persona reviews, gaps logged, revisions applied.
3. **Decision** (v1.0) - accepted, revised, or rejected with explicit rationale.
4. **Implementation** - links to tickets, code branches, or rollouts.
5. **Archived** - final state; keep for history and reuse.

## Update Protocol (Every Change)
1. Update `spec.md` (bump version, update Last Updated).
2. Update `progress.md` (resume snapshot, new risks, updated decisions).
3. Add a `journal/` entry with the change summary and rationale.
4. If persona reviews occurred, add/update `reviews/` entries and mark status in `progress.md`.
5. Update `src/specs/INDEX.md` for current status and version.

## Avery Retrieval Path (Always the Same)
1. Open `src/specs/INDEX.md` to find the active spec.
2. Read `src/specs/<SPEC-ID>/progress.md` for the current stage and next action.
3. Read `src/specs/<SPEC-ID>/spec.md` for the latest spec content.
4. Skim `src/specs/<SPEC-ID>/journal/` for history and rationale.
5. Review `src/specs/<SPEC-ID>/reviews/` for persona rankings.
