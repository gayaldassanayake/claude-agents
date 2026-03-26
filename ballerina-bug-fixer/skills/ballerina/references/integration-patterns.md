# Ballerina Integration Patterns

## Database (SQL)

### MySQL / PostgreSQL
```ballerina
import ballerinax/mysql;
import ballerina/sql;

mysql:Client dbClient = check new (
    host = "localhost",
    port = 3306,
    user = "root",
    password = "secret",
    database = "mydb"
);

// Query returning a stream
stream<record {int id; string name;}, sql:Error?> resultStream =
    dbClient->query(`SELECT id, name FROM users`);

// Query single row
record {int id; string name;}|sql:Error row =
    dbClient->queryRow(`SELECT id, name FROM users WHERE id = ${userId}`);

// Execute (insert/update/delete)
sql:ExecutionResult result = check dbClient->execute(
    `INSERT INTO users (name, email) VALUES (${name}, ${email})`
);
int lastInsertId = <int>result.lastInsertId;

// Transaction
transaction {
    check dbClient->execute(`UPDATE accounts SET balance = balance - ${amount} WHERE id = ${from}`);
    check dbClient->execute(`UPDATE accounts SET balance = balance + ${amount} WHERE id = ${to}`);
    check commit;
}
```

Add to `Ballerina.toml`:
```toml
[[dependency]]
org = "ballerinax"
name = "mysql"
version = "1.x.x"
```

---

## Kafka

### Producer
```ballerina
import ballerinax/kafka;

kafka:Producer producer = check new (kafka:DEFAULT_URL);

check producer->send({
    topic: "orders",
    value: orderJson.toJsonString().toBytes()
});
check producer->close();
```

### Consumer (service)
```ballerina
import ballerinax/kafka;

service on new kafka:Listener({
    bootstrapServers: "localhost:9092",
    groupId: "order-group",
    topics: ["orders"]
}) {
    remote function onConsumerRecord(kafka:Caller caller,
                                     kafka:BytesConsumerRecord[] records) returns error? {
        foreach kafka:BytesConsumerRecord rec in records {
            string message = check string:fromBytes(rec.value);
            // process message
        }
    }
}
```

---

## GraphQL

### Service
```ballerina
import ballerina/graphql;

service /graphql on new graphql:Listener(4000) {
    resource function get user(int id) returns User|error {
        return fetchUser(id);
    }

    remote function createUser(string name, string email) returns User|error {
        return insertUser(name, email);
    }
}
```

---

## gRPC

### Server
```ballerina
// After running: bal grpc --input hello.proto --output gen/
import ballerina/grpc;

@grpc:ServiceDescriptor {descriptor: ROOT_DESCRIPTOR, descMap: getDescriptorMap()}
service "HelloService" on new grpc:Listener(9090) {
    remote function sayHello(HelloRequest value) returns HelloResponse|error {
        return {message: "Hello, " + value.name};
    }
}
```

### Client
```ballerina
HelloServiceClient grpcClient = check new ("http://localhost:9090");
HelloResponse response = check grpcClient->sayHello({name: "World"});
```

---

## File I/O

```ballerina
import ballerina/io;

// Read text
string content = check io:fileReadString("data.txt");

// Read JSON
json data = check io:fileReadJson("config.json");

// Write
check io:fileWriteString("output.txt", "Hello");
check io:fileWriteJson("result.json", {status: "ok"});

// CSV
string[][] csvData = check io:fileReadCsv("data.csv");
```

---

## Concurrency Patterns

### Named workers
```ballerina
function processAsync() returns error? {
    worker w1 {
        // runs concurrently
        doTaskA();
    }
    worker w2 {
        doTaskB();
    }
    // wait for both
    check wait w1;
    check wait w2;
}
```

### Start (fire and forget / async)
```ballerina
future<int> f = start expensiveCalc();
// ... do other work ...
int result = check wait f;
```

### Lock (shared mutable state)
```ballerina
int counter = 0;
lock {
    counter += 1;
}
```

---

## Environment Variables and Config

```ballerina
import ballerina/os;

string host = os:getEnv("DB_HOST");
// Returns "" if not set; use ?: for default
string port = os:getEnv("DB_PORT") ?: "5432";
```

### Configurable values (Ballerina config)
```ballerina
configurable string dbHost = "localhost";
configurable int dbPort = 5432;
configurable string dbPassword = ?;  // required, no default
```

Set via `Config.toml` or environment variables:
```toml
dbHost = "prod-db.example.com"
dbPassword = "secret"
```
Or: `BAL_CONFIG_VAR_dbPassword=secret bal run`

---

## HTTP Interceptors and Middleware

```ballerina
import ballerina/http;

service class AuthInterceptor {
    *http:RequestInterceptor;
    resource function 'default [string... path](http:RequestContext ctx,
                                                 http:Request req)
            returns http:NextService|error? {
        string|error token = req.getHeader("Authorization");
        if token is error {
            return error http:Unauthorized();
        }
        return ctx.next();
    }
}

@http:ServiceConfig {interceptors: [new AuthInterceptor()]}
service /api on new http:Listener(8080) { ... }
```
