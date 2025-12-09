# What is Protovalidate?

Protovalidate is a runtime validation library for Protocol Buffers that lets you define validation rules directly in your `.proto` files. It's the successor to `protoc-gen-validate` and is powered by Google's Common Expression Language (CEL).

## The Problem It Solves

Protocol Buffers guarantee *type safety*—a field declared as `int32` will always be an integer. But they can't enforce *semantic rules*:

- Is this string a valid email address?
- Is this number within an acceptable range?
- Do these two fields have a valid relationship?

Without protovalidate, you'd write validation logic separately in every service that consumes your messages—duplicating code across Go, Java, Python, and TypeScript implementations.

## Schema-Driven Validation

Protovalidate moves validation rules into the schema itself:

```protobuf
message Order {
  string customer_email = 1 [(buf.validate.field).string.email = true];
  
  int32 quantity = 2 [(buf.validate.field).int32 = {
    gte: 1,
    lte: 1000
  }];
}
```

Now every consumer of `Order`—regardless of language—validates against the same rules defined in one place.

## Supported Languages

Protovalidate provides runtime libraries for:

| Language   | Package                          | Status |
|------------|----------------------------------|--------|
| Go         | `buf.build/go/protovalidate`     | Stable |
| Java       | `protovalidate-java`             | Stable |
| Python     | `protovalidate-python`           | Stable |
| C++        | `protovalidate-cc`               | Beta   |
| TypeScript | `protovalidate-es`               | Stable |

## How It Works

1. **Define rules** in your `.proto` files using `buf.validate` annotations
2. **Generate code** with the Buf CLI (rules are embedded in descriptors)
3. **Validate at runtime** using language-specific libraries

```go
// Go example
import "buf.build/go/protovalidate"

func validateOrder(order *orderpb.Order) error {
    validator, _ := protovalidate.New()
    return validator.Validate(order)
}
```

## Key Concepts

**Standard Rules**
:   Built-in validators for common patterns: email, UUID, URL, regex, numeric ranges, and more.

**CEL Expressions**
:   Custom validation logic using Google's Common Expression Language for cross-field validation.

**Message-Level Rules**
:   Constraints that apply to entire messages, enabling validation across multiple fields.

**Field-Level Rules**
:   Constraints that apply to individual fields—the most common type.

## Next Steps

Ready to start? Head to the [Quick Start](quickstart.md) guide to add protovalidate to your project.
