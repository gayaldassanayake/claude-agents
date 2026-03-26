---
name: ballerina
description: This skill should be used when the user is working with Ballerina — writing Ballerina code, setting up Ballerina projects, debugging Ballerina compiler or runtime errors, or implementing integration patterns (HTTP services/clients, databases, messaging, Kafka). Trigger on ".bal" files, "Ballerina.toml", "bal new", "bal build", "bal run", or any mention of the Ballerina programming language.
version: 1.0.0
---

# Ballerina Skill

Ballerina is a statically-typed, cloud-native programming language designed for writing network-distributed applications. It has a structural type system, built-in concurrency model (strands), first-class support for services, and an extensive standard library for integrations.

## Key Language Characteristics

- **Structural type system**: Compatibility is determined by structure, not name
- **Union types**: `int|string`, optional values as `T?` (sugar for `T|()`)
- **Error handling**: Errors are values; `check` propagates them; `error` type is distinct
- **Services**: First-class language construct, not a framework
- **Concurrency**: Lightweight threads called "strands"; `lock` for shared state

## Project Setup

### Initialize a project
```bash
bal new <project-name>          # creates a package with Ballerina.toml + main.bal
bal new <project-name> --template service  # HTTP service template
```

### Ballerina.toml structure
```toml
[package]
org = "myorg"
name = "mypackage"
version = "0.1.0"
distribution = "2201.10.0"   # Ballerina distribution version

[dependencies]
# Pulled from Ballerina Central (central.ballerina.io)
```

### Common CLI commands
```bash
bal build          # compile
bal run            # run main.bal / service
bal test           # run tests in tests/ directory
bal push           # publish to Ballerina Central
bal pull myorg/pkg # add a dependency
```

### Package structure
```
my-package/
├── Ballerina.toml
├── Dependencies.toml   # auto-generated, commit this
├── main.bal            # or service.bal for services
├── modules/
│   └── mymodule/       # sub-modules
│       └── module.bal
└── tests/
    └── main_test.bal
```

## Writing Idiomatic Ballerina Code

### Types and records
```ballerina
// Open record (allows extra fields)
type Person record {
    string name;
    int age;
};

// Closed record (exact shape)
type Point record {|
    float x;
    float y;
|};

// Union type
type StringOrInt string|int;

// Optional field
type Config record {
    string host;
    int port = 8080;       // default value
    string? apiKey = ();   // optional, defaults to nil
};
```

### Error handling
```ballerina
// check propagates errors up (function must return error|T)
json data = check io:fileReadJson("config.json");

// trap converts panics to errors
int result = check trap riskyOperation();

// Custom error types
type DatabaseError error<record {| string sqlState; |}>;

// Return error explicitly
function divide(int a, int b) returns float|error {
    if b == 0 {
        return error("Division by zero");
    }
    return <float>a / <float>b;
}
```

### Nil handling
```ballerina
string? name = ();          // nil
string val = name ?: "default";  // Elvis operator
if name is string {         // type narrowing
    // name is string here
}
```

## HTTP Services

```ballerina
import ballerina/http;

service /api on new http:Listener(8080) {
    resource function get users() returns json {
        return [{id: 1, name: "Alice"}];
    }

    resource function post users(@http:Payload Person body)
            returns http:Created|http:BadRequest {
        // handle creation
        return http:CREATED;
    }

    resource function get users/[int id]() returns Person|http:NotFound {
        // path param
        return {name: "Alice", age: 30};
    }
}
```

## HTTP Client

```ballerina
import ballerina/http;

http:Client apiClient = check new ("https://api.example.com");

function fetchUser(int id) returns json|error {
    json response = check apiClient->get("/users/" + id.toString());
    return response;
}
```

For more integration patterns (databases, Kafka, GraphQL, gRPC), see `references/integration-patterns.md`.

## Testing

```ballerina
import ballerina/test;

@test:Config {}
function testDivide() {
    float|error result = divide(10, 2);
    test:assertTrue(result is float);
    test:assertEquals(result, 5.0);
}

@test:Config {dependsOn: [testDivide]}
function testDivideByZero() {
    float|error result = divide(10, 0);
    test:assertTrue(result is error);
}
```

## Common Compiler Errors and Fixes

Refer to `references/debugging.md` for a detailed list of Ballerina compiler errors, runtime errors, and their fixes.

Quick reference:
- `incompatible types: expected 'X', found 'Y'` → type mismatch; cast with `<X>value` or fix the type
- `variable 'x' is not initialized` → assign before use or use `?` for optional
- `undefined module 'X'` → add import or `bal pull org/module`
- `missing required field 'x' in record 'Y'` → provide the field or make it optional (`x?`)
- `check` in a function that doesn't return `error` → add `|error` to return type
