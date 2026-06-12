# ballerina-vscode (monorepo) — trigger recipe

After the Integrator 5.0.0 release, `ballerina-language-server` and `vscode-extensions` were merged into a
single monorepo, **`wso2/ballerina-vscode`**. Use this path for **new features / front-ports** (work that
ships in the next minor). Patches to an already-released minor still go to the old separate repos — see
`references/language-server.md` and `references/vscode-extensions.md`.

Repo: `wso2/ballerina-vscode`. A trigger here is **one PR** that carries both the LS changes and the VSCode
code changes. Base branch is user-chosen (CP0) — typically `main`. Sample PR: `wso2/ballerina-vscode#632`
("Onboard MySQL CDC trigger", base `main`).

## The change points are unchanged — only the paths get a prefix

Do **not** re-learn the edits here. Apply exactly the same change points from `references/language-server.md`
and `references/vscode-extensions.md`, with these path prefixes:

| Old repo / path | Monorepo path |
|---|---|
| `ballerina-language-server` → `service-model-generator/...` | `packages/ballerina-language-server/service-model-generator/...` |
| `ballerina-language-server` → `architecture-model-generator/...` | `packages/ballerina-language-server/architecture-model-generator/...` |
| `vscode-extensions` → `workspaces/ballerina/<pkg>/...` | `packages/<pkg>/...` (e.g. `ballerina-extension`, `ballerina-visualizer`, `component-diagram`) |

So the LS reference's base path becomes
`packages/ballerina-language-server/service-model-generator/modules/service-model-generator-ls-extension/src/main/`,
and the three `getCustomEntryNodeIcon` edits + CDC toggle land under `packages/<pkg>/...`. The LS test
fixtures and `service-index.sqlite` move under `packages/ballerina-language-server/...` too.

## Icon assets live in the submodule — separate PR

The SVG assets and the 24×24 font glyph are **not in this monorepo**. They live in `wso2/vscode-extensions`,
consumed here as the git submodule `submodules/wso2-vscode-extensions` (pinned to `release/ballerina-5.12.x`).
The monorepo PR is therefore **code-only**; the assets ship via a **separate PR to `wso2/vscode-extensions`**
on the submodule's branch (the three files in the "Icon assets" section of `references/vscode-extensions.md`).

**Check before opening that asset PR** — the trigger may already have its assets if the old-repos path was
done first. On the submodule branch (`release/ballerina-5.12.x`), look for:

- `workspaces/bi/bi-extension/assets/dark-bi-<x>.svg`
- `workspaces/bi/bi-extension/assets/light-bi-<x>.svg`
- `workspaces/common-libs/font-wso2-vscode/src/icons/bi-<x>.svg`

If all three already exist, skip the asset PR. Otherwise add the missing ones and open it.

## Build / verify

```bash
# language-server (from inside the monorepo)
cd packages/ballerina-language-server
./gradlew clean pack -x test
./gradlew :service-model-generator:service-model-index-generator:run   # only if index resources changed
./gradlew :service-model-generator:service-model-generator-ls-extension:test --tests "io.ballerina.servicemodelgenerator.extension.<TestClass>"

# VSCode code (from the monorepo root) — scoped Rush package names
rush build --to @wso2/ballerina-visualizer      # or --to @wso2/component-diagram, etc.
rush build --to ballerina                        # full extension (equivalent to plain `rush build`)
```
Confirm the exact commands against the monorepo's `CONTRIBUTING.md` and root `AGENTS.md` — they document the
canonical `rush` / `./gradlew` invocations (`rush update` before first build, Java 21 + `packageUser`/
`packagePAT` for the LS).

## Git / PR workflow (one PR)

```bash
git fetch upstream
git checkout -b onboard-<trigger>-trigger <base-branch>   # base-branch from CP0, usually main
# ... LS changes under packages/ballerina-language-server/ + VSCode code under packages/ ...
git push -u origin onboard-<trigger>-trigger
gh pr create --repo wso2/ballerina-vscode --base <base-branch> --head <fork-user>:onboard-<trigger>-trigger \
  --title "Onboard <Trigger> trigger" --body "..."
```
Plus, when needed: the separate `wso2/vscode-extensions` asset PR (above) and the `product-integrator`
icon PR (`references/product-integrator.md`) — both only after their existence checks.
