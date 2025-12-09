# Numeric Constraints

Protovalidate provides comprehensive rules for validating integer and floating-point fields, from simple bounds to complex business logic.

## Basic Comparisons

All numeric types support standard comparison operators:

```protobuf
message Product {
  // Greater than zero (exclusive)
  int32 stock = 1 [(buf.validate.field).int32.gt = 0];
  
  // Greater than or equal (inclusive)
  uint32 min_order = 2 [(buf.validate.field).uint32.gte = 1];
  
  // Less than
  double discount = 3 [(buf.validate.field).double.lt = 1.0];
  
  // Less than or equal
  float rating = 4 [(buf.validate.field).float.lte = 5.0];
}
```

| Operator | Meaning | Example |
|----------|---------|---------|
| `gt` | Greater than (exclusive) | `gt = 0` → value > 0 |
| `gte` | Greater than or equal | `gte = 1` → value ≥ 1 |
| `lt` | Less than (exclusive) | `lt = 100` → value < 100 |
| `lte` | Less than or equal | `lte = 100` → value ≤ 100 |

## Ranges

Combine operators for range validation:

```protobuf
message User {
  // Age between 0 and 150 (inclusive)
  uint32 age = 1 [(buf.validate.field).uint32 = {
    gte: 0,
    lte: 150
  }];
}
```

```protobuf
message Sensor {
  // Temperature between -40 and 85 Celsius
  float temperature_c = 1 [(buf.validate.field).float = {
    gte: -40.0,
    lte: 85.0
  }];
}
```

!!! tip "Exclusive vs. Inclusive"
    Use `gt`/`lt` for exclusive bounds and `gte`/`lte` for inclusive bounds:
    
    - `gte: 0, lte: 100` → 0 ≤ value ≤ 100
    - `gt: 0, lt: 100` → 0 < value < 100

## Exact Values

Require a specific value with `const`:

```protobuf
message ApiVersion {
  // Must be exactly 2
  int32 version = 1 [(buf.validate.field).int32.const = 2];
}
```

## Allowed and Denied Values

Restrict to a set of values:

```protobuf
message Config {
  // Only allow specific ports
  uint32 port = 1 [(buf.validate.field).uint32 = {
    in: [80, 443, 8080, 8443]
  }];
  
  // Block reserved ports
  uint32 custom_port = 2 [(buf.validate.field).uint32 = {
    not_in: [0, 22, 23, 25]
  }];
}
```

## Integer Types

Rules are type-specific. Use the correct type for your field:

```protobuf
message TypeExamples {
  int32 signed_32 = 1 [(buf.validate.field).int32.gte = -100];
  int64 signed_64 = 2 [(buf.validate.field).int64.lte = 9223372036854775807];
  uint32 unsigned_32 = 3 [(buf.validate.field).uint32.gt = 0];
  uint64 unsigned_64 = 4 [(buf.validate.field).uint64.lt = 1000000];
  sint32 signed_int = 5 [(buf.validate.field).sint32.gte = 0];
  fixed32 fixed = 6 [(buf.validate.field).fixed32.lte = 255];
  sfixed64 signed_fixed = 7 [(buf.validate.field).sfixed64.gt = -1000];
}
```

## Floating Point

Float and double fields support the same operators:

```protobuf
message Measurement {
  // Latitude: -90 to 90
  double latitude = 1 [(buf.validate.field).double = {
    gte: -90.0,
    lte: 90.0
  }];
  
  // Longitude: -180 to 180
  double longitude = 2 [(buf.validate.field).double = {
    gte: -180.0,
    lte: 180.0
  }];
  
  // Percentage: 0 to 1
  float percentage = 3 [(buf.validate.field).float = {
    gte: 0.0,
    lte: 1.0
  }];
}
```

!!! warning "Floating Point Precision"
    Be cautious with exact comparisons on floats due to precision issues. Use ranges where possible.

## Business Logic Examples

### Pricing

```protobuf
message PriceUpdate {
  // Price must be positive
  int64 price_cents = 1 [(buf.validate.field).int64.gt = 0];
  
  // Discount percentage: 0-100
  uint32 discount_percent = 2 [(buf.validate.field).uint32 = {
    gte: 0,
    lte: 100
  }];
}
```

### Pagination

```protobuf
message ListRequest {
  // Page size: 1-100 items
  uint32 page_size = 1 [(buf.validate.field).uint32 = {
    gte: 1,
    lte: 100
  }];
  
  // Page number: starts at 1
  uint32 page = 2 [(buf.validate.field).uint32.gte = 1];
}
```

### Time-Based

```protobuf
message ScheduleRequest {
  // Unix timestamp must be in the future (use CEL for dynamic validation)
  int64 scheduled_at = 1 [(buf.validate.field).int64.gt = 0];
  
  // TTL in seconds: 1 minute to 1 year
  uint32 ttl_seconds = 2 [(buf.validate.field).uint32 = {
    gte: 60,
    lte: 31536000
  }];
}
```

### Quantity Limits

```protobuf
message OrderItem {
  string product_id = 1 [(buf.validate.field).string.uuid = true];
  
  // Quantity: 1-999
  uint32 quantity = 2 [(buf.validate.field).uint32 = {
    gte: 1,
    lte: 999
  }];
}
```

## Quick Reference

| Use Case | Pattern |
|----------|---------|
| Positive integer | `gt = 0` or `gte = 1` |
| Non-negative | `gte = 0` |
| Percentage (0-100) | `gte = 0, lte = 100` |
| Percentage (0-1) | `gte = 0.0, lte = 1.0` |
| Valid port | `gte = 1, lte = 65535` |
| HTTP status | `gte = 100, lte = 599` |
