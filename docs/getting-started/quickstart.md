# Quick Start

Get protovalidate running in your project in under five minutes.

## Prerequisites

- [Buf CLI](https://buf.build/docs/installation) installed
- An existing Protobuf project with `buf.yaml`

## Step 1: Add the Dependency

Add protovalidate to your `buf.yaml`:

```yaml title="buf.yaml"
version: v2
modules:
  - path: proto
deps:
  - buf.build/bufbuild/protovalidate  # (1)!
lint:
  use:
    - STANDARD
breaking:
  use:
    - FILE
```

1.  This pulls the protovalidate annotations from the Buf Schema Registry.

Update your dependencies:

```bash
buf dep update
```

## Step 2: Import and Annotate

Add validation rules to your `.proto` file:

```protobuf title="proto/user/v1/user.proto"
syntax = "proto3";

package user.v1;

import "buf/validate/validate.proto";  // (1)!

message CreateUserRequest {
  // Email must be valid format, max 255 chars
  string email = 1 [(buf.validate.field).string = {
    email: true,
    max_len: 255
  }];
  
  // Username: 3-32 chars, alphanumeric + underscore
  string username = 2 [(buf.validate.field).string = {
    min_len: 3,
    max_len: 32,
    pattern: "^[a-zA-Z0-9_]+$"
  }];
  
  // Age must be positive and reasonable
  uint32 age = 3 [(buf.validate.field).uint32.lte = 150];
}
```

1.  Always import `buf/validate/validate.proto` to access validation annotations.

## Step 3: Generate Code

Run code generation as usual:

```bash
buf generate
```

The validation rules are embedded in the generated descriptors—no additional code generation step required.

## Step 4: Validate at Runtime

=== "Go"

    ```go
    import (
        "fmt"
        "buf.build/go/protovalidate"
        userpb "your/gen/user/v1"
    )
    
    func main() {
        req := &userpb.CreateUserRequest{
            Email:    "invalid-email",  // Will fail validation
            Username: "ab",             // Too short
            Age:      200,              // Exceeds max
        }
        
        validator, err := protovalidate.New()
        if err != nil {
            panic(err)
        }
        
        if err := validator.Validate(req); err != nil {
            fmt.Println("Validation failed:", err)
        }
    }
    ```
    
    Install with:
    ```bash
    go get buf.build/go/protovalidate
    ```

=== "Python"

    ```python
    from protovalidate import validate
    from user.v1 import user_pb2
    
    req = user_pb2.CreateUserRequest(
        email="invalid-email",
        username="ab",
        age=200
    )
    
    try:
        validate(req)
    except Exception as e:
        print(f"Validation failed: {e}")
    ```
    
    Install with:
    ```bash
    pip install protovalidate
    ```

=== "Java"

    ```java
    import build.buf.protovalidate.Validator;
    import build.buf.protovalidate.ValidationResult;
    
    Validator validator = new Validator();
    ValidationResult result = validator.validate(request);
    
    if (!result.isSuccess()) {
        System.out.println("Validation failed: " + result.getViolations());
    }
    ```

## What's Next?

Now that you have protovalidate running, explore common patterns:

- [String validation patterns](../patterns/strings.md) – Email, UUID, regex
- [Numeric constraints](../patterns/numbers.md) – Ranges and bounds
- [Custom CEL rules](../patterns/custom-cel.md) – Cross-field validation
