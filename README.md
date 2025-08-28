# Xarxa Engine WIT contract

### Package Structure.

For small projects
```
wit/
└── world.wit  
```

For medium projects
```
wit/
├── world.wit       # only world definition
├── types.wit       # common types
└── interfaces.wit  # all interfaces
```

For big projects
```
wit/
├── world.wit
├── types/
│   ├── common.wit
│   └── domain.wit
└── interfaces/
    ├── database.wit
    ├── http.wit
    └── auth.wit
```


### Architectural Principles for WIT Development.

#### Core Architectural Principles.

1. Domain Separation:
```wit
// Don't mix different domains in one interface
interface user-management { ... }    // ✅
interface database-operations { ... } // ✅
interface everything { ... }          // ❌
```

2. Minimal Interfaces:
```wit
// Better to have several small interfaces
interface user-auth { login: func(...); }
interface user-profile { get-profile: func(...); }

// Than one large interface
interface user { login: func(...); get-profile: func(...); /* +20 functions */ }
```

3. Explicit Dependencies:
```wit
world my-app {
    // Explicitly specify what you import
    import wasi:filesystem/types;
    import wasi:http/types;
    
    // Don't do "import everything"
}
```

4. Naming Conventions:
```wit
// Use kebab-case for everything
interface http-client { ... }    // ✅
type user-id = u64;             // ✅
my-function: func();            // ✅

// Don't use CamelCase or snake_case  
interface HttpClient { ... }    // ❌
type user_id = u64;            // ❌
```

5. Code Reusability:
```wit
// Extract common types into separate interfaces
interface common-types {
    type timestamp = u64;
    type user-id = string;
}

interface auth {
    use common-types.{user-id, timestamp};
    // ...
}
```

6. Interface Cohesion:
```
// High cohesion - related functionality together
interface file-operations {
    read-file: func(path: string) -> result<string, error>;
    write-file: func(path: string, content: string) -> result<_, error>;
    delete-file: func(path: string) -> result<_, error>;
}

// Low cohesion - unrelated functions
interface mixed-utilities {
    read-file: func(...);
    send-email: func(...);    // ❌ Different domain
    calculate-tax: func(...); // ❌ Different domain
}
```

7. Stable Public APIs:
```wit
// Design for extensibility
interface user-service {
    // Core functionality - stable
    get-user: func(id: user-id) -> result<user, error>;
    
    // Optional extensions can be added later
    // get-user-preferences: func(id: user-id) -> result<preferences, error>;
}
```

8. Error Handling Consistency:
```wit
// Consistent error handling across interfaces
variant result<T, E> {
    ok(T),
    err(E),
}

interface database {
    get-record: func(id: u64) -> result<record, db-error>;
    insert-record: func(data: record) -> result<u64, db-error>;
}
```