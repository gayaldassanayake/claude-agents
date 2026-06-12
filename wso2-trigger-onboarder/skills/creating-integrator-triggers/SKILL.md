---
name: creating-integrator-triggers
description: Use when adding a new low-code integration trigger (event/webhook, database CDC, or message-broker/file source such as shopify, mysql cdc, kafka, rabbitmq, ftp) to the WSO2 Integrator. Trigger on requests to "onboard"/"add a trigger", wire a Ballerina listener/connector into the low-code editor, or produce the ballerina-vscode monorepo (or legacy language-server + vscode-extensions) plus product-integrator changes for a new event integration.
version: 1.1.0
---

# Creating WSO2 Integrator Triggers

A **trigger** is a low-code event-integration entry point in the WSO2 Integrator, backed by a Ballerina
connector's listener + service (e.g. shopify, mysql cdc, kafka, rabbitmq, ftp). The pro-code capability
already exists in the Ballerina module; this skill adds the **low-code UX** for it.

## Choose the target first (CP0)

There are now **two delivery paths** — ask the user which one applies before anything else, and advise them:

- **New feature / front-port** (ships in the next minor) → the **`ballerina-vscode` monorepo**. After the
  Integrator 5.0.0 release, `ballerina-language-server` and `vscode-extensions` were merged into the single
  repo `wso2/ballerina-vscode`; new work goes there as **one PR** carrying both LS and VSCode code.
- **Patch to an already-released minor** → the **old separate repos** (`ballerina-language-server` +
  `vscode-extensions`), unchanged from before — one PR each.

Either way, also ask for the exact **base branch** for the chosen target (no default — always ask; the
monorepo default is usually `main`). The change points are identical across paths; only the repo set, paths,
and PR count differ.

| Path | Repos & references | PRs |
|---|---|---|
| **Monorepo** (new feature / front-port) | `ballerina-vscode` (`references/ballerina-vscode.md`, which reuses `references/language-server.md` + `references/vscode-extensions.md` under a path prefix) | **1** code PR + a `vscode-extensions` asset PR *(only if icon assets missing)* + a `product-integrator` icon PR *(only if not already done)* |
| **Old repos** (patch) | `ballerina-language-server` (`references/language-server.md`) + `vscode-extensions` (`references/vscode-extensions.md`) | **2** PRs + a `product-integrator` icon PR *(only if not already done)* |

Regardless of path, the **icon work** is: form-model/wiring edits, the SVG + font-glyph assets (these always
live in `vscode-extensions` — in the monorepo it's consumed as the `submodules/wso2-vscode-extensions`
submodule, so assets ship via a separate `vscode-extensions` PR), and a **`product-integrator` duplicate**
(`references/product-integrator.md`). The same trigger may be onboarded via *both* paths over time (patch +
front-port), so the `vscode-extensions` assets and the `product-integrator` icons may already exist — **always
check / ask before opening those PRs** so you don't duplicate work.

Default local clones live under `/Users/gayaldassanayake/Documents/event-integration/bi-repos/`
(`ballerina-vscode`, `ballerina-language-server`, `vscode-extensions`, `product-integrator`) — confirm the
paths with the user rather than assuming.

## Classify the trigger first

This determines the LS builder base class and which files you touch (full detail in
`references/language-server.md`):

- **Event / webhook** (Shopify, GitHub) → `AbstractServiceBuilder`, module name `trigger.<x>`.
- **Database CDC** (MySQL, MSSQL, PostgreSQL) → `AbstractCdcServiceBuilder`, module name `<x>`.
- **Message-broker / file** (Kafka, RabbitMQ, FTP) → minimal `AbstractServiceBuilder`. If such a trigger
  needs richer behavior, prefer the Shopify (event) or CDC pattern over the minimal one.

## Checkpoints (stop and confirm with the user)

This is an interactive workflow. Do not skip ahead.

- **CP0 — Target.** Ask whether this is a monorepo (`ballerina-vscode`) or old-repos delivery, advise per the
  rule above, and get the base branch(es). This decides the repo set and PR shape for everything that follows.
- **CP1 — Ballerina module.** Ask for the underlying module (`orgName`, `packageName`, `version`). Read its
  listener `init` params and service-type/remote-function contract (from Ballerina Central or the source) to
  derive the form. Present what you found before writing anything.
- **CP2 — Form preview.** Show the user how the listener + service forms will look — the fields, their types,
  and the "Create new / Use existing" listener choice — and get consent before implementing.
- **CP3 — Icons.** Request the dark/light 64×64 SVGs and the 24×24 glyph. Never invent brand assets; stop and
  ask if they're not provided.
- **CP4 — Before pushing / PRs.** Confirm fork remotes are correct, the base branch(es) for the chosen
  target, and that each PR targets its intended upstream branch. For the monorepo path also confirm whether
  the `vscode-extensions` asset PR is needed (run the asset existence check) and whether the
  `product-integrator` icon PR is needed (ask if it was already done via the other path). Pushing and opening
  PRs affect shared state — get explicit go-ahead.
- **CP5 — UI screenshot.** When raising the PRs, ask the user for a screenshot of the finalized view (the new
  trigger's form/panel in the editor). If they provide one, embed it in the code PR description — the
  **`ballerina-vscode`** PR on the monorepo path, or the **language-server** and **vscode-extensions** PRs on
  the old-repos path — so reviewers can see the new UI. It's optional — proceed without it if the user
  declines.

## Execution order

Same change points either way; the repo(s) you apply them in depend on CP0.

1. **Setup** — for the target repo(s): ensure cloned with `origin` = the user's fork and `upstream` = the
   official repo; `git fetch upstream`; **ask the user which branch to branch from** (a local branch or an
   upstream branch — no default, always ask) and create the working branch off it.
2. **LS changes** — apply `references/language-server.md`, then **write and run the LS tests** (mandatory —
   see below). Monorepo path: under `packages/ballerina-language-server/…` per `references/ballerina-vscode.md`.
3. **VSCode code** — icon wiring + CDC toggle per `references/vscode-extensions.md`. Monorepo path: under
   `packages/<pkg>/…`.
4. **Icon assets** — the SVG + font-glyph files (`references/vscode-extensions.md`). Old-repos path: same
   `vscode-extensions` branch as step 3. Monorepo path: a **separate `vscode-extensions` PR** on the submodule
   branch, **only if the assets don't already exist** (`references/ballerina-vscode.md`).
5. **product-integrator** — after asking whether it was already done, duplicate the two SVGs per
   `references/product-integrator.md` (skip if already present).
6. **PRs** — after CP4, push each branch to `origin` and open the PRs against the chosen upstream branches:
   monorepo path → one `ballerina-vscode` PR (+ conditional asset & PI PRs); old-repos path → a
   `ballerina-language-server` PR + a `vscode-extensions` PR (+ conditional PI PR).

## LS tests are mandatory

Every trigger must add tests matching its flavor (modeled on the Shopify / MySQL-CDC fixtures) and they must
**pass before you commit or open the LS PR**. See the "LS tests" section of `references/language-server.md`
for the test classes, fixture layout, and the per-class run command. Do not blindly accept regenerated
expected output — the data-driven harness rewrites the config JSON on failure; review the diff.

## Build / verify

```bash
# language-server (old repos: from repo root; monorepo: from packages/ballerina-language-server/)
./gradlew clean pack -x test
./gradlew :service-model-generator:service-model-index-generator:run         # only if index resources changed
./gradlew :service-model-generator:service-model-generator-ls-extension:test --tests "io.ballerina.servicemodelgenerator.extension.<TestClass>"
# vscode-extensions (old repos)
rush build --to ballerina
# ballerina-vscode (monorepo) — scoped Rush package names
rush build --to @wso2/ballerina-visualizer    # or --to @wso2/component-diagram; `rush build --to ballerina` builds the full extension
```
`product-integrator` is icon-only — verify visually. On the monorepo path the LS build runs from inside
`packages/ballerina-language-server/`; see `references/ballerina-vscode.md` for the full command set.

Ask the user for the local clone paths rather than assuming a layout. The exact build commands can vary by
checkout — check each repo's `README.md`/`CLAUDE.md`/`CONTRIBUTING.md`/`AGENTS.md` if the commands above don't
apply.

## Git / PR workflow

Per target repo (ask the user which branch to branch from — a local branch or an upstream branch; no default):
```bash
git fetch upstream
# upstream branch: git checkout -b onboard-<trigger>-trigger upstream/<base-branch>
# local branch:    git checkout -b onboard-<trigger>-trigger <base-branch>
git checkout -b onboard-<trigger>-trigger <base-branch>
# ... changes ...
git push -u origin onboard-<trigger>-trigger
gh pr create --repo <upstream> --base <base-branch> --head <fork-user>:onboard-<trigger>-trigger
```

Which PRs to open:
- **Monorepo path:** one `wso2/ballerina-vscode` PR (LS + VSCode code). Then, only if needed, a separate
  `wso2/vscode-extensions` asset PR (after the existence check) and a `wso2/product-integrator` icon PR (after
  asking). See `references/ballerina-vscode.md`.
- **Old-repos path:** a `ballerina-platform/ballerina-language-server` PR + a `wso2/vscode-extensions` PR
  (code + assets), plus a `wso2/product-integrator` icon PR (after asking).

Keep each repo's changes minimal and on-topic. Don't push or open PRs before CP4.

**UI preview in PRs (CP5).** Ask the user for a screenshot of the finalized view and, if provided, add it
under a `## UI preview` heading in the code PR — the `ballerina-vscode` PR (monorepo) or the language-server
and vscode-extensions PRs (old repos). GitHub only renders images it hosts, so a local file path won't embed:
either embed a URL the user supplies (`![UI preview](<url>)`), or create the PR and ask the user to drag the
screenshot into the description on GitHub. Skip the heading if no screenshot is given.
