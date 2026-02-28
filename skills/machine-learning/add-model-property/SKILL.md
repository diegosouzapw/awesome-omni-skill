---
name: Add Model Property
description: Add a new property to an existing data model and propagate changes through model generation to client and server. Use when adding fields to entities, extending models, or modifying data structures. Handles source model editing, regeneration, ViewModel updates, and server-side changes.
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
---

# Add Model Property

This skill walks through the complete workflow of adding a property to a data model, including generation and propagation to client/server.

## When to Use

- Adding a new field to an existing entity
- Extending a model with additional properties
- Adding enum types to models
- Modifying entity data structures

## CRITICAL Warning

**NEVER edit generated model files!** Only edit source models in `/model/*.model/src/`

Generated files (DO NOT EDIT):
- `/client/libs/*/model/` ❌
- `/server/com.*.model/` ❌

Source files (EDIT HERE):
- `/model/*.model/src/` ✅

**Reference**: [MODEL_GENERATION_GUIDE.md - Critical Concepts](../../ai-guide/_ARCHITECTURE/MODEL_GENERATION_GUIDE.md#critical-concepts)

## Process

### Step 1: Locate the Source Model

Find the model file in `/model/*/src/`:
```bash
# Example: Find InitConfig model
grep -r "export class InitConfig" /workspaces/faad-hmi/model/*/src/
```

### Step 2: Choose Property Type

Determine the appropriate property type:

**Property Types**:
- `ValueProperty<T>` - Read-only from server (timestamps, computed values)
- `CommandedProperty<T>` - User can request changes (settings, modes)
- `RangedCommandedProperty<T>` - Numbers with min/max constraints
- `RangedProperty<T>` - Read-only numbers with constraints

**Supported Types**:
- `string` - Text values
- `number` - Numeric values (prefer RangedCommandedProperty if constrained)
- `boolean` - True/false flags
- `EnumType` - Custom enum types (must be defined separately)
- `string[]` - Arrays of entity IDs (ONLY string arrays supported!)
- `Date` - Timestamps (rare, usually use number for epoch)

**Array Limitations** (CRITICAL):
- ✅ `string[]` - Supported (for entity ID lists)
- ❌ `number[]` - NOT supported by generator
- ❌ `boolean[]` - NOT supported by generator
- ❌ `EnumType[]` - NOT supported by generator

**Workaround for unsupported arrays**: Create wrapper entities
```typescript
// Instead of: values?: CommandedProperty<number[]>
// Create wrapper entity:
export class NumberValue extends Entity {
    value?: CommandedProperty<number>;
}
// Then use: valueIds?: CommandedProperty<string[]>
```

**Reference**: [MODEL_GENERATION_GUIDE.md - Common Property Types](../../ai-guide/_ARCHITECTURE/MODEL_GENERATION_GUIDE.md#common-property-types)

### Step 3: Choose Property Name (CRITICAL)

**Include units in property names!** Comments don't transfer during generation.

**Good Names**:
- `contourLineWidthPixels` (not `contourLineWidth`)
- `targetAltitudeMeters` (not `targetAltitude`)
- `speedKnots` (not `speed`)
- `brightnessPercentFraction` (not `brightness`)
- `intervalSeconds` (not `interval`)

**Common Unit Suffixes**:
- Distance: `Meters`, `Kilometers`, `Feet`, `Miles`
- Screen: `Pixels`, `Rem`
- Angles: `Degrees`, `Radians`
- Time: `Seconds`, `Milliseconds`, `Minutes`
- Speed: `Knots`, `Mph`, `Kph`, `MetersPerSecond`
- Fractions: `PercentFraction` (0.0-1.0)
- Multipliers: `Multiplier` (e.g., `terrainExaggerationMultiplier`)

**Reference**: [MODEL_GENERATION_GUIDE.md - Best Practices](../../ai-guide/_ARCHITECTURE/MODEL_GENERATION_GUIDE.md#best-practices-for-property-naming)

### Step 4: Edit Source Model

1. Add property to class
2. Add property name to `"members"` array in `@description` comment
3. Import any new types

**Example**:
```typescript
/**
 * @description {
 *      "clazz" : {
 *          "extends": ["Entity"],
 *          "members": [
 *              "class",
 *              "existingProp1",
 *              "existingProp2",
 *              "newPropertyName"  // ← Add to members array!
 *          ]
 *      }
 * }
 */
export class MyModel extends Entity {
    /**
     * @default quicktype.projectname.MyModel
     */
    public static class: string = "quicktype.MyModel";

    existingProp1?: ValueProperty<string>;
    existingProp2?: CommandedProperty<number>;
    newPropertyName?: CommandedProperty<EnumType>;  // ← New property

    constructor(id: string) {
        super(id);
        this.className = MyModel.class;
    }
}
```

**CRITICAL Class Name Pattern**:
```typescript
/**
 * @default quicktype.projectname.ClassName    // ✅ Include package
 */
public static class: string = "quicktype.ClassName";  // ❌ NO package
```

**Reference**: [MODEL_GENERATION_GUIDE.md - Class/Entity Types](../../ai-guide/_ARCHITECTURE/MODEL_GENERATION_GUIDE.md#classentity-types)

### Step 5: Create Enum Type (If Needed)

If adding an enum property, create the enum file:

**File**: `/model/projectname.model/src/yourEnumType.ts`
```typescript
/**
 * @description {
 *      "clazz" : {
 *         "enum": "VALUE1,VALUE2,VALUE3",
 *         "title":"YourEnumType"
 *      }
 * }
 * @type integer
 */
export enum YourEnumType {
    VALUE1 = 0,
    VALUE2 = 1,
    VALUE3 = 2
}
```

**CRITICAL**: Enum values must be **globally unique across ALL enums**!
- ❌ WRONG: Multiple enums with `UNKNOWN` (generator collision!)
- ✅ CORRECT: Use prefixes like `UNKNOWN_KIND`, `UNKNOWN_SOURCE`

**Reference**: [MODEL_GENERATION_GUIDE.md - Enum Types](../../ai-guide/_ARCHITECTURE/MODEL_GENERATION_GUIDE.md#enum-types)

### Step 6: Run Model Generation

```bash
npm run generate-model
```

This generates:
- TypeScript models in `/client/libs/*/model/`
- C# models in `/server/com.*.model/`

**Check for errors** - Common issues:
- Missing property in `members` array
- Enum not defined
- Invalid property type
- Circular dependencies

**Reference**: [MODEL_GENERATION_GUIDE.md - Model Generation Workflow](../../ai-guide/_ARCHITECTURE/MODEL_GENERATION_GUIDE.md#model-generation-workflow)

### Step 7: Update Entity ViewModel

**IMPORTANT**: Property changes flow through both client and server. See full data flow in CLIENT_SERVER_BRIDGE.md.

Add property ViewModel getter:

```typescript
@computed
get newPropertyNameVM(): ICommandedVM<EnumType, IEnumFormatOptions<EnumType>> {
    const vm = this.createPropertyVM('newPropertyName', CommandedEnumViewModel<EnumType>);
    vm.configure({
        labelConverter: YourEnumTypeLabel,
        defaultValue: YourEnumType.VALUE1
    });
    return vm;
}
```

**@computed Usage**: Use `@computed` when getter includes configuration or derives values.

**Create label converter** in `adapters/types.ts`:
```typescript
export const YourEnumTypeLabel: Record<YourEnumType, string> = {
    [YourEnumType.VALUE1]: 'Display Label 1',
    [YourEnumType.VALUE2]: 'Display Label 2',
};
```

**References**:
- [ENTITY_ARCHITECTURE.md - Property ViewModel Patterns](../../ai-guide/_ARCHITECTURE/ENTITY_ARCHITECTURE.md#3-property-viewmodel-patterns)
- [CLIENT_SERVER_BRIDGE.md](../../ai-guide/_ARCHITECTURE/CLIENT_SERVER_BRIDGE.md) - Cross-stack property lifecycle

### Step 8: Update Server-Side Code (C#)

If this entity has server-side simulation/builder code:

1. Find builder class (usually `*Builder.cs`)
2. Add property initialization
3. Update any simulation logic that uses the entity

**Example locations**:
- `/server/com.faad.runner/simulations/`
- `/server/com.faad.runner/builders/`

**Reference**: [SERVER.md](../../ai-guide/_SERVER/SERVER.md)

### Step 9: Verify Build

```bash
# Check client-side errors
./tools/build-helpers/count-client-errors.sh
./tools/build-helpers/show-client-errors.sh 10

# Check server-side errors
./tools/build-helpers/count-server-errors.sh
./tools/build-helpers/show-server-errors.sh 10
```

All errors should be resolved.

### Step 10: Update UI Components

If the property should be visible/editable in UI:
1. Find the view component
2. Add input component bound to property VM
3. Use appropriate Phoenix component (Select, TextInput, Checkbox, etc.)

**Reference**: [UI_COMPONENT_GUIDELINES.md](../../ai-guide/_DAILY/UI_COMPONENT_GUIDELINES.md)

## Common Pitfalls

**Reference**: [COMMON_PITFALLS.md](../../ai-guide/_DAILY/COMMON_PITFALLS.md)

1. ❌ Editing generated files instead of source
2. ❌ Forgetting to add property to `members` array
3. ❌ Using unsupported array types (`number[]`, `boolean[]`)
4. ❌ Not including units in property names
5. ❌ Duplicate enum values across different enums
6. ❌ Forgetting to run `npm run generate-model`
7. ❌ Missing label converter for enum types

## Verification Checklist

- [ ] Source model edited in `/model/*/src/`
- [ ] Property added to `members` array
- [ ] Enum created (if applicable) with unique values
- [ ] `npm run generate-model` ran successfully
- [ ] Entity ViewModel updated with property VM
- [ ] Label converter created (for enums)
- [ ] Server builder updated (if applicable)
- [ ] Build passes (0 client/server errors)
- [ ] UI component updated (if user-facing)
- [ ] Tests updated

## Ask User If Unclear

- Property type (Value, Commanded, Ranged)
- Default value for the property
- Whether property should be visible/editable in UI
- Units for numeric properties
- Display labels for enum values
