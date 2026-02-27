---
name: rename-method
description: Refactoring technique to improve code readability by renaming methods to better reflect their purpose and functionality
---

# Rename Method

## Overview

Rename Method is a fundamental refactoring technique applied when method names no longer accurately describe what the method does. Clear, descriptive method names are essential for code readability and maintainability. This refactoring improves code quality by making method intent explicit through proper naming conventions.

## Motivation

Methods acquire poor names for several reasons:
- Rushed initial development leaves temporary names in place
- Method functionality evolves beyond its original scope
- Inconsistent naming conventions across the codebase
- Comments become necessary to explain what a poorly-named method does

Unclear method names force developers to read implementation details to understand intent, reducing productivity and increasing the chance of misuse.

## Mechanics

The refactoring process follows these essential steps:

1. **Declare a new method** with the desired name, copying the original implementation
2. **Delegate the old method** to call the new method to maintain backward compatibility during transition
3. **Find and update all references** throughout the codebase to use the new method name
4. **Remove the old method** once all references are updated, or mark as deprecated if it's a public API

## Before/After Examples (PHP 8.3+)

### Example 1: Descriptive Method Name

**Before:**
```php
class UserRepository
{
    public function get(int $id): User
    {
        return $this->queryBuilder
            ->where('id = ?', $id)
            ->fetchOne();
    }
}

$user = $repository->get(1);
```

**After:**
```php
class UserRepository
{
    public function findUserById(int $id): User
    {
        return $this->queryBuilder
            ->where('id = ?', $id)
            ->fetchOne();
    }

    #[Deprecated(
        message: 'Use findUserById() instead',
        since: '2.0'
    )]
    public function get(int $id): User
    {
        return $this->findUserById($id);
    }
}

$user = $repository->findUserById(1);
```

### Example 2: Boolean Getter Clarity

**Before:**
```php
class Order
{
    public function valid(): bool
    {
        return $this->items->count() > 0 &&
               $this->total > 0 &&
               $this->customer !== null;
    }
}

if ($order->valid()) {
    // process order
}
```

**After:**
```php
class Order
{
    public function isReadyForProcessing(): bool
    {
        return $this->items->count() > 0 &&
               $this->total > 0 &&
               $this->customer !== null;
    }

    #[Deprecated(
        message: 'Use isReadyForProcessing() instead',
        since: '1.5'
    )]
    public function valid(): bool
    {
        return $this->isReadyForProcessing();
    }
}

if ($order->isReadyForProcessing()) {
    // process order
}
```

### Example 3: Transformation Methods

**Before:**
```php
class ProductPresenter
{
    public function calc(Product $product): array
    {
        return [
            'name' => $product->name,
            'price' => $product->price * 1.19,
            'stock' => $product->quantity,
        ];
    }
}
```

**After:**
```php
class ProductPresenter
{
    public function formatProductForDisplay(Product $product): array
    {
        return [
            'name' => $product->name,
            'price' => $product->price * 1.19,
            'stock' => $product->quantity,
        ];
    }

    #[Deprecated(
        message: 'Use formatProductForDisplay() instead',
        since: '3.0'
    )]
    public function calc(Product $product): array
    {
        return $this->formatProductForDisplay($product);
    }
}
```

## Benefits

- **Self-documenting code** – Method names clearly communicate intent without needing comments
- **Reduced cognitive load** – Developers understand code purpose at a glance
- **Fewer bugs** – Clear names reduce misuse and misunderstanding
- **Improved maintainability** – Future developers spend less time deciphering code logic
- **Better IDE support** – Clear names enable better autocomplete and refactoring suggestions
- **Cleaner codebase** – Eliminates the need for explanatory comments

## When NOT to Use

- **Public API contracts** – Renaming public methods breaks backward compatibility; use deprecation instead
- **Framework hooks** – Methods that override parent/interface contracts shouldn't be renamed
- **Legacy systems** – When widely dependent code can't be updated simultaneously
- **Name conflicts** – If a new name conflicts with existing methods in the class hierarchy
- **During active development** – Wait until functionality stabilizes before finalizing names

## Related Refactorings

- **Extract Method** – Often applied before Rename Method to isolate and clarify functionality
- **Move Method** – Combines renaming with relocating methods to more appropriate classes
- **Change Function Declaration** – Related technique for updating function signatures alongside names
- **Rename Variable** – Similar principle applied to variable naming clarity
- **Deprecation** – Use PHP 8.1+ `#[Deprecated]` attribute for safe public API transitions
