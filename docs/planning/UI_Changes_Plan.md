# OMC UI Changes Plan

## Goal
Plan the UI changes required to make OMC feel native inside the T3 Code interface.

The current assumption is that the T3 sidebar remains the primary navigation surface and OMC extends it rather than replacing it.

## Confirmed Product Requirements

### 1) Master Agent entry in sidebar
The Master Agent should be directly openable from the main sidebar. It should appear:
- above the `Projects` section
- as a first-class navigation target
- always visible, regardless of which project is active

### 2) Admin Agent visual treatment
Admin Agent threads should have their font rendered **bold** in the sidebar thread list.

This applies to all Admin Agents in a project, since a project can have multiple admins.

## MVP UI Changes

### A. Sidebar: add Master Agent section above Projects
Expected behavior:
- add a dedicated sidebar group above the current `Projects` header
- suggested order:
  1. Search
  2. Master Agent
  3. Projects
  4. Footer / settings / update pill
- clicking `Master Agent` navigates to the master thread route and uses the same active-state treatment as a normal thread row

Recommended label:
- `Master Agent`

Likely files once code exists:
- `apps/web/src/components/Sidebar.tsx`
- `apps/web/src/threadRoutes.ts`
- `apps/web/src/store.ts`

### B. Sidebar: Admin Agent threads should render bold
Expected behavior:
- any thread designated as an Admin Agent inside a project should render with bold title text in the sidebar thread row

Minimum styling direction:
- current style equivalent: `truncate text-xs`
- admin variant: `truncate text-xs font-semibold`

Likely files once code exists:
- `apps/web/src/components/Sidebar.tsx`
- `apps/web/src/types.ts`
- `apps/web/src/store.ts`

## Recommended Additional MVP Refinements

### Sort Admin Agents near the top of each project list
If easy, Admins should render before regular threads inside the same project while preserving normal sort order inside each bucket.

### Master should be command-palette searchable
Add an `Open Master Agent` action once command palette integration exists.

### Master should look distinct from ordinary project threads
Use:
- its own section
- a unique icon
- the same active-state pattern as thread rows

## Data Model Needs for UI
To support the UI cleanly, the frontend will likely need:
- `isAdmin: boolean` on sidebar thread summaries
- a clean selector for the current master thread ref/shell

## Navigation Recommendation
- Master lives in a hidden "OMC Control" project with `workspaceRoot: ~` (user home directory)
- rendered separately in sidebar above Projects
- the control project is **not** exposed as a normal project entry
- Master is a normal chat thread — no distinct dashboard view
- agent status views (running, waiting, archived) live in `hatch` CLI, not in UI

## App Lifecycle
- app stays active even when screen is locked (standard macOS behavior)
- app only stops when user explicitly quits
- heartbeat scheduler only fires while app is running

## Heartbeat Settings
- configurable in app settings panel
- settings: interval (e.g. every 4 hours), injected prompt text, enable/disable toggle
- heartbeat injects a user-turn message into the Master thread on schedule

## MVP Acceptance Criteria
- sidebar contains a `Master` section above `Projects`
- clicking `Master Agent` opens the master thread
- Master project does not appear in the normal Projects list
- Admin Agent thread titles render bold in the sidebar
- multiple Admin threads can appear bold in one project
- existing thread navigation and archive behavior continue to work
