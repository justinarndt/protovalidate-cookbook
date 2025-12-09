# Troubleshooting

Common issues and solutions when working with protovalidate.

## Import Errors

### "buf/validate/validate.proto" not found

**Symptom**: Build fails with import error.

**Solution**: Add the protovalidate dependency to `buf.yaml`:

```yaml
deps:
  - buf.build/bufbuild/protovalidate
```

Then run:

```bash
buf dep update
```

### Module not resolving

**Symptom**: `buf dep update` fails or module not found.

**Solution**: Verify your `buf.yaml` version and structure:

```yaml
version: v2  # Ensure v2 syntax
modules:
  - path: proto
deps:
  - buf.build/bufbuild/protovalidate
```

## Validation Not Running

### Rules being ignored

**Symptom**: Invalid data passes validation.

**Cause**: Validator not initialized or wrong message type.

**Solution** (Go):

```go
// Correct: Create validator once, reuse
validator, err := protovalidate.New()
if err != nil {
    // Handle error - don't ignore!
}

// Validate the actual proto message
err = validator.Validate(myProtoMessage)
```

### Optional fields not validated

**Symptom**: Validation passes when optional field is missing.

**Expected behavior**: By default, unset optional fields skip validation.

**Solution**: Use message-level rules if the field is required:

```protobuf
message Request {
  optional string email = 1 [(buf.validate.field).string.email = true];
  
  // Require email to be set
  option (buf.validate.message).cel = {
    id: "email.required",
    message: "email is required",
    expression: "has(this.email)"
  };
}
```

## CEL Expression Errors

### Syntax errors

**Symptom**: Build fails with CEL parse error.

**Common causes**:

1. **Unescaped backslashes** in regex:
   ```protobuf
   // Wrong
   expression: "this.matches('\d+')"
   
   // Correct
   expression: "this.matches('\\d+')"
   ```

2. **Missing quotes** around strings:
   ```protobuf
   // Wrong
   expression: "this == value"
   
   // Correct
   expression: "this == 'value'"
   ```

3. **Wrong field reference**:
   ```protobuf
   // Wrong (in field-level rule)
   expression: "this.email.size() > 0"
   
   // Correct (this IS the field value)
   expression: "this.size() > 0"
   ```

### Type mismatch

**Symptom**: CEL expression fails at runtime.

**Solution**: Verify types match:

```protobuf
// Wrong: comparing int to string
expression: "this.count == '5'"

// Correct
expression: "this.count == 5"
```

### has() not working

**Symptom**: `has(this.field)` always returns false.

**Cause**: `has()` only works on optional fields or message fields.

**Solution**: For required scalar fields, check for zero value:

```protobuf
// For optional fields
expression: "has(this.optional_email)"

// For required scalar fields (check non-zero)
expression: "this.name.size() > 0"
```

## Repeated Field Issues

### Validating items

**Symptom**: Items in repeated field not being validated.

**Solution**: Use `repeated.items` to validate each element:

```protobuf
repeated string emails = 1 [(buf.validate.field).repeated = {
  min_items: 1,
  items: {
    string: {
      email: true
    }
  }
}];
```

### Unique constraint

**Symptom**: Duplicate values allowed in list.

**Solution**: Add unique constraint:

```protobuf
repeated string tags = 1 [(buf.validate.field).repeated = {
  unique: true
}];
```

## Enum Issues

### Undefined values accepted

**Symptom**: Invalid enum integers pass validation.

**Solution**: Add `defined_only`:

```protobuf
Status status = 1 [(buf.validate.field).enum.defined_only = true];
```

### Using enum names in rules

**Symptom**: Build error when using enum name in `in`/`not_in`.

**Cause**: CEL uses integer values, not enum names.

**Solution**: Use integer values:

```protobuf
// Wrong
expression: "this in [STATUS_ACTIVE, STATUS_PENDING]"

// Correct
(buf.validate.field).enum.in = [1, 2]
```

## Performance Issues

### Validator initialization slow

**Symptom**: First validation call is slow.

**Solution**: Initialize validator once and reuse:

```go
// Do this once at startup
var validator *protovalidate.Validator

func init() {
    var err error
    validator, err = protovalidate.New()
    if err != nil {
        panic(err)
    }
}

// Reuse in handlers
func handleRequest(msg *pb.Request) error {
    return validator.Validate(msg)
}
```

### Warm up with expected types

**Solution**: Pre-compile validation for known types:

```go
validator, err := protovalidate.New(
    protovalidate.WithMessages(
        &pb.User{},
        &pb.Order{},
        &pb.Product{},
    ),
)
```

## Migration from protoc-gen-validate

### Different syntax

**Old (PGV)**:
```protobuf
import "validate/validate.proto";
string email = 1 [(validate.rules).string.email = true];
```

**New (protovalidate)**:
```protobuf
import "buf/validate/validate.proto";
string email = 1 [(buf.validate.field).string.email = true];
```

### Migration tool

Use the official migration tool:

```bash
buf convert --from=pgv --to=protovalidate ./proto
```

## Still Stuck?

1. Check the [protovalidate GitHub issues](https://github.com/bufbuild/protovalidate/issues)
2. Review the [official documentation](https://buf.build/docs/protovalidate/)
3. Join the [Buf Slack community](https://buf.build/links/slack)
