# product-integrator — trigger recipe

`product-integrator` applies the WSO2 Integrator extension on top of a VSCode fork to ship a standalone IDE.
It does **not** re-implement icon logic or type detection — those come from `vscode-extensions`. The only
trigger change here is duplicating the icon assets, which are loaded at runtime from the extension's `assets/`
dir and are not symlinked/inherited from `vscode-extensions`.

Repo: `product-integrator` (upstream `wso2/product-integrator`). This step is required on **both** delivery
paths (monorepo and old-repos) — it is independent of where the code/assets went.

## Check first — may already be done

The same trigger can be onboarded via both paths over time (a patch *and* a front-port), so its
`product-integrator` icons may already have been submitted. **Before doing anything, ask the user whether a
`product-integrator` icon PR for this trigger already exists**, and check the target branch for
`wi/wi-extension/assets/{dark,light}-bi-<x>.svg`. If both are already present, **skip this repo entirely** —
don't open a duplicate PR.

## Duplicate the two SVGs (no code changes)

Copy the **same** dark/light 64×64 SVGs you added to `vscode-extensions` into:

- `wi/wi-extension/assets/dark-bi-<x>.svg`
- `wi/wi-extension/assets/light-bi-<x>.svg`

Use identical file contents to the `vscode-extensions` versions
(`workspaces/bi/bi-extension/assets/dark-bi-<x>.svg` / `light-bi-<x>.svg`). The 24×24 font glyph is **not**
needed here. No code edits.

## Verify

There's nothing to build for an icon-only change — open the IDE side panel and confirm the trigger entry
renders its icon in both light and dark themes.
