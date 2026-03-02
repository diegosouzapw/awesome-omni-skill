---
name: architecture-principles
description: Core architecture principles (SSOT, DRY, Anti-Spaghetti) for maintainable code design. Use when planning features, implementing code, or reviewing architecture to prevent duplication and technical debt.
---

# Architecture Principles

## Instructions

### Apply 3 principles

**SSOT (Single Source of Truth):** One place for each constant or logic
**DRY (Don't Repeat Yourself):** Extract when repeated 2+ times
**Anti-Spaghetti:** Clear layers, minimal dependencies

### When to apply

**Planning:** Design with SSOT in mind
**Implementing:** Extract shared logic
**Reviewing:** Check for duplication

## Example

<!-- CUSTOMIZE: Replace with {{MAIN_TECH_STACK}}-appropriate examples -->

### Python Example
```python
# ❌ Duplicated
def validate_api(value):
    if value < 10.0: raise ValueError("Too low")

def validate_webhook(value):
    if value < 10.0: raise ValueError("Too low")

# ✅ SSOT + DRY
class Validator:
    MIN_VALUE = 10.0  # SSOT: Single source

    def validate_value(self, value):  # DRY: Reusable
        if value < self.MIN_VALUE:
            raise ValueError(f"Below {self.MIN_VALUE}")

# Anti-Spaghetti: Use everywhere
validator.validate_value(value)
```

### JavaScript/Node.js Example
```javascript
// ❌ Duplicated
function validateApi(value) {
    if (value < 10.0) throw new Error("Too low");
}

function validateWebhook(value) {
    if (value < 10.0) throw new Error("Too low");
}

// ✅ SSOT + DRY
class Validator {
    static MIN_VALUE = 10.0;  // SSOT: Single source

    validateValue(value) {  // DRY: Reusable
        if (value < Validator.MIN_VALUE) {
            throw new Error(`Below ${Validator.MIN_VALUE}`);
        }
    }
}

// Anti-Spaghetti: Use everywhere
const validator = new Validator();
validator.validateValue(value);
```

### Go Example
```go
// ❌ Duplicated
func validateApi(value float64) error {
    if value < 10.0 { return errors.New("Too low") }
    return nil
}

func validateWebhook(value float64) error {
    if value < 10.0 { return errors.New("Too low") }
    return nil
}

// ✅ SSOT + DRY
type Validator struct {
    MinValue float64  // SSOT: Single source
}

func NewValidator() *Validator {
    return &Validator{MinValue: 10.0}
}

func (v *Validator) ValidateValue(value float64) error {  // DRY: Reusable
    if value < v.MinValue {
        return fmt.Errorf("Below %.2f", v.MinValue)
    }
    return nil
}

// Anti-Spaghetti: Use everywhere
validator := NewValidator()
validator.ValidateValue(value)
```

## Checklist

```markdown
[ ] Constants in one place (SSOT)?
[ ] Repeated logic extracted (DRY)?
[ ] Clear layer separation (Anti-Spaghetti)?
```

---

**For detailed patterns, see [reference.md](reference.md)**
**For more examples, see [examples.md](examples.md)**
