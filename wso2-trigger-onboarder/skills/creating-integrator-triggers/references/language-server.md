# ballerina-language-server — trigger recipe

The language server (LS) is the backend of the low-code editor. It turns a Ballerina connector's
listener/service into the form models, source-generation and source-parsing logic the editor uses.
Adding a trigger here is the bulk of the work.

Repo: `ballerina-language-server` (upstream `ballerina-platform/ballerina-language-server`).

Unless noted, all paths below are relative to:
`service-model-generator/modules/service-model-generator-ls-extension/src/main/`
and the Java package is `io.ballerina.servicemodelgenerator.extension`.

## Pick the builder flavor first

Every trigger is wired through a Java `ServiceBuilder` registered in `ServiceBuilderRouter`. There is no
pure-JSON trigger — even the simplest still has a builder class. Choose the base class by trigger type:

| Trigger type | Canonical example | Base class | Service JSON |
|---|---|---|---|
| Event / webhook | Shopify (`trigger.shopify`), GitHub (`trigger.github`) | `AbstractServiceBuilder` | `services/<x>_trigger.json` |
| Database CDC | MySQL/MSSQL/PostgreSQL (`mysql` …) | `AbstractCdcServiceBuilder` | `services/cdc_<x>.json` |
| Message-broker / file | Kafka, RabbitMQ, Solace, FTP | `AbstractServiceBuilder` (minimal override) | `services/<x>.json` (FTP splits into `<x>_init.json` + `<x>_service.json`) |

**Guidance:** model new event triggers on `ShopifyTriggerServiceBuilder` and new CDC triggers on
`MysqlCdcServiceBuilder`. Kafka/RabbitMQ are minimal builders (override `kind()` + a little data binding)
and lean entirely on a JSON model; if a new trigger is that simple you may follow them, but when in doubt
prefer the richer Shopify/CDC approach. The fallback for an unregistered module is `DefaultServiceBuilder`
(`ServiceBuilderRouter.getServiceBuilder` uses `getOrDefault(protocol, DefaultServiceBuilder::new)`).

## Change points (checklist)

1. **Constant** — `util/Constants.java`
   Add the module-name constant. Event triggers prefix with `trigger.`:
   ```java
   public static final String TRIGGER_SHOPIFY = "trigger.shopify";
   public static final String MYSQL = "mysql";
   ```

2. **Service builder (NEW)** — `builder/service/<X>ServiceBuilder.java`
   - Event: `extends AbstractServiceBuilder`; override `kind()` (return the module name), load the JSON in
     `getServiceInitModel(...)`, and implement `addServiceInitSource(...)` (collapse the `configureListener`
     CHOICE via `applyEnabledChoiceProperty`, `unwrapGroupSections`, decide create-new vs use-existing, emit
     edits). See `ShopifyTriggerServiceBuilder`.
   - CDC: `extends AbstractCdcServiceBuilder`; the base class does init/source/parse. Override only:
     `getCdcServiceModelLocation()`, `getCdcDriverModuleName()`, `getListenerFields()`, `kind()`,
     `getDisplayLabel()`, `getMetadataKeys()`, and `extractDatabaseConfigFields(...)`. See
     `MysqlCdcServiceBuilder` (≈130 lines — the cleanest template for a new CDC DB).
   - Minimal (Kafka-style): override `kind()` and, if needed, `getModelFromSource(...)` to add data binding.

3. **Function builder** — `builder/FunctionBuilderRouter.java` (CDC and most triggers)
   Register the module in `CONSTRUCTOR_MAP`. CDC types reuse the shared `CdcFunctionBuilder`:
   ```java
   put(MYSQL, () -> new CdcFunctionBuilder(MYSQL));
   ```
   Event triggers without bespoke function handling can omit this (they fall through to the default).

4. **Service / form model JSON** — `resources/services/<x>.json`
   Defines the listener + service form: `configureListener` CHOICE ("Create new" / "Use existing"),
   `listenerConfig` GROUP_SECTIONs, field `metadata` (label/description), `types[].fieldType`
   (TEXT, NUMBER, EXPRESSION, CHOICE, FORM, GROUP_SECTION, SINGLE_SELECT, TEXT_SET, FLAG…), and `codedata`
   (`argType` + `originalName`) that maps a form field back to a Ballerina listener argument. Build this from
   the connector's listener `init` params and service-type contract — copy the closest existing file
   (`shopify_trigger.json` for event, `cdc_mysql.json` for CDC) and adapt fields.

5. **Service builder registration** — `builder/ServiceBuilderRouter.java`
   Add the import + a `CONSTRUCTOR_MAP` entry keyed by the Constant:
   ```java
   put(TRIGGER_SHOPIFY, ShopifyTriggerServiceBuilder::new);
   put(MYSQL, MysqlCdcServiceBuilder::new);
   ```

6. **Artifact entry point** — `architecture-model-generator/modules/architecture-model-generator-core/src/main/java/io/ballerina/artifactsgenerator/Artifact.java`
   - `entryPointMap`: key is the **bare** module name (last segment), value is the display label:
     ```java
     Map.entry("shopify", "Shopify Event Integration"),
     Map.entry("mysql", "CDC MySQL Service")
     ```
   - `moduleAnnotationFields` (only if the service is named via an annotation field, e.g. CDC `tables`):
     ```java
     "mysql", new String[]{"tables"}
     ```

7. **Gradle** — `gradle.properties` + `build.gradle`
   Add a version property and a `pullBallerinaModule` line in the `pullBallerinaModules` task. CDC needs the
   module **and** its driver:
   ```properties
   ballerinaxShopifyTriggerVersion = 1.6.0
   ballerinaxMysqlVersion=1.18.0
   ballerinaxMySQLCDCDriverVersion=1.0.2
   ```
   ```gradle
   dependsOn pullBallerinaModule('trigger.shopify', ballerinaxShopifyTriggerVersion)
   dependsOn pullBallerinaModule('mysql', ballerinaxMysqlVersion)
   dependsOn pullBallerinaModule('mysql.cdc.driver', ballerinaxMySQLCDCDriverVersion)
   ```

8. **Trigger registry** — `resources/trigger_properties.json`
   Add a numbered entry (next free id). `triggerName` is the human label; CDC entries usually omit it:
   ```json
   "15": { "name": "mysql", "orgName": "ballerinax", "packageName": "mysql",
           "keywords": ["mysql", "cdc", "event"] }
   ```

9. **Service artifacts index** — `service-model-generator/modules/service-model-index-generator/src/main/resources/service_artifacts.json`
   Add the full service declaration (service types, remote/resource functions, listener kind). Copy the
   Shopify or MySQL block as a template.

10. **SQLite index** — `resources/service-index.sqlite`
    Regenerated by the index generator (see below); do not hand-edit.
    **Do NOT touch the flow `central-index.sqlite`** — triggers don't require it.

## LS tests (REQUIRED — write and run; must pass before commit)

Data-driven TestNG tests under `.../service-model-generator-ls-extension/src/test/`, all extending
`AbstractLSTest`. Each test has a `@DataProvider` that scans its `config/` dir of JSON fixtures; each fixture
holds the input model and the expected output, run against a `source/sample<N>/` Ballerina package. On
mismatch the harness **rewrites the actual result back into the config JSON and fails** — so review the diff,
don't blindly accept regenerated output.

| Test class | `getResourceDir()` | What it checks | Fixtures to add |
|---|---|---|---|
| `AddServiceAndListenerTest` | `add_service_and_listener` | generate source from a `serviceInitModel` | `config/<x>_*.json` (input `serviceInitModel` + expected `output` TextEdits) + an empty `source/sample<N>/{Ballerina.toml,main.bal}` |
| `GetServiceModelFromSourceTest` | `get_sm_from_source` | parse existing source → service model | `config/*.json` + a fully-written `source/sample<N>/main.bal` |
| `GetServiceInitModelTest` | `get_service_init_model` | return the init/form model | `config/<x>_service_model.json` (request meta + expected `response`) |

Match the flavor to the canonical examples:
- **CDC (MySQL):** config variants in `add_service_and_listener/config/` — e.g. `cdc_<x>_all_ops.json`,
  `cdc_<x>_skip_update.json`, `cdc_<x>_no_databases.json` — plus the generation `source/sample<N>/`, and a
  full-service sample for `get_sm_from_source` (CDC shares `get_sm_from_source/config/cdc_service_model.json`).
- **Event (Shopify):** a `get_service_init_model/config/<x>_service_model.json` fixture.

If the new module isn't already pulled for tests, add it to `test.dependsOn` in the module's `build.gradle`
(e.g. `test.dependsOn rootProject.pullBallerinaModule('<module>', <versionProp>, '<org>')`).

Run (verify green before committing):
```bash
./gradlew :service-model-generator:service-model-generator-ls-extension:test \
  --tests "io.ballerina.servicemodelgenerator.extension.AddServiceAndListenerTest"
# repeat for GetServiceModelFromSourceTest / GetServiceInitModelTest
# single fixture: --tests "...AddServiceAndListenerTest.test[cdc_<x>_all_ops.json]"
```

## Build / regenerate

```bash
./gradlew clean pack -x test                                              # build the LS
./gradlew :service-model-generator:service-model-index-generator:run      # regen service-index.sqlite after index changes
```

## Event vs CDC at a glance

| Aspect | Event/webhook (Shopify) | Database CDC (MySQL) |
|---|---|---|
| Constant | `TRIGGER_SHOPIFY = "trigger.shopify"` | `MYSQL = "mysql"` |
| Base class | `AbstractServiceBuilder` | `AbstractCdcServiceBuilder` |
| Service JSON | `services/shopify_trigger.json` | `services/cdc_mysql.json` |
| Function builder | usually none | `CdcFunctionBuilder` via `FunctionBuilderRouter` |
| `codedata.argType` | `LISTENER_PARAM_*` | `databaseConfig`, `CDC_OPERATION_ENABLE`, `LISTENER_PARAM_*` |
| Gradle modules | `trigger.shopify` | `mysql` + `mysql.cdc.driver` (+ shared `cdc`) |
| Annotation fields | none | `tables` (in `moduleAnnotationFields`) |
| Init-model test | `get_service_init_model` fixture | `add_service_and_listener` + `get_sm_from_source` fixtures |
