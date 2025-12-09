# Message-Level Rules

Message-level rules validate entire messages rather than individual fields. They're essential for cross-field validation where the validity of one field depends on another.

## When to Use Message Rules

Use message-level rules when:

- Two or more fields must be validated together
- A field's validity depends on another field's value
- You need to enforce business logic across multiple fields

## Basic Syntax

Message rules use `option (buf.validate.message)` on the message itself:

```protobuf
message DateRange {
  int64 start_date = 1;
  int64 end_date = 2;
  
  option (buf.validate.message).cel = {
    id: "date_range.end_after_start",
    message: "end_date must be after start_date",
    expression: "this.end_date > this.start_date"
  };
}
```

## Cross-Field Validation

### Conditional Requirements

Require one field when another is present:

```protobuf
message User {
  string first_name = 1;
  string last_name = 2;
  
  // If first_name is set, last_name must also be set
  option (buf.validate.message).cel = {
    id: "name.complete",
    message: "last_name is required when first_name is provided",
    expression: "!has(this.first_name) || has(this.last_name)"
  };
}
```

### Mutual Exclusion

Ensure only one of several fields is set:

```protobuf
message PaymentMethod {
  string credit_card = 1;
  string bank_account = 2;
  string crypto_wallet = 3;
  
  option (buf.validate.message).cel = {
    id: "payment.single_method",
    message: "exactly one payment method must be provided",
    expression: "(has(this.credit_card) ? 1 : 0) + (has(this.bank_account) ? 1 : 0) + (has(this.crypto_wallet) ? 1 : 0) == 1"
  };
}
```

### At Least One Required

Require at least one of several optional fields:

```protobuf
message ContactInfo {
  string email = 1;
  string phone = 2;
  string address = 3;
  
  option (buf.validate.message).cel = {
    id: "contact.at_least_one",
    message: "at least one contact method is required",
    expression: "has(this.email) || has(this.phone) || has(this.address)"
  };
}
```

## Numeric Relationships

### Range Validation

```protobuf
message PriceRange {
  int64 min_price = 1;
  int64 max_price = 2;
  
  option (buf.validate.message).cel = {
    id: "price.valid_range",
    message: "max_price must be greater than or equal to min_price",
    expression: "this.max_price >= this.min_price"
  };
}
```

### Percentage Totals

```protobuf
message AllocationSplit {
  uint32 engineering_percent = 1;
  uint32 marketing_percent = 2;
  uint32 operations_percent = 3;
  
  option (buf.validate.message).cel = {
    id: "allocation.total_100",
    message: "allocation percentages must total 100",
    expression: "this.engineering_percent + this.marketing_percent + this.operations_percent == 100"
  };
}
```

## Combined Field Length

```protobuf
message UserProfile {
  string first_name = 1 [(buf.validate.field).string.max_len = 50];
  string last_name = 2 [(buf.validate.field).string.max_len = 50];
  
  // Combined length check
  option (buf.validate.message).cel = {
    id: "name.combined_length",
    message: "first_name and last_name combined cannot exceed 100 characters",
    expression: "this.first_name.size() + this.last_name.size() <= 100"
  };
}
```

## Different Fields, Same Value

Prevent duplicate values:

```protobuf
message MoneyTransfer {
  string from_account_id = 1 [(buf.validate.field).string.uuid = true];
  string to_account_id = 2 [(buf.validate.field).string.uuid = true];
  
  option (buf.validate.message).cel = {
    id: "transfer.different_accounts",
    message: "from_account_id and to_account_id must be different",
    expression: "this.from_account_id != this.to_account_id"
  };
}
```

## Multiple Message Rules

You can have multiple validation rules on a single message:

```protobuf
message EventSchedule {
  int64 start_time = 1;
  int64 end_time = 2;
  int64 registration_deadline = 3;
  
  // Rule 1: End after start
  option (buf.validate.message).cel = {
    id: "schedule.end_after_start",
    message: "end_time must be after start_time",
    expression: "this.end_time > this.start_time"
  };
  
  // Rule 2: Registration before start
  option (buf.validate.message).cel = {
    id: "schedule.registration_before_start",
    message: "registration_deadline must be before start_time",
    expression: "this.registration_deadline < this.start_time"
  };
}
```

## Disabling Validation

Skip validation for specific messages (useful during development):

```protobuf
message LegacyData {
  option (buf.validate.message).disabled = true;
  
  // No validation applied to any fields
  string data = 1;
}
```

!!! warning "Use sparingly"
    Disabling validation should be temporary. Consider migrating legacy data to validated messages.

## CEL Expression Reference

Common CEL functions for message rules:

| Function | Description | Example |
|----------|-------------|---------|
| `has(field)` | Check if optional field is set | `has(this.email)` |
| `size()` | String length or list size | `this.name.size() > 0` |
| `matches()` | Regex match | `this.code.matches("^[A-Z]+$")` |
| `startsWith()` | String prefix | `this.url.startsWith("https")` |
| `contains()` | Substring check | `this.tags.contains("urgent")` |

## Best Practices

1. **Use descriptive IDs**: `id: "order.valid_date_range"` not `id: "check1"`
2. **Write clear messages**: Error messages should explain what's wrong and how to fix it
3. **Combine with field rules**: Use message rules for cross-field logic, field rules for individual constraints
4. **Test edge cases**: Empty strings, zero values, and boundary conditions
