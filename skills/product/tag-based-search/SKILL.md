---
name: tag-based-search
description: Code tagging system using @FEAT, @COMP, @TYPE tags for easy discovery and navigation. Use when adding documentation tags or searching for related code across the codebase.
---

# Tag-Based Search

## Instructions

### Tag system

**@FEAT:feature-name** - Feature grouping
**@COMP:component-name** - Component type
**@TYPE:category** - Code category

### Usage

Add in comments:
```{{LANG}}
# @FEAT:{{feature-name}} @COMP:{{component}} @TYPE:{{type}}
class {{ClassName}}:
    pass
```

Search with grep:
```bash
grep -r "@FEAT:{{feature-name}}" {{src_directory}}/
```

## Example

<!-- CUSTOMIZE: Replace with {{PROJECT_NAME}} domain examples -->

```{{LANG}}
# @FEAT:user-auth @COMP:service @TYPE:core
class AuthenticationService:
    '''Handles user authentication'''
    pass

# @FEAT:user-auth @COMP:validator @TYPE:utility
def validate_credentials(username, password):
    return username and password
```

## Common Tags by Domain

### Generic/Universal Tags

**FEAT (Feature):**
- `user-auth`, `data-processing`, `file-upload`, `notification`, `payment`

**COMP (Component):**
- `service`, `validator`, `route`, `model`, `controller`, `middleware`, `util`, `config`

**TYPE (Category):**
- `core`, `api`, `data`, `utility`, `business`, `integration`, `validation`

---

### E-commerce Tags

**FEAT:**
```
@FEAT:product-catalog
@FEAT:shopping-cart
@FEAT:checkout
@FEAT:payment-processing
@FEAT:order-management
@FEAT:inventory-tracking
@FEAT:shipping-integration
@FEAT:customer-reviews
@FEAT:discount-pricing
```

**COMP:**
```
@COMP:product-service
@COMP:cart-service
@COMP:payment-gateway
@COMP:order-processor
@COMP:inventory-manager
@COMP:shipping-calculator
@COMP:price-engine
```

**Example:**
```{{LANG}}
# @FEAT:shopping-cart @COMP:service @TYPE:core
class CartService:
    def add_item(self, cart_id, product_id, quantity):
        pass

# @FEAT:checkout @COMP:validator @TYPE:validation
def validate_checkout(cart, address, payment):
    pass
```

---

### SaaS Application Tags

**FEAT:**
```
@FEAT:user-management
@FEAT:subscription-billing
@FEAT:team-collaboration
@FEAT:role-permissions
@FEAT:audit-logging
@FEAT:api-integration
@FEAT:webhook-notifications
@FEAT:data-export
@FEAT:analytics-dashboard
```

**COMP:**
```
@COMP:tenant-service
@COMP:subscription-manager
@COMP:billing-processor
@COMP:permission-checker
@COMP:audit-logger
@COMP:webhook-dispatcher
@COMP:analytics-aggregator
```

**Example:**
```{{LANG}}
# @FEAT:subscription-billing @COMP:service @TYPE:core
class SubscriptionService:
    def charge_customer(self, customer_id, amount):
        pass

# @FEAT:role-permissions @COMP:middleware @TYPE:integration
def require_permission(permission_name):
    pass
```

---

### Data Platform Tags

**FEAT:**
```
@FEAT:data-ingestion
@FEAT:data-transformation
@FEAT:data-validation
@FEAT:data-pipeline
@FEAT:data-quality
@FEAT:schema-evolution
@FEAT:data-lineage
@FEAT:job-scheduling
```

**COMP:**
```
@COMP:extractor
@COMP:transformer
@COMP:loader
@COMP:validator
@COMP:pipeline-orchestrator
@COMP:schema-registry
@COMP:job-scheduler
```

**Example:**
```{{LANG}}
# @FEAT:data-ingestion @COMP:extractor @TYPE:integration
class APIDataExtractor:
    def fetch_data(self, endpoint, params):
        pass

# @FEAT:data-validation @COMP:validator @TYPE:validation
def validate_schema(data, schema):
    pass
```

---

### Mobile/IoT Tags

**FEAT:**
```
@FEAT:device-registration
@FEAT:device-monitoring
@FEAT:firmware-update
@FEAT:telemetry-collection
@FEAT:remote-control
@FEAT:offline-sync
@FEAT:push-notification
@FEAT:geolocation
```

**COMP:**
```
@COMP:device-manager
@COMP:telemetry-collector
@COMP:firmware-updater
@COMP:sync-engine
@COMP:notification-sender
@COMP:location-tracker
```

**Example:**
```{{LANG}}
# @FEAT:device-monitoring @COMP:service @TYPE:core
class DeviceMonitoringService:
    def check_health(self, device_id):
        pass

# @FEAT:offline-sync @COMP:sync-engine @TYPE:core
class SyncEngine:
    def resolve_conflicts(self, local, remote):
        pass
```

---

### CRM/Marketing Tags

**FEAT:**
```
@FEAT:contact-management
@FEAT:lead-scoring
@FEAT:email-campaign
@FEAT:workflow-automation
@FEAT:sales-pipeline
@FEAT:reporting-analytics
@FEAT:contact-import
@FEAT:activity-tracking
```

**COMP:**
```
@COMP:contact-service
@COMP:lead-scorer
@COMP:email-sender
@COMP:workflow-engine
@COMP:pipeline-manager
@COMP:report-generator
```

**Example:**
```{{LANG}}
# @FEAT:lead-scoring @COMP:service @TYPE:business
class LeadScoringService:
    def calculate_score(self, lead):
        pass

# @FEAT:email-campaign @COMP:sender @TYPE:integration
class EmailCampaignSender:
    def send_campaign(self, campaign_id, recipients):
        pass
```

---

## Multi-Language Examples

### Python
```python
# @FEAT:user-auth @COMP:service @TYPE:core
class AuthenticationService:
    '''Handles user authentication'''

    # @FEAT:user-auth @COMP:service @TYPE:core
    def authenticate(self, username, password):
        pass

# @FEAT:user-auth @COMP:validator @TYPE:utility
def validate_password_strength(password):
    return len(password) >= 8
```

### JavaScript/TypeScript
```javascript
// @FEAT:user-auth @COMP:service @TYPE:core
class AuthenticationService {
    /**
     * Handles user authentication
     */

    // @FEAT:user-auth @COMP:service @TYPE:core
    authenticate(username, password) {
        // ...
    }
}

// @FEAT:user-auth @COMP:validator @TYPE:utility
function validatePasswordStrength(password) {
    return password.length >= 8;
}
```

### Go
```go
// @FEAT:user-auth @COMP:service @TYPE:core
// AuthenticationService handles user authentication
type AuthenticationService struct{}

// @FEAT:user-auth @COMP:service @TYPE:core
// Authenticate validates user credentials
func (s *AuthenticationService) Authenticate(username, password string) bool {
    // ...
    return false
}

// @FEAT:user-auth @COMP:validator @TYPE:utility
// ValidatePasswordStrength checks password requirements
func ValidatePasswordStrength(password string) bool {
    return len(password) >= 8
}
```

### Java
```java
// @FEAT:user-auth @COMP:service @TYPE:core
/**
 * Handles user authentication
 */
public class AuthenticationService {

    // @FEAT:user-auth @COMP:service @TYPE:core
    public boolean authenticate(String username, String password) {
        // ...
        return false;
    }
}

// @FEAT:user-auth @COMP:validator @TYPE:utility
/**
 * Validates password strength
 */
public class PasswordValidator {
    public static boolean validateStrength(String password) {
        return password.length() >= 8;
    }
}
```

---

## Search Strategies

### Find all code for a feature
```bash
# Find everything related to user-auth
grep -r "@FEAT:user-auth" src/

# With line numbers
grep -rn "@FEAT:user-auth" src/

# Show file names only
grep -rl "@FEAT:user-auth" src/
```

### Find specific component type
```bash
# Find all services
grep -r "@COMP:service" src/

# Find all validators
grep -r "@COMP:validator" src/
```

### Find by code category
```bash
# Find all core logic
grep -r "@TYPE:core" src/

# Find all integrations
grep -r "@TYPE:integration" src/
```

### Combined searches
```bash
# Find core services for user-auth
grep -r "@FEAT:user-auth" src/ | grep "@COMP:service" | grep "@TYPE:core"

# Find all validation code across features
grep -r "@TYPE:validation" src/
```

### Using Claude Code's Grep tool
```
Pattern: "@FEAT:user-auth"
Output mode: files_with_matches
Path: src/

# Then drill down:
Pattern: "@FEAT:user-auth.*@COMP:service"
Output mode: content
```

---

## Tag Maintenance

### When to add tags

**Always tag:**
- New classes/functions
- Core business logic
- Integration points
- Validation logic

**Optional (but recommended):**
- Utility functions
- Configuration files
- Test files

### Tag consistency rules

1. **One tag per line** (easier to grep)
   ```python
   # ✅ Good
   # @FEAT:user-auth @COMP:service @TYPE:core

   # ❌ Avoid
   # @FEAT:user-auth
   # @COMP:service
   # @TYPE:core
   ```

2. **Feature names lowercase with hyphens**
   ```
   ✅ @FEAT:user-auth
   ❌ @FEAT:UserAuth
   ❌ @FEAT:user_auth
   ```

3. **Component names match architecture**
   ```
   ✅ @COMP:service (if you use "services" in your architecture)
   ❌ @COMP:manager (if not a standard component type)
   ```

### Updating tags

When refactoring:
- Update @FEAT if feature name changes
- Update @COMP if component type changes
- Keep @TYPE consistent with new purpose

---

**For detailed patterns, see [reference.md](reference.md)**
**For more examples, see [examples.md](examples.md)**
