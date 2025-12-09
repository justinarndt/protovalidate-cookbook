# Error Handling

When validation fails, protovalidate returns structured error information. Understanding error handling helps you provide meaningful feedback to users.

## Violation Structure

Each validation failure produces a `Violation` with:

| Field | Description |
|-------|-------------|
| `field_path` | Path to the invalid field (e.g., `user.email`) |
| `constraint_id` | The `id` from the violated rule |
| `message` | Human-readable error message |

## Language-Specific Handling

=== "Go"

    ```go
    import (
        "buf.build/go/protovalidate"
    )
    
    func validate(msg proto.Message) error {
        validator, err := protovalidate.New()
        if err != nil {
            return fmt.Errorf("failed to create validator: %w", err)
        }
        
        if err := validator.Validate(msg); err != nil {
            var valErr *protovalidate.ValidationError
            if errors.As(err, &valErr) {
                for _, violation := range valErr.Violations {
                    fmt.Printf("Field: %s\n", violation.GetFieldPath())
                    fmt.Printf("Rule: %s\n", violation.GetConstraintId())
                    fmt.Printf("Message: %s\n", violation.GetMessage())
                }
            }
            return err
        }
        return nil
    }
    ```

=== "Python"

    ```python
    from protovalidate import validate, ValidationError
    
    def validate_message(msg):
        try:
            validate(msg)
        except ValidationError as e:
            for violation in e.violations:
                print(f"Field: {violation.field_path}")
                print(f"Rule: {violation.constraint_id}")
                print(f"Message: {violation.message}")
            raise
    ```

=== "Java"

    ```java
    import build.buf.protovalidate.Validator;
    import build.buf.protovalidate.ValidationResult;
    import build.buf.protovalidate.Violation;
    
    Validator validator = new Validator();
    ValidationResult result = validator.validate(message);
    
    if (!result.isSuccess()) {
        for (Violation violation : result.getViolations()) {
            System.out.println("Field: " + violation.getFieldPath());
            System.out.println("Rule: " + violation.getConstraintId());
            System.out.println("Message: " + violation.getMessage());
        }
    }
    ```

## Nested Field Paths

For nested messages, `field_path` includes the full path:

```protobuf
message Order {
  Customer customer = 1;
}

message Customer {
  string email = 1 [(buf.validate.field).string.email = true];
}
```

Invalid email produces: `field_path: "customer.email"`

## Repeated Field Paths

For repeated fields, the index is included:

```protobuf
message Order {
  repeated LineItem items = 1;
}

message LineItem {
  int32 quantity = 1 [(buf.validate.field).int32.gt = 0];
}
```

Invalid quantity in second item: `field_path: "items[1].quantity"`

## Map Field Paths

For maps, the key is included:

```protobuf
message Config {
  map<string, Setting> settings = 1;
}

message Setting {
  string value = 1 [(buf.validate.field).string.min_len = 1];
}
```

Empty value for key "timeout": `field_path: "settings[\"timeout\"].value"`

## Multiple Violations

A single message can have multiple violations. Always iterate through all of them:

```go
if err := validator.Validate(msg); err != nil {
    var valErr *protovalidate.ValidationError
    if errors.As(err, &valErr) {
        // May contain multiple violations
        fmt.Printf("Found %d violations\n", len(valErr.Violations))
        for _, v := range valErr.Violations {
            // Handle each violation
        }
    }
}
```

## Converting to API Errors

### gRPC

Map violations to gRPC status details:

```go
import (
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
    "google.golang.org/genproto/googleapis/rpc/errdetails"
)

func toGRPCError(err error) error {
    var valErr *protovalidate.ValidationError
    if !errors.As(err, &valErr) {
        return status.Error(codes.Internal, "validation failed")
    }
    
    st := status.New(codes.InvalidArgument, "validation failed")
    
    br := &errdetails.BadRequest{}
    for _, v := range valErr.Violations {
        br.FieldViolations = append(br.FieldViolations, 
            &errdetails.BadRequest_FieldViolation{
                Field:       v.GetFieldPath(),
                Description: v.GetMessage(),
            })
    }
    
    st, _ = st.WithDetails(br)
    return st.Err()
}
```

### REST/JSON

Convert to a structured JSON response:

```go
type ValidationErrorResponse struct {
    Error  string            `json:"error"`
    Fields []FieldViolation  `json:"fields"`
}

type FieldViolation struct {
    Field   string `json:"field"`
    Message string `json:"message"`
}

func toJSONError(err error) ValidationErrorResponse {
    var valErr *protovalidate.ValidationError
    if !errors.As(err, &valErr) {
        return ValidationErrorResponse{Error: "validation failed"}
    }
    
    resp := ValidationErrorResponse{
        Error:  "validation failed",
        Fields: make([]FieldViolation, 0, len(valErr.Violations)),
    }
    
    for _, v := range valErr.Violations {
        resp.Fields = append(resp.Fields, FieldViolation{
            Field:   v.GetFieldPath(),
            Message: v.GetMessage(),
        })
    }
    
    return resp
}
```

Example JSON output:

```json
{
  "error": "validation failed",
  "fields": [
    {
      "field": "email",
      "message": "value must be a valid email address"
    },
    {
      "field": "age",
      "message": "value must be less than or equal to 150"
    }
  ]
}
```

## Writing Good Error Messages

Custom CEL rules should include helpful error messages:

```protobuf
// Bad: Generic message
option (buf.validate.message).cel = {
  id: "check",
  message: "invalid",
  expression: "this.end > this.start"
};

// Good: Specific and actionable
option (buf.validate.message).cel = {
  id: "date_range.end_after_start",
  message: "end_date must be after start_date",
  expression: "this.end > this.start"
};
```

## Best Practices

1. **Always handle multiple violations**: Don't stop at the first error
2. **Preserve field paths**: They're essential for client-side form validation
3. **Use constraint IDs**: They enable programmatic error handling
4. **Write actionable messages**: Tell users what's wrong and how to fix it
5. **Log full errors server-side**: Include constraint IDs for debugging
