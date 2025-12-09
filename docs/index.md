# Protovalidate Patterns Cookbook

A practical guide to implementing validation patterns with [protovalidate](https://github.com/bufbuild/protovalidate), Buf's runtime validation library for Protocol Buffers.

---

## Why This Cookbook?

Protovalidate lets you define validation rules directly in your `.proto` files using annotations. But knowing the syntax is only half the battle—knowing *which patterns to use* and *when* makes the difference between maintainable schemas and validation spaghetti.

This cookbook provides battle-tested patterns for common validation scenarios.

## What You'll Learn

<div class="grid cards" markdown>

-   :material-format-text:{ .lg .middle } **String Validation**

    ---

    Email, UUID, URL, and custom regex patterns with clear error messages.

    [:octicons-arrow-right-24: String patterns](patterns/strings.md)

-   :material-numeric:{ .lg .middle } **Numeric Constraints**

    ---

    Ranges, bounds, and business logic for integers and floats.

    [:octicons-arrow-right-24: Number patterns](patterns/numbers.md)

-   :material-format-list-bulleted:{ .lg .middle } **Enum Validation**

    ---

    Defined-only checks and value restrictions for type-safe enums.

    [:octicons-arrow-right-24: Enum patterns](patterns/enums.md)

-   :material-code-braces:{ .lg .middle } **Custom CEL Rules**

    ---

    Cross-field validation using Common Expression Language.

    [:octicons-arrow-right-24: CEL patterns](patterns/custom-cel.md)

</div>

## Quick Example

```protobuf
syntax = "proto3";

import "buf/validate/validate.proto";

message User {
  // UUID format validation
  string id = 1 [(buf.validate.field).string.uuid = true];
  
  // Email with max length
  string email = 2 [(buf.validate.field).string = {
    email: true,
    max_len: 255
  }];
  
  // Age must be reasonable
  uint32 age = 3 [(buf.validate.field).uint32 = {
    gte: 0,
    lte: 150
  }];
}
```

## Who Is This For?

This cookbook assumes familiarity with Protocol Buffers. If you're new to Protobuf, start with the [official Protocol Buffers documentation](https://protobuf.dev/).

For comprehensive protovalidate reference documentation, see [buf.build/docs/protovalidate](https://buf.build/docs/protovalidate/).

---

*Built with [MkDocs](https://www.mkdocs.org/) and [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/).*
