---
name: add-bc-contract
description: Add Contract for inter-BC communication using Provider pattern. Use when one Bounded Context needs to access data from another BC (e.g., Inventory needs Articles from Admin). Creates Contract interface, Provider implementation, and configuration.
---

# Add BC Contract

Create Contract for inter-BC communication using Provider pattern.

---

## When to Use

- BC needs to read data from another BC
- Creating select options from another BC (TwigComponent)
- Exposing domain data to other contexts

---

## Inputs/Outputs

| Input | Example | Output |
|-------|---------|--------|
| provider_bc | Admin | `ProviderBC/Contracts/ContractName.php` |
| consumer_bc | Inventory | `ProviderBC/Adapters/Contracts/ProviderBCContractName.php` |
| contract_name | ArticleProvider | `ProviderBC/Frameworks/config/services.yaml` (updated) |
| methods | ['provide', 'provideAll'] | `ConsumerBC/Frameworks/deptrac.yaml` (updated) |

---

## Process

| Step | File | Action |
|------|------|--------|
| **Contract** | `ProviderBC/Contracts/ContractName.php` | Interface with methods (template: `contract.php.tpl`) |
| **Provider** | `ProviderBC/Adapters/Contracts/ProviderBCContractName.php` | Implementation with Finder (template: `provider.php.tpl`) |
| **Config** | `ProviderBC/Frameworks/config/services.yaml` | Autowire Contract → Provider |
| **Deptrac** | `ConsumerBC/Frameworks/deptrac.yaml` | Allow ProviderBC\Contracts |
| **Validate** | - | `make cs-fixer && make stan && bin/deptrac analyse && make qa` |

---

## Structures

**Contract** (interface in `ProviderBC/Contracts/`):
```php
interface ContractName {
    public function provide(string $uuid): EntityData; // throws
    public function provideAll(?array $ids = null): iterable;
}
```

**Provider** (`readonly`, uses Finder):
```php
final readonly class ProviderBCContractName implements ContractName {
    public function __construct(private EntityFinder $finder) {} // Finder, NOT Repository

    public function provide(string $uuid): EntityData {
        $entity = $this->finder->find($uuid) ?? throw EntityNotFound::fromUuid($uuid);
        return $this->toData($entity);
    }

    public function provideAll(?array $ids = null): iterable {
        $entities = $ids ? $this->finder->findByUuids($ids) : $this->finder->findAll();
        foreach ($entities as $entity) {
            yield $this->toData($entity);
        }
    }

    private function toData(Entity $entity): EntityData { /* convert to DTO */ }
}
```

**Config** (`ProviderBC/Frameworks/config/services.yaml`):
```yaml
ProviderBC\Contracts\ContractName:
    class: ProviderBC\Adapters\Contracts\ProviderBCContractName
```

**Deptrac** (`ConsumerBC/Frameworks/deptrac.yaml`):
```yaml
ConsumerBC\Adapters:
    - ProviderBC\Contracts  # ONLY Contracts, NOT Entities/UseCases
```

**See**: `docs/GLOSSARY.md#contract`, `#provider`, `#data-dto`

---

## Rules

**CRITICAL**:
- Provider uses Finder (NOT Repository) - providers are read-only
- Consumer depends ONLY on Contract interface (never Provider implementation)
- Deptrac allows ONLY `Contracts` namespace (not Entities/UseCases)

**Locations**:
- Contract: `ProviderBC/Contracts/`
- Provider: `ProviderBC/Adapters/Contracts/`
- Exception: `ProviderBC/Contracts/Exception/` if needed
- DTO: `ProviderBC/Contracts/DTO/` if complex data

**Naming**:
- Contract: `{Entity}Provider` (e.g., ArticleProvider)
- Provider: `{BC}{Contract}` (e.g., AdminArticleProvider)

---

## Variants

**Query Provider** (data access):
```php
public function provide(string $uuid): EntityData;
public function provideAll(?array $ids = null): iterable;
```

**TwigComponent Provider** (form select):
```php
/** @return array<string, string> [uuid => label] */
public function getAllForChoice(): array;
```

---

## Templates

- `contract.php.tpl` - Contract interface
- `provider.php.tpl` - Provider implementation

**Location**: `.claude/templates/`

---

## References

- Contract/Provider pattern: `docs/GLOSSARY.md#contract`, `#provider`
- Inter-BC architecture: `docs/architecture.md#inter-bc`
- Detailed guide: `docs/guides/bounded-contexts.md`
