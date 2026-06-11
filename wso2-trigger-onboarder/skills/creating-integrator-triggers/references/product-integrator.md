# product-integrator — trigger recipe

`product-integrator` applies the WSO2 Integrator extension on top of a VSCode fork to ship a standalone IDE.
It does **not** re-implement icon logic or type detection — those come from `vscode-extensions`. The only
trigger change here is duplicating the icon assets, which are loaded at runtime from the extension's `assets/`
dir and are not symlinked/inherited from `vscode-extensions`.

Repo: `product-integrator` (upstream `wso2/product-integrator`).

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
