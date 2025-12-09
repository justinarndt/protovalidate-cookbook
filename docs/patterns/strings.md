# String Validation Patterns

String fields are the most common validation target. Protovalidate provides built-in rules for format validation, length constraints, and pattern matching.

## Email Addresses

The `email` rule validates RFC 5322 email format:

```protobuf
message ContactRequest {
  string email = 1 [(buf.validate.field).string.email = true];
}
```

!!! tip "Combine with length constraints"
    Email addresses can technically be very long. Add `max_len` to prevent abuse:
    
    ```protobuf
    string email = 1 [(buf.validate.field).string = {
      email: true,
      max_len: 255
    }];
    ```

## UUIDs

Validate UUID format (any version):

```protobuf
message Resource {
  string id = 1 [(buf.validate.field).string.uuid = true];
}
```

This accepts UUIDs with or without hyphens:
- ✅ `550e8400-e29b-41d4-a716-446655440000`
- ✅ `550e8400e29b41d4a716446655440000`
- ❌ `not-a-uuid`

## URLs

Validate URI format:

```protobuf
message Webhook {
  string callback_url = 1 [(buf.validate.field).string.uri = true];
}
```

For URLs that must include a host:

```protobuf
string callback_url = 1 [(buf.validate.field).string.uri_ref = true];
```

## Length Constraints

Control string length with `min_len`, `max_len`, and `len`:

```protobuf
message UserProfile {
  // Exact length (e.g., ISO country code)
  string country_code = 1 [(buf.validate.field).string.len = 2];
  
  // Minimum length
  string username = 2 [(buf.validate.field).string.min_len = 3];
  
  // Maximum length
  string bio = 3 [(buf.validate.field).string.max_len = 500];
  
  // Range
  string display_name = 4 [(buf.validate.field).string = {
    min_len: 1,
    max_len: 64
  }];
}
```

!!! note "Length vs. Bytes"
    `min_len` and `max_len` count Unicode code points. For byte length, use `min_bytes` and `max_bytes`.

## Pattern Matching

Use `pattern` for regex validation:

```protobuf
message Account {
  // Alphanumeric with underscores, 3-20 chars
  string username = 1 [(buf.validate.field).string = {
    pattern: "^[a-zA-Z0-9_]{3,20}$"
  }];
  
  // US phone number format
  string phone = 2 [(buf.validate.field).string = {
    pattern: "^\\+1[0-9]{10}$"
  }];
  
  // Semantic version
  string version = 3 [(buf.validate.field).string = {
    pattern: "^v[0-9]+\\.[0-9]+\\.[0-9]+$"
  }];
}
```

!!! warning "Escape backslashes"
    In proto strings, backslashes must be escaped: `\\d` not `\d`.

## Prefix and Suffix

Validate string boundaries without regex:

```protobuf
message S3Object {
  // Must start with "s3://"
  string uri = 1 [(buf.validate.field).string.prefix = "s3://"];
  
  // Must end with ".json"
  string config_file = 2 [(buf.validate.field).string.suffix = ".json"];
  
  // Must contain substring
  string path = 3 [(buf.validate.field).string.contains = "/api/"];
}
```

## Allowed and Denied Values

Restrict to specific values:

```protobuf
message Config {
  // Only allow specific environments
  string environment = 1 [(buf.validate.field).string = {
    in: ["development", "staging", "production"]
  }];
  
  // Block reserved words
  string identifier = 2 [(buf.validate.field).string = {
    not_in: ["admin", "root", "system"]
  }];
}
```

## IP Addresses

Validate IPv4 and IPv6 formats:

```protobuf
message NetworkConfig {
  string ipv4_address = 1 [(buf.validate.field).string.ipv4 = true];
  string ipv6_address = 2 [(buf.validate.field).string.ipv6 = true];
  
  // Either IPv4 or IPv6
  string ip_address = 3 [(buf.validate.field).string.ip = true];
}
```

## Combining Rules

Use message literal syntax to combine multiple constraints cleanly:

```protobuf
message AdvancedUser {
  string username = 1 [(buf.validate.field).string = {
    min_len: 3,
    max_len: 32,
    pattern: "^[a-z][a-z0-9_]*$",
    not_in: ["admin", "root", "system", "null", "undefined"]
  }];
}
```

## Common Patterns Reference

| Use Case | Rule | Example |
|----------|------|---------|
| Email | `email: true` | `user@example.com` |
| UUID | `uuid: true` | `550e8400-e29b-41d4-a716-446655440000` |
| URL | `uri: true` | `https://example.com/path` |
| Hostname | `hostname: true` | `api.example.com` |
| IPv4 | `ipv4: true` | `192.168.1.1` |
| IPv6 | `ipv6: true` | `::1` |
| Non-empty | `min_len: 1` | Any non-empty string |
