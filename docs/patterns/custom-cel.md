# Custom CEL Rules

When standard rules aren't enough, protovalidate supports custom validation logic using Google's Common Expression Language (CEL). CEL provides a safe, fast expression language for defining complex constraints.

## CEL Basics

CEL expressions in protovalidate have access to:

- `this` - The current field value (in field rules) or message (in message rules)
- `rules` - The rule configuration values
- Standard CEL functions and operators

## Field-Level CEL

Add custom validation to individual fields:

```protobuf
message Product {
  // SKU must be uppercase alphanumeric
  string sku = 1 [(buf.validate.field).cel = {
    id: "sku.format",
    message: "SKU must be uppercase letters and numbers only",
    expression: "this.matches('^[A-Z0-9]+$')"
  }];
}
```

## Multiple CEL Rules

Apply multiple custom rules to one field:

```protobuf
message Username {
  string value = 1 [
    // Rule 1: No consecutive underscores
    (buf.validate.field).cel = {
      id: "username.no_consecutive_underscores",
      message: "username cannot contain consecutive underscores",
      expression: "!this.contains('__')"
    },
    // Rule 2: Cannot start or end with underscore
    (buf.validate.field).cel = {
      id: "username.underscore_position",
      message: "username cannot start or end with underscore",
      expression: "!this.startsWith('_') && !this.endsWith('_')"
    }
  ];
}
```

## String Operations

CEL provides rich string manipulation:

```protobuf
message ContentPolicy {
  // No profanity (simplified example)
  string content = 1 [(buf.validate.field).cel = {
    id: "content.clean",
    message: "content contains prohibited words",
    expression: "!this.contains('badword')"
  }];
  
  // Must contain at least one hashtag
  string social_post = 2 [(buf.validate.field).cel = {
    id: "post.has_hashtag",
    message: "post must include at least one hashtag",
    expression: "this.contains('#')"
  }];
  
  // Trim and check non-empty
  string title = 3 [(buf.validate.field).cel = {
    id: "title.not_blank",
    message: "title cannot be blank or whitespace only",
    expression: "this.trim().size() > 0"
  }];
}
```

## Numeric Expressions

Complex numeric validation:

```protobuf
message OrderLine {
  int32 quantity = 1;
  int64 unit_price_cents = 2;
  int64 total_cents = 3;
  
  // Verify total calculation
  option (buf.validate.message).cel = {
    id: "order.total_correct",
    message: "total_cents must equal quantity * unit_price_cents",
    expression: "this.total_cents == this.quantity * this.unit_price_cents"
  };
}
```

## List Operations

Validate repeated fields with CEL:

```protobuf
message TaggedItem {
  // All tags must be lowercase
  repeated string tags = 1 [(buf.validate.field).cel = {
    id: "tags.lowercase",
    message: "all tags must be lowercase",
    expression: "this.all(tag, tag == tag.lowerAscii())"
  }];
  
  // At least one priority tag
  repeated string labels = 2 [(buf.validate.field).cel = {
    id: "labels.has_priority",
    message: "must include at least one priority label (P0, P1, P2)",
    expression: "this.exists(l, l in ['P0', 'P1', 'P2'])"
  }];
}
```

### List Quantifiers

| Function | Description | Example |
|----------|-------------|---------|
| `all(x, expr)` | All elements satisfy condition | `this.all(t, t.size() > 0)` |
| `exists(x, expr)` | At least one satisfies | `this.exists(t, t == 'urgent')` |
| `exists_one(x, expr)` | Exactly one satisfies | `this.exists_one(t, t == 'primary')` |

## Map Operations

Validate map fields:

```protobuf
message Configuration {
  // All keys must be lowercase
  map<string, string> settings = 1 [(buf.validate.field).cel = {
    id: "settings.lowercase_keys",
    message: "all setting keys must be lowercase",
    expression: "this.all(key, key == key.lowerAscii())"
  }];
  
  // Must include 'version' key
  map<string, string> metadata = 2 [(buf.validate.field).cel = {
    id: "metadata.has_version",
    message: "metadata must include 'version' key",
    expression: "'version' in this"
  }];
}
```

## Conditional Validation

Different rules based on field values:

```protobuf
message Notification {
  enum Channel {
    CHANNEL_UNSPECIFIED = 0;
    CHANNEL_EMAIL = 1;
    CHANNEL_SMS = 2;
  }
  
  Channel channel = 1;
  string recipient = 2;
  
  // If email channel, recipient must be valid email format
  option (buf.validate.message).cel = {
    id: "notification.valid_recipient",
    message: "email channel requires valid email address",
    expression: "this.channel != 1 || this.recipient.contains('@')"
  };
}
```

## Dynamic Error Messages

Include field values in error messages using `format()`:

```protobuf
message BoundedValue {
  int32 value = 1 [(buf.validate.field).cel = {
    id: "value.in_range",
    expression: "this >= 0 && this <= 100",
    message: "value must be between 0 and 100, got %s".format([this])
  }];
}
```

## Timestamp Validation

Validate temporal constraints:

```protobuf
import "google/protobuf/timestamp.proto";

message Event {
  google.protobuf.Timestamp scheduled_at = 1 [(buf.validate.field).cel = {
    id: "event.future_date",
    message: "scheduled_at must be in the future",
    expression: "this > now"
  }];
}
```

## Duration Validation

```protobuf
import "google/protobuf/duration.proto";

message RateLimitConfig {
  google.protobuf.Duration window = 1 [(buf.validate.field).cel = {
    id: "window.reasonable",
    message: "window must be between 1 second and 1 hour",
    expression: "this >= duration('1s') && this <= duration('1h')"
  }];
}
```

## CEL Function Reference

### String Functions

| Function | Example | Description |
|----------|---------|-------------|
| `contains(s)` | `this.contains('x')` | Substring check |
| `startsWith(s)` | `this.startsWith('http')` | Prefix check |
| `endsWith(s)` | `this.endsWith('.json')` | Suffix check |
| `matches(regex)` | `this.matches('^[a-z]+$')` | Regex match |
| `size()` | `this.size() > 0` | Character count |
| `lowerAscii()` | `this.lowerAscii()` | Lowercase |
| `upperAscii()` | `this.upperAscii()` | Uppercase |
| `trim()` | `this.trim()` | Remove whitespace |

### Comparison Operators

| Operator | Example |
|----------|---------|
| `==`, `!=` | `this == 'expected'` |
| `<`, `<=`, `>`, `>=` | `this.size() >= 10` |
| `in` | `this in ['a', 'b', 'c']` |
| `&&`, `||`, `!` | `this > 0 && this < 100` |

### Type Checks

| Function | Example |
|----------|---------|
| `type(x)` | `type(this) == string` |
| `has(field)` | `has(this.optional_field)` |

## Best Practices

1. **Keep expressions simple**: Complex logic is hard to debug
2. **Use descriptive IDs**: `user.email.corporate_domain` not `check1`
3. **Write helpful messages**: Tell users what's wrong and how to fix it
4. **Test edge cases**: Empty values, nulls, boundary conditions
5. **Combine with standard rules**: Use CEL for logic that standard rules can't express
