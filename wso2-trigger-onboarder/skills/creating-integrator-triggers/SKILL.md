---
name: creating-integrator-triggers
description: Use when adding a new low-code integration trigger (event/webhook, database CDC, or message-broker/file source such as shopify, mysql cdc, kafka, rabbitmq, ftp) to the WSO2 Integrator. Trigger on requests to "onboard"/"add a trigger", wire a Ballerina listener/connector into the low-code editor, or produce the language-server + vscode-extensions changes for a new event integration.
version: 1.0.0
---

# Creating WSO2 Integrator Triggers

A **trigger** is a low-code event-integration entry point in the WSO2 Integrator, backed by a Ballerina
connector's listener + service (e.g. shopify, mysql cdc, kafka, rabbitmq, ftp). The pro-code capability
already exists in the Ballerina module; this skill adds the **low-code UX** for it.

Adding a trigger spans **two repos**, each with its own reference file:

| Repo | Role | Reference |
|---|---|---|
| `ballerina-language-server` | Backend: form models, source gen/parse, indexes (the bulk) | `references/language-server.md` |
| `vscode-extensions` | Frontend: icon wiring + SVG assets (+ CDC toggle) | `references/vscode-extensions.md` |

The end result is **two PRs**, one per repo, each from a branch off the user-chosen base branch, pushed to
the user's fork, targeting that branch on upstream.

## Classify the trigger first

This determines the LS builder base class and which files you touch (full detail in
`references/language-server.md`):

- **Event / webhook** (Shopify, GitHub) → `AbstractServiceBuilder`, module name `trigger.<x>`.
- **Database CDC** (MySQL, MSSQL, PostgreSQL) → `AbstractCdcServiceBuilder`, module name `<x>`.
- **Message-broker / file** (Kafka, RabbitMQ, FTP) → minimal `AbstractServiceBuilder`. If such a trigger
  needs richer behavior, prefer the Shopify (event) or CDC pattern over the minimal one.

## Checkpoints (stop and confirm with the user)

This is an interactive workflow. Do not skip ahead.

- **CP1 — Ballerina module.** Ask for the underlying module (`orgName`, `packageName`, `version`). Read its
  listener `init` params and service-type/remote-function contract (from Ballerina Central or the source) to
  derive the form. Present what you found before writing anything.
- **CP2 — Form preview.** Show the user how the listener + service forms will look — the fields, their types,
  and the "Create new / Use existing" listener choice — and get consent before implementing.
- **CP3 — Icons.** Request the dark/light 64×64 SVGs and the 24×24 glyph. Never invent brand assets; stop and
  ask if they're not provided.
- **CP4 — Before pushing / PRs.** Confirm fork remotes are correct, the per-repo base branches, and that
  each PR targets its chosen upstream branch. Pushing and opening PRs affect shared state — get explicit
  go-ahead.
- **CP5 — UI screenshot.** When raising the PRs, ask the user for a screenshot of the finalized view (the new
  trigger's form/panel in the editor). If they provide one, embed it in the **language-server** and
  **vscode-extensions** PR descriptions so reviewers can see the new UI. It's optional — proceed without it if
  the user declines.

## Execution order

1. **Setup** — for each repo: ensure it's cloned with `origin` = the user's fork and `upstream` = the
   official repo; `git fetch upstream`; **ask the user which branch to branch from** (a local branch or an
   upstream branch — no default, always ask) and create the working branch off it.
2. **language-server** — make the changes in `references/language-server.md`, then **write and run the LS
   tests** (mandatory — see below).
3. **vscode-extensions** — icon edits + SVG assets per `references/vscode-extensions.md`.
4. **PRs** — after CP4, push each branch to `origin` and open two PRs against the chosen upstream branches.

## LS tests are mandatory

Every trigger must add tests matching its flavor (modeled on the Shopify / MySQL-CDC fixtures) and they must
**pass before you commit or open the LS PR**. See the "LS tests" section of `references/language-server.md`
for the test classes, fixture layout, and the per-class run command. Do not blindly accept regenerated
expected output — the data-driven harness rewrites the config JSON on failure; review the diff.

## Build / verify

```bash
# language-server
./gradlew clean pack -x test
./gradlew :service-model-generator:service-model-index-generator:run         # only if index resources changed
./gradlew :service-model-generator:service-model-generator-ls-extension:test --tests "io.ballerina.servicemodelgenerator.extension.<TestClass>"
# vscode-extensions
rush build --to ballerina
```

Ask the user for the local clone paths rather than assuming a layout. The exact build commands can vary by
checkout — check each repo's `README.md`/`CLAUDE.md` if the commands above don't apply.

## Git / PR workflow (×2)

For each repo (ask the user which branch to branch from — a local branch or an upstream branch; no default):
```bash
git fetch upstream
# upstream branch: git checkout -b onboard-<trigger>-trigger upstream/<base-branch>
# local branch:    git checkout -b onboard-<trigger>-trigger <base-branch>
git checkout -b onboard-<trigger>-trigger <base-branch>
# ... changes ...
git push -u origin onboard-<trigger>-trigger
gh pr create --repo <upstream> --base <base-branch> --head <fork-user>:onboard-<trigger>-trigger
```
Keep each repo's changes minimal and on-topic. Don't push or open PRs before CP4.

**UI preview in PRs (CP5).** Ask the user for a screenshot of the finalized view and, if provided, add it to
the language-server and vscode-extensions PR bodies under a `## UI preview` heading. GitHub only renders
images it hosts, so a local file path won't embed: either embed a URL the user supplies
(`![UI preview](<url>)`), or create the PR and ask the user to drag the screenshot into the description on
GitHub. Skip the heading if no screenshot is given.
