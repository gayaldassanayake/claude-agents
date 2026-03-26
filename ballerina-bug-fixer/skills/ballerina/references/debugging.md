# Ballerina Debugging Reference

## Common Compiler Errors

### Type errors

| Error | Cause | Fix |
|-------|-------|-----|
| `incompatible types: expected 'X', found 'Y'` | Type mismatch | Cast with `<X>value` or fix source type |
| `operator '+' not defined for types 'X' and 'Y'` | Operator on incompatible types | Convert types first: `x.toString()` |
| `invalid usage of check expression` | `check` used in non-error-returning function | Add `\|error` to function return type |
| `missing required field 'x' in record 'Y'` | Record literal missing field | Add the field or mark it optional with `?` |
| `undefined field 'x' in record 'Y'` | Field doesn't exist on type | Check field name spelling or use open record |
| `cannot assign a value to a final 'x'` | Mutating a `final` variable | Remove `final` or use a new variable |
| `unreachable code` | Code after `return`/`panic`/infinite loop | Remove unreachable block |

### Import and module errors

| Error | Cause | Fix |
|-------|-------|-----|
| `undefined module 'X'` | Module not imported or not available | Add import statement; run `bal pull org/module` |
| `no module named 'X' in Ballerina Central` | Typo or wrong org | Check central.ballerina.io |
| `cyclic module dependency detected` | Module A imports B which imports A | Restructure modules |

### Nil and optional errors

| Error | Cause | Fix |
|-------|-------|-----|
| `value will always be nil` | Assigning `()` where non-nil expected | Use `?` type or provide a real value |
| `variable might be nil` | Using optional without nil check | Guard with `if x is T { }` or use `x ?: default` |

### Service and resource errors

| Error | Cause | Fix |
|-------|-------|-----|
| `invalid resource method return type` | Return type not compatible with HTTP spec | Use `http:Ok`, `json`, `record`, or `http:Response` |
| `service path not allowed` | Invalid path segment | Fix path syntax (no leading `/` in segment, use `[string seg]` for params) |

---

## Runtime Errors

### Panic messages and their causes

| Panic | Cause | Fix |
|-------|-------|-----|
| `index out of range` | Array/tuple access beyond bounds | Check length before access |
| `null pointer dereference` (rare) | Accessing field on nil | Guard nil before field access |
| `arithmetic overflow` | Integer overflow (not auto-promoted) | Use `int:MAX_VALUE` check or use `float` |
| `key not found in map` | `map["key"]` where key absent | Use `map["key"]` (returns `T?`) or check with `map.hasKey("key")` |
| `conversion error` | `<int>"abc"` type cast failure | Use `int:fromString(s)` which returns `int|error` |

### Debugging techniques

```ballerina
import ballerina/log;
import ballerina/io;

// Structured logging
log:printInfo("Processing request", id = requestId, user = username);
log:printError("Failed", err = e, context = "database");
log:printDebug("Value", val = someVar.toString());

// Quick print to console (development only)
io:println("Debug: ", value);

// Type inspection
string typeName = (typeof value).toString();

// Check if value is a specific type
if value is http:ClientError {
    log:printError("HTTP error", err = value);
}
```

### Enable debug logging
```bash
BAL_LOG_LEVEL=DEBUG bal run
# or per module:
BAL_LOG_LEVEL=DEBUG BAL_LOG_FILTERS=myorg/mypackage=DEBUG bal run
```

---

## Test Failures

```ballerina
import ballerina/test;

// Assert helpers
test:assertEquals(actual, expected);
test:assertNotEquals(actual, expected);
test:assertTrue(condition);
test:assertFalse(condition);
test:assertFail("optional message");  // force fail

// Test with mock
@test:Mock {functionName: "fetchUser"}
function mockFetchUser(int id) returns User|error {
    return {name: "MockUser", age: 25};
}

// Before/after hooks
@test:BeforeEach
function setup() { ... }

@test:AfterSuite {}
function teardown() { ... }
```

---

## Ballerina Version and Compatibility Issues

```bash
bal version                    # check current distribution
bal dist list                  # list installed distributions
bal dist pull 2201.10.0        # install specific version
bal dist use 2201.10.0         # switch version
```

Check `Ballerina.toml` `distribution` field matches your local version.

---

## Performance Tips

- Prefer `stream` over loading full result sets into memory for large DB queries
- Use `readonly` for immutable records to enable sharing across strands without copying
- Avoid `anydata` / `json` in hot paths — use typed records for better performance
- `lock` blocks are strand-aware; keep them short to avoid blocking other strands
- For CPU-bound work, use named workers to leverage multiple strands
