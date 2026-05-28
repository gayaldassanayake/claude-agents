# vscode-extensions — trigger recipe

The TypeScript VSCode frontend. For a trigger, changes here are icon wiring + SVG assets (and, for CDC, one
toggle). The form rendering itself is driven by the LS models, so there is no form code to write here.

Repo: `vscode-extensions` (upstream `wso2/vscode-extensions`).
Edit only files under `src/` — never the generated `build/`, `lib/`, `dist/`, or `resources/jslibs/` outputs.

## Icon mapping (3 edits)

Three `getCustomEntryNodeIcon(type)` functions map a module name to the `bi-<x>` icon. Add a `case` to each.
Match the `type` value the surrounding cases use — it differs by file (some key on `trigger.<x>`, some on the
bare `<x>`), so add both forms when unsure, mirroring how Shopify is handled.

1. `workspaces/ballerina/ballerina-extension/src/utils/project-artifacts.ts` — `getCustomEntryNodeIcon`
   (≈line 525) returns a **string**:
   ```ts
   case "trigger.shopify":
   case "shopify":
       return "bi-shopify";
   case "mysql":
       return "bi-mysql";
   ```

2. `workspaces/ballerina/ballerina-visualizer/src/views/BI/ComponentListView/EventIntegrationPanel.tsx` —
   `getCustomEntryNodeIcon` (≈line 96) returns **JSX** with a brand color:
   ```tsx
   case "trigger.shopify":
       return <Icon name="bi-shopify" sx={{ color: "#95BF47" }} />;
   case "mysql":
       return <Icon name="bi-mysql" sx={{ color: "#00678c" }} />;
   ```

3. `workspaces/ballerina/component-diagram/src/components/nodes/EntryNode/components/GeneralWidget.tsx` —
   `getCustomEntryNodeIcon` (≈line 91), same JSX form (note this file keys Shopify on `"shopify"`):
   ```tsx
   case "shopify":
       return <Icon name="bi-shopify" sx={{ color: "#95BF47" }} />;
   ```

## CDC toggle (CDC triggers only)

`workspaces/ballerina/ballerina-visualizer/src/views/BI/ServiceDesigner/index.tsx` (≈line 366) — extend the
`setIsCdcService(...)` OR-condition with the new module so the designer shows CDC templates / inline data
binding:
```ts
setIsCdcService(service.moduleName === "mssql" || service.moduleName === "postgresql"
    || service.moduleName === "mysql");
```
Event/webhook triggers skip this.

## Icon assets (3 SVGs)

Ask the user for brand icons (CP3 in SKILL.md) — never invent them. Add, named `bi-<x>`:

- `workspaces/bi/bi-extension/assets/dark-bi-<x>.svg` — 64×64, tuned for dark theme.
- `workspaces/bi/bi-extension/assets/light-bi-<x>.svg` — 64×64, tuned for light theme.
- `workspaces/common-libs/font-wso2-vscode/src/icons/bi-<x>.svg` — 24×24 glyph, `fill="currentColor"`.

Include the Apache-2.0 / WSO2 LLC copyright header used by the neighbouring SVGs.

The font glyph codepoint is **auto-generated** by the `gen-icons` script at build time (it can shift other
icons' codepoints in the generated `package.json` files — that's expected; don't hand-edit codepoints).

## Build / verify

```bash
rush build --to ballerina
```
Then confirm the new trigger shows the icon in the Event Integration panel and component diagram.
