# Enum Validation Patterns

Enums in Protocol Buffers have a quirk: they accept any integer value, even ones not defined in the schema. Protovalidate helps enforce enum integrity at runtime.

## The Problem

Consider this enum:

```protobuf
enum Status {
  STATUS_UNSPECIFIED = 0;
  STATUS_PENDING = 1;
  STATUS_ACTIVE = 2;
  STATUS_INACTIVE = 3;
}

message User {
  Status status = 1;
}
```

Without validation, a client can send `status = 99`—and Protobuf will happily accept it. This can cause unexpected behavior downstream.

## Defined Values Only

Use `defined_only` to reject undefined enum values:

```protobuf
message User {
  Status status = 1 [(buf.validate.field).enum.defined_only = true];
}
```

Now only `0`, `1`, `2`, or `3` are accepted.

!!! tip "Always use defined_only"
    This should be your default for enum fields. Undefined values are almost never intentional.

## Excluding Unspecified

The `0` value in enums typically represents "unspecified" and often shouldn't be allowed in requests:

```protobuf
message CreateUserRequest {
  // Must be a defined value AND not zero
  Status status = 1 [(buf.validate.field).enum = {
    defined_only: true,
    not_in: [0]
  }];
}
```

Alternative approach using `in`:

```protobuf
message CreateUserRequest {
  // Only allow specific values
  Status status = 1 [(buf.validate.field).enum = {
    in: [1, 2, 3]  // Excludes STATUS_UNSPECIFIED
  }];
}
```

## Allowed Values

Restrict to a subset of defined values:

```protobuf
enum OrderStatus {
  ORDER_STATUS_UNSPECIFIED = 0;
  ORDER_STATUS_DRAFT = 1;
  ORDER_STATUS_PENDING = 2;
  ORDER_STATUS_CONFIRMED = 3;
  ORDER_STATUS_SHIPPED = 4;
  ORDER_STATUS_DELIVERED = 5;
  ORDER_STATUS_CANCELLED = 6;
}

message UpdateOrderRequest {
  string order_id = 1 [(buf.validate.field).string.uuid = true];
  
  // Can only transition to these states
  OrderStatus new_status = 2 [(buf.validate.field).enum = {
    in: [3, 4, 5, 6]  // CONFIRMED, SHIPPED, DELIVERED, CANCELLED
  }];
}
```

## Denied Values

Block specific enum values:

```protobuf
message OrderQuery {
  // Cannot query cancelled orders through this endpoint
  OrderStatus status_filter = 1 [(buf.validate.field).enum = {
    defined_only: true,
    not_in: [6]  // Block ORDER_STATUS_CANCELLED
  }];
}
```

## Repeated Enums

Validate collections of enum values:

```protobuf
message NotificationPreferences {
  enum Channel {
    CHANNEL_UNSPECIFIED = 0;
    CHANNEL_EMAIL = 1;
    CHANNEL_SMS = 2;
    CHANNEL_PUSH = 3;
  }
  
  // At least one channel, all must be defined
  repeated Channel channels = 1 [(buf.validate.field) = {
    repeated: {
      min_items: 1,
      unique: true,
      items: {
        enum: {
          defined_only: true,
          not_in: [0]
        }
      }
    }
  }];
}
```

## Practical Patterns

### Required Enum (No Default)

```protobuf
message Request {
  enum Priority {
    PRIORITY_UNSPECIFIED = 0;
    PRIORITY_LOW = 1;
    PRIORITY_MEDIUM = 2;
    PRIORITY_HIGH = 3;
  }
  
  // Must explicitly set a priority
  Priority priority = 1 [(buf.validate.field).enum = {
    defined_only: true,
    not_in: [0]
  }];
}
```

### Optional Enum (Allow Default)

```protobuf
message Query {
  enum SortOrder {
    SORT_ORDER_UNSPECIFIED = 0;  // Will use server default
    SORT_ORDER_ASC = 1;
    SORT_ORDER_DESC = 2;
  }
  
  // 0 is valid (means "use default")
  SortOrder sort = 1 [(buf.validate.field).enum.defined_only = true];
}
```

### State Machine Transitions

```protobuf
message TaskUpdate {
  enum TaskState {
    TASK_STATE_UNSPECIFIED = 0;
    TASK_STATE_TODO = 1;
    TASK_STATE_IN_PROGRESS = 2;
    TASK_STATE_REVIEW = 3;
    TASK_STATE_DONE = 4;
    TASK_STATE_ARCHIVED = 5;
  }
  
  string task_id = 1;
  
  // Only forward transitions allowed
  TaskState new_state = 2 [(buf.validate.field).enum = {
    in: [2, 3, 4, 5]  // Cannot go back to TODO
  }];
}
```

## Quick Reference

| Use Case | Pattern |
|----------|---------|
| Only defined values | `defined_only: true` |
| Required (non-zero) | `defined_only: true, not_in: [0]` |
| Specific allowed values | `in: [1, 2, 3]` |
| Block specific values | `defined_only: true, not_in: [4, 5]` |

!!! note "Enum values are integers"
    In `in` and `not_in`, use the integer values, not the enum names.
