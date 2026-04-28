# OMC Rebrand / Collision-Prevention Implementation Plan

> **For Hermes:** Use subagent-driven-development skill to implement this plan task-by-task after the T3 Code source is imported/forked into the OMC repo.

**Goal:** Rebrand the T3 Code-derived app to OMC so final `.app` packages, user-data dirs, ports, updater caches, package names, and CLI binaries do not collide with the existing T3 Code Alpha install.

**Architecture:** OMC remains a T3 Code-derived UI/runtime with agent-native spawn tools and external CLI-agent adapters. The rebrand is an isolation layer: it changes identity/config defaults without changing the core T3 project/thread UX. Hermes/OpenClaw/Claude Code/Codex remain external installs; OMC only connects to them through adapters.

**Tech Stack:** T3 Code monorepo, TypeScript, Electron packaging, Node/Bun package metadata, macOS app bundle metadata, local config/data dirs, CLI package/bin names.

---

## Non-Negotiable Collision Targets

Existing T3 Code Alpha on João's machine uses:

- app path: `/Applications/T3 Code (Alpha).app`
- bundle id: `com.t3tools.t3code`
- app/user data: `~/Library/Application Support/t3code`
- caches/preferences: `com.t3tools.t3code*`, `t3-code-desktop-updater`
- data home: `~/.t3`
- default backend port: `3773`
- server package/bin naming: `t3`

OMC must not reuse those defaults.

Target OMC identity:

- product name: `OMC`
- macOS app name: `OMC.app`
- bundle/app id: `com.jfsantoropereira.omc`
- Electron userData dir: `~/Library/Application Support/omc`
- OMC data home: `~/.omc` or repo-local `.omc-home` for dev
- default backend port: choose non-3773, e.g. `13783`
- package names: avoid publishing/linking as `t3`; use OMC-specific names
- CLI binary: prefer `ospawn`; avoid global `t3`

## Acceptance Criteria

- Building OMC creates `OMC.app`, not `T3 Code (Alpha).app`.
- App bundle id is not `com.t3tools.t3code`.
- Running OMC does not read/write `~/.t3` unless explicitly configured.
- Running OMC does not bind `127.0.0.1:3773` by default.
- OMC package/bin names do not overwrite a global `t3` command.
- Existing T3 Code Alpha can remain installed and running while OMC dev/build work happens.
- Rebrand changes are documented in `README.md`, `SystemOutline.md`, and relevant package docs.

---

## Task 1: Locate all upstream identity defaults

**Objective:** Find every T3 Code identity string that can collide with existing installs.

**Files:**
- Inspect after T3 Code source import:
  - root `package.json`
  - `apps/desktop/package.json`
  - `apps/server/package.json`
  - Electron builder/forge config files
  - desktop main process files
  - server config files
  - scripts under `scripts/`

**Step 1: Search identity strings**

Run:

```bash
rg -n "T3 Code|t3code|t3-code|com\.t3tools|~/.t3|\.t3|3773|\bt3\b" . \
  --glob '!*node_modules*' \
  --glob '!*dist*' \
  --glob '!*build*'
```

Expected: list of candidate identity/config references.

**Step 2: Classify each hit**

Create a temporary checklist in the implementation PR/commit message or plan notes with categories:

- app display/product names;
- bundle/app ids;
- userData/data home paths;
- updater/cache ids;
- ports;
- package names;
- CLI bin names;
- docs/examples only.

**Step 3: Commit classification only if saved in docs**

If a repo note/checklist is created:

```bash
git add docs/planning/2026-04-28-omc-rebrand-collision-plan.md
 git commit -m "docs: map OMC rebrand collision targets"
```

## Task 2: Centralize OMC identity constants

**Objective:** Avoid scattered hardcoded replacements by creating a single OMC identity source where possible.

**Files:**
- Create or modify a config file appropriate to the imported T3 Code structure, e.g.:
  - `packages/config/src/identity.ts`, or
  - `apps/desktop/src/identity.ts`, or
  - nearest existing config module.

**Step 1: Add identity constants**

Use equivalent constants adapted to actual repo structure:

```ts
export const OMC_IDENTITY = {
  productName: "OMC",
  appName: "OMC",
  appId: "com.jfsantoropereira.omc",
  protocolScheme: "omc",
  dataDirName: "omc",
  homeEnvVar: "OMC_HOME",
  defaultHomeDirName: ".omc",
  defaultPort: 13783,
  cliName: "ospawn",
} as const;
```

**Step 2: Replace consumers gradually**

Only replace code paths that are actually imported/used by runtime/build config. Do not mechanically replace docs/examples until runtime identity is safe.

**Step 3: Verify typecheck/build config import**

Run the smallest available validation command from the imported repo, e.g.:

```bash
pnpm typecheck
```

or the repo's actual equivalent.

Expected: no import/type errors.

## Task 3: Rebrand Electron/macOS app identity

**Objective:** Ensure packaged app does not collide with `/Applications/T3 Code (Alpha).app`.

**Files:**
- Electron package config/build script files discovered in Task 1.

**Step 1: Change product/app id**

Set:

```text
productName = OMC
appId = com.jfsantoropereira.omc
artifact/app name = OMC
```

**Step 2: Change updater/cache identifiers**

Any updater/cache id containing `t3code`, `t3-code`, or `com.t3tools.t3code` should become OMC-specific.

Suggested:

```text
omc-desktop-updater
com.jfsantoropereira.omc
```

**Step 3: Build/package smoke test**

Run the actual package command once known, e.g.:

```bash
pnpm desktop:package
```

Expected:

- output artifact includes `OMC.app`;
- no artifact named `T3 Code (Alpha).app`;
- generated plist/bundle id is `com.jfsantoropereira.omc`.

**Step 4: Inspect built app metadata**

Run:

```bash
plutil -p path/to/OMC.app/Contents/Info.plist | egrep 'CFBundleIdentifier|CFBundleName|CFBundleDisplayName|CFBundleExecutable'
```

Expected:

```text
CFBundleIdentifier => com.jfsantoropereira.omc
CFBundleName / DisplayName / Executable => OMC
```

## Task 4: Isolate OMC home/userData paths

**Objective:** Ensure OMC does not read/write `~/.t3` or `~/Library/Application Support/t3code` by default.

**Files:**
- desktop main process config;
- server config;
- dev runner scripts;
- any path helpers discovered in Task 1.

**Step 1: Change default home env var**

Preferred semantics:

```text
OMC_HOME overrides default OMC home.
T3CODE_HOME should not be the primary OMC env var.
```

Default:

```text
~/.omc
```

For dev, use repo-local or explicit env:

```bash
export OMC_HOME=/Users/joaofelipe/Desktop/OMC/.omc-home
```

**Step 2: Change Electron userData path**

Ensure Electron resolves to:

```text
~/Library/Application Support/omc
```

or explicitly set:

```ts
app.setPath("userData", path.join(app.getPath("appData"), "omc"));
```

adapted to actual T3 Code structure.

**Step 3: Verify no default `.t3` writes**

Run OMC in isolated dev mode with an explicit local home:

```bash
OMC_HOME=/Users/joaofelipe/Desktop/OMC/.omc-home OMC_PORT=13783 <dev-command>
```

In another terminal, verify:

```bash
find /Users/joaofelipe/Desktop/OMC/.omc-home -maxdepth 3 -print | sed -n '1,80p'
find ~/.t3 -maxdepth 1 -newer /tmp/omc-start-marker -print
```

Expected:

- OMC writes under `.omc-home` or `~/.omc`;
- no new default writes under `~/.t3`.

## Task 5: Change default ports

**Objective:** Ensure OMC can run while T3 Code Alpha is listening on `127.0.0.1:3773`.

**Files:**
- server config;
- desktop backend port config;
- dev runner scripts;
- docs/env examples.

**Step 1: Replace default production/backend port**

Set OMC default to:

```text
13783
```

Use env override:

```text
OMC_PORT
```

If compatibility with upstream code is needed, support `T3CODE_PORT` only as fallback, not primary.

**Step 2: Replace dev port defaults or add offset**

Ensure dev server/web ports avoid upstream defaults if possible.

Document actual chosen defaults after inspecting source.

**Step 3: Port verification**

Before launch:

```bash
lsof -nP -iTCP:3773 -sTCP:LISTEN || true
lsof -nP -iTCP:13783 -sTCP:LISTEN || true
```

After launch:

```bash
lsof -nP -iTCP:13783 -sTCP:LISTEN
```

Expected:

- existing T3 Code Alpha may keep `3773`;
- OMC binds `13783` or configured OMC port.

## Task 6: Rename package/bin surfaces that can collide

**Objective:** Prevent global/package-manager collisions with upstream `t3` bin/package names.

**Files:**
- root package metadata;
- server package metadata;
- CLI package metadata;
- docs/scripts that globally link/install bins.

**Step 1: Inspect bins**

Run:

```bash
node -e 'for (const f of ["package.json","apps/server/package.json"]) { try { const p=require("./"+f); console.log(f, p.name, p.bin) } catch {} }'
```

**Step 2: Rename OMC bins**

Preferred:

```json
{
  "bin": {
    "ospawn": "./dist/bin.mjs"
  }
}
```

Avoid exposing global `t3` unless explicitly kept as an internal, non-global command.

**Step 3: Verify local binary name**

Run after build:

```bash
pnpm exec ospawn --help
```

Expected: OMC/ospawn help output. No dependency on global `t3`.

## Task 7: Add startup guardrails

**Objective:** Fail loudly if OMC is about to run with collision-prone defaults.

**Files:**
- startup config module;
- server boot file;
- desktop main process.

**Step 1: Add guard conditions**

At startup, warn or fail in dev if:

- app id/product name is still T3 Code;
- home path resolves to `~/.t3` without explicit opt-in;
- port resolves to `3773` without explicit opt-in;
- userData path contains `t3code`.

**Step 2: Add tests if config module is testable**

Example assertions:

```ts
expect(resolveDefaultHome()).not.toContain(".t3");
expect(resolveDefaultPort()).toBe(13783);
expect(OMC_IDENTITY.appId).toBe("com.jfsantoropereira.omc");
```

**Step 3: Run tests**

Run actual test command once known, e.g.:

```bash
pnpm test -- identity
```

Expected: guardrail tests pass.

## Task 8: Documentation update

**Objective:** Keep repo docs aligned with the rebrand/isolation requirement.

**Files:**
- Modify: `README.md`
- Modify: `SystemOutline.md`
- Modify: relevant component README(s) after source import

**Step 1: Document final identity**

Add/update:

```text
OMC product name: OMC
Bundle id: com.jfsantoropereira.omc
Default home: ~/.omc
Default port: 13783
CLI: ospawn
```

**Step 2: Document coexistence guarantee**

State that OMC must coexist with T3 Code Alpha without touching:

- `/Applications/T3 Code (Alpha).app`
- `~/Library/Application Support/t3code`
- `~/.t3`
- `127.0.0.1:3773`

**Step 3: Commit docs**

```bash
git add README.md SystemOutline.md docs/planning/2026-04-28-omc-rebrand-collision-plan.md
git commit -m "docs: plan OMC rebrand collision prevention"
```

## Task 9: Final coexistence smoke test

**Objective:** Prove OMC and existing T3 Code Alpha can coexist.

**Files:**
- No source files unless smoke test reveals bugs.

**Step 1: Leave T3 Code Alpha running**

Verify:

```bash
lsof -nP -iTCP:3773 -sTCP:LISTEN
```

Expected: T3 Code Alpha is still using `3773`.

**Step 2: Launch OMC isolated**

Run with explicit env:

```bash
OMC_HOME=/Users/joaofelipe/Desktop/OMC/.omc-home OMC_PORT=13783 <omc-dev-command>
```

Expected: OMC starts and binds non-3773 port.

**Step 3: Verify app/data separation**

```bash
lsof -nP -iTCP:13783 -sTCP:LISTEN
find /Users/joaofelipe/Desktop/OMC/.omc-home -maxdepth 3 -print | sed -n '1,80p'
plutil -p path/to/OMC.app/Contents/Info.plist | egrep 'CFBundleIdentifier|CFBundleName|CFBundleDisplayName|CFBundleExecutable'
```

Expected:

- T3 Code Alpha remains untouched;
- OMC uses OMC identity/data/port;
- packaged app metadata says OMC.

## Implementation Notes

- Do this before implementing `ospawn` adapters.
- Do not globally link/install during early testing.
- Do not install any built `.app` into `/Applications` until bundle identity is verified.
- Prefer repo-local/dev env first: `.omc-home`, non-default port, no global bin.
- Keep external agents external; this rebrand is about OMC/T3 package identity, not Hermes/OpenClaw/Claude Code installation.
