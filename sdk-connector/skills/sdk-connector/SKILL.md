---
name: sdk-connector
description: Guides a developer end-to-end through generating a brand-new Ballerina connector from an existing Java SDK, driver, or client library. DO NOT use this if a service already has an OpenAPI specification, for a service that can be implemented by a simple HTTP wrapper, webhook, or patching an existing connector.
---

# SDK to Ballerina connector builder

This skill turns an existing Java SDK into a hand-written Ballerina connector through gated phases. Get explicit approval in each step before moving on to the next.

## Notes
- At the beginning of each phase, announce that the phase is starting, at the end of each phase, announce that the phase is complete.

## Phase 0 — Feasibility check

Decide whether an SDK-based connector is actually the right approach here, or whether the service is better served by an OpenAPI specification, a simple HTTP wrapper, a webhook, or any other mechanism.

**Hard gate.** If SDK-based generation isn't the right call, explain why, point at the better alternative, and stop — produce nothing further. Otherwise, resume to the next phase.

## Phase 1 — Domain education

Ask the user if they want a domain knowledge refresher. If they don't, move directly to *phase 2 - SDK choice*. If yes, create an *artifact* (see [Artifact requirements](#artifact-requirements)) explaining the domain and the scope, with below sections:

1. Domain - Domain is the subject area that the SDK operates in. You have to assume the developer has no prior knowledge of the domain. Explain the domain in plain language before any design conversation.
2. Capabilities - Include what kind of capabilities exist in that domain, with a summary and a deep dive for each with diagrams where necessary. These are general capabilities of the service, not tied to any specific SDK. Add a quiz on the service and its capabilities.

**Hard gate.** Get explicit developer sign-off on the domain understanding before moving on.

## Phase 2 - SDK choice

Ask a set of coarse, domain-shape questions — just enough to distinguish realistic SDK candidates. Do not walk the full capability list yet — that happens in Phase 4.

Use the Ask question tool with structured choice prompts, letting the developer add free-text reasoning for each answer.

If more than one SDK or library exists for this service, create an *artifact* (see [Artifact requirements](#artifact-requirements)) that includes a survey of the SDKs. Evaluate the realistic candidates, with their capabilities, and recommend one specific choice with reasons, factoring in the coarse answers above, and the general landscape.

After generating the artifact, use the Ask question tool to gather user's final decision on the SDK choice.

**Hard gate.** Get explicit developer sign-off on the SDK choice before moving on.

## Phase 3 - SDK exploration

Create an *artifact* (see [Artifact requirements](#artifact-requirements)) that explores the SDK and its capabilities. Add a quiz on the SDK capabilities.

**Hard gate.** Get explicit developer sign-off on SDK exploration before moving on.

## Phase 4 - Detailed scope negotiation

Walk the domain's capability surface capability-by-capability. Specify if each capability is built-in to the SDK, assembled from SDK primitives, or unsupported.

If user wants to have an unsupported capability, ask the developer to drop the capability, or reconsider the SDK choice (loop back to Phase 2) explicitly.

Use the Ask question tool with structured choice prompts, letting the developer add free-text reasoning for each answer. Curate the final scope from the feedback and write it into a `scope.md` as a list of capabilities that are in scope and out of scope.

**Hard gate.** Get explicit developer sign-off on the negotiated scope before moving on.

## Phase 5 — Ballerina API design

Ask the developer if there are one or more Ballerina connector for a similar service they'd like used as a reference. If provided, use it for idiom and convention consistency. DO NOT blindly copy the structure.

Come up with a design for the Ballerina API that wraps the SDK. Base this off the `scope.md`. Include only the publicly exposed client, listener, service, types, errors, annotations, configurations, public functions etc. Do not implement any logic yet — give every public function and method a `panic error("not implemented")` body so the design compiles as-is.

Expose this specification as a `spec.md` file. The `spec.md` should contain the Ballerina APIs added as code snippets. Document the spec with documentation comments explaining each construct. Any construct that cannot be expressed in Ballerina should be documented separately. (eg:- Listener/ service shapes, compiler plugin validations, etc). Ensure that the documentation comments are clear and concise.

Ask the developer if they want an additional *artifact* (see [Artifact requirements](#artifact-requirements)) that explains each construct in the API design, and the reasoning behind the design decisions. If yes, create the artifact.

Use the Ask question tool to gather feedback on the API design, and update the Ballerina API design in `spec.md` based on the feedback. Repeat this process until the developer approves the API design.

Use the Ballerina skill specified in the References section when creating the `spec.md`.

**Hard gate.** Get explicit developer sign-off on the finalized API specification before writing any tests.

### Compiler plugins

Identify whether any part of the finalized API needs compile-time validation that only a Ballerina compiler plugin can provide (required annotation fields, invalid listener signatures, invalid config combinations). If a plugin is warranted, inform the developer specifically which validations it would cover ("X, Y, and Z need compile-time validation because ..."), and ask for approval/ disapproval.

## Phase 6 - Create connector scaffold

Clone the github repository if the developer provides one. If they do not, set up the initial code repository structure in a new repository. Create the repository structure using the template - https://github.com/gayaldassanayake/sdk-connector-template. Replace the placeholders in the scaffold with the appropriate values. If you need inputs from the developer, use the Ask question tool to gather feedback. Use `TEMPLATE.md` for the list of placeholders and their descriptions. Once the scaffold is created, delete the `TEMPLATE.md` file from the scaffold.

If a compiler plugin is approved, keep the compiler-plugin, compiler-plugin-tests directories in the scaffold. If not, or if none is needed, proceed without one.

**Hard gate.** Get explicit approval on the connector scaffold before moving on to the next phase.

## Phase 7 — Test suite (before implementation)

Write a comprehensive test suite before any implementation code exists. This should cover end-to-end tests. Use sandboxed instance of the service where possible; tests can rely on it (Docker images etc). You should present different local setup options to the developer and ask for approval on which one to use.

Ballerina tests are a must. However, based on the service, you can also write Java tests in addition. Include adding compiler-plugin-tests if a compiler plugin is included in the scaffold. Writing tests doesn't need heavy developer involvement; the bar is thoroughness.

**Hard gate.** Get explicit developer sign-off on the test suite before moving on to the next phase.

## Phase 8 — Implementation

Move the spec.md file created in phase 5 into <repo-root>/docs/spec/spec.md. This is the finalized API specification.

Create a TODO list of tasks to implement the connector based on the `spec.md`. Sort the tasks by priority and complexity. 

Comment or disable all the tests initially. Implement the Java native layer and the Ballerina layer for each task. At the end of each task, uncomment relevant tests, run them and fix any issues until everything passes. Cross check with the `spec.md` to ensure that the implementation is in line with the finalized API specification. Use the Ballerina skill specified in the References section to implement the connector. Use ballerina version 2201.12.0 for the implementation.

Once all tasks are completed, run the full test suite and fix issues iteratively until everything passes. No tests should be disabled at this point. This is a loop — keep going until green, don't hand back a partially-passing suite.

**Hard gate.** Get explicit developer sign-off on the implementation before moving on to the next phase.

## Phase 9 — Test coverage

Verify that the combined test coverage is at least 85%. If not, add more tests to increase the coverage. Use the Ask question tool to gather feedback on the test coverage and update the test suite based on the feedback. Repeat this process until the developer approves the test coverage.

**Hard gate.** Get explicit developer sign-off on the test coverage before moving on to the next phase.

## Phase 10 — Standalone examples

Pack and publish the ballerina module to the local repository, before generating the standalone examples.

``` bash
cd ballerina
ballerina pack
ballerina push --repository=local
```

Write three to four example programs living outside the test suite, to validate real-world usage independent of the automated tests. They should be in the <repo-root>/examples directory. Pick the examples that cover the most common use cases of the connector, with less overlap. Ask the developer for approval on the examples before writing them. Use the Ask question tool to gather feedback on the examples and update them based on the feedback. Repeat this process until the developer approves the examples.

The examples should be self-contained and runnable with required sandbox configurations (docker-compose etc), and a README.md file explaining how to run the examples. In their `Ballerina.toml` include the following table to specify the dependency on the connector module.

``` toml
[[dependency]]
org = "<connector-org>"
name = "<connector-name>"
version = "<connector-version>"
repository = "local"
```

**Hard gate.** Get explicit developer sign-off on the examples before moving on to the next phase.

## Phase 11 — Documentation

- Improve the connector's user-facing README (`ballerina/README.md`, published to Ballerina Central). The scaffold should already have a ballerina/README.md file with a template. Enhance the README.md file with the following sections:
  - Overview
  - Setup Guide
  - Quickstart - Include simple examples for the most common use cases of the connector
  - Examples - Link to the examples in the <repo-root>/examples directory

- Improve the repository README (`<repo-root>/README.md`) with the following sections:
  - Overview
  - Examples
  - Build from source
  - Contribute to Ballerina
  - Code of conduct
  - Useful links

- Add the following fields to the Ballerina.toml [package] table:
  - keywords = ["ballerina", "connector", "<domain>", "<service>", "Vendor/<vendor-name>", "Area/<Area>", "Type/<Connector|Trigger|Driver>", "Type/<Connector|Trigger|Driver>"] # Refer `References` section for the allowed values of Area and Type.
  - repository = "The URL of the repository where the connector is hosted"
  - authors = ["Ballerina"]
  - license = ["Apache-2.0"]
  - distribution = "2201.12.0"
  - icon = "icon.png" # Ask the developer for an icon (png). save it as `ballerina/icon.png`.
  - documentation = "The URL of the documentation for the connector"

- Confirm every public Ballerina construct already has a doc comment from Phase 5 — this phase checks that's actually true rather than writing them from scratch.

**Hard gate.** Get explicit developer sign-off on the documentation before moving on to the next phase.

## Phase 12 - GraalVM verification

Ask the developer if they want to verify the connector works with GraalVM native image. If yes, use skill https://github.com/ballerina-platform/ballerina-library/tree/graalvm-skills/agent-skills/skills/making-graalvm-compatible to verify this. If no, skip this step.


## Artifact requirements
- Render a rich, interactive explanation *artifact* with diagrams used as appropriate. The artifact should be an HTML artifact that includes CSS and JavaScript. Don't use ASCII diagrams. Always use simple HTML designs for your diagrams, HTML lists for lists of things, etc.

- If it's explicitly asked to add a quiz, include 5-10 MCQ questions that test the developers's knowledge of the content in the artifact, at the end. This should be medium difficulty, difficult enough that you actually need to understand the substance of the artifact to answer them, but not gotchas.

## References

- Use skill https://github.com/ballerina-platform/skills/tree/main/skills/ballerina for Ballerina code generation.
- Allowed `Type` values - `Type/Connector`, `Type/Trigger`, `Type/Driver`. `Connector` if there are clients, `Trigger` if there are Listeners/Services. `Driver` if this is a driver containing only Java Dependencies. Can have multiple `Type/` keywords if the connector has both clients and Listeners/Services. `Area` can be `Area/Database`, `Area/Messaging`, `Area/Storage`, `Area/Utility`, etc. depending on the domain of the service.
- Allowed `Area` values - `Area/AI`, `Area/Analytics`, `Area/Cloud`, `Area/Communication`, `Area/CRM`, `Area/Database`, `Area/Developer`, `Area/DevOps`, `Area/E-Commerce`, `Area/ERP`, `Area/Finance`, `Area/Healthcare`, `Area/HRMS`, `Area/Marketing`, `Area/Messaging`, `Area/Other`, `Area/Productivity`, `Area/Security`, `Area/Storage`
