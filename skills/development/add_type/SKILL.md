---
name: add_type
description: gwexpyに新しい配列型（Array/Series/Field）とコレクションを実装する
---

# Implement GWExPy Type

This skill guides the implementation of new array/field types in `gwexpy`, ensuring consistency with the existing class hierarchy, metadata management, and documentation standards.

## Workflow

### 1. Survey & Plan
*   **Identify Base Class**: Inherit from `gwexpy.types.Array`, `Array2D`, or similar.
*   **Metadata**: Determine new metadata slots needed (e.g., `_axis0_name`, `_unit`).
*   **Behavior**: Define slicing behavior (does it drop dimensions or maintain them?), arithmetic rules, and domain logic (FFT, etc.).

### 2. Implementation: Core Class
*   **File**: Create `gwexpy/types/yourtype.py`.
*   **Class Definition**:
    ```python
    class YourType(BaseArray):
        _metadata_slots = BaseArray._metadata_slots + ("_new_slot",)
    ```
*   **`__new__`**:
    *   Validate input structure (ndim, etc.).
    *   Initialize metadata slots (handling defaults).
    *   Call `super().__new__`.
*   **`__array_finalize__`**:
    *   Handle 3 creation scenarios: `obj` is None (explicit new), `obj` is subclass (view casting), `obj` is different type (copy/slice).
    *   Copy metadata from `obj` to `self`.
*   **Metadata-Preserving Indexing**: 
    `Field4D` 等の多次元クラスでは、インデックス操作 (`__getitem__`) 時に次元を落とさないことが推奨されます。整数インデックス `i` を `slice(i, i+1)` に変換して処理することで、軸の数とメタデータ（軸名称や単位）を一貫して維持できます。詳細は `manage_field_metadata` スキルを参照してください。
*   **Transpose/Swapaxes**:
    *   Override to update axis-dependent metadata if applicable.

### 3. Implementation: Collections
*   **File**: `gwexpy/types/yourtype_collections.py` (or inside the same file if small).
*   **List Class**: Inherit `list`. Add batch methods (e.g., `process_all`).
*   **Dict Class**: Inherit `dict`. Add batch methods.

### 4. Integration
*   **Export**: Add the new classes to `gwexpy/types/__init__.py`.
*   **Docs**:
    *   Create `docs/reference/en/YourType.md` and `docs/reference/ja/YourType.md`.
    *   Add to `docs/reference/{en,ja}/index.rst`.

### 5. Testing
*   **Location**: `tests/types/test_yourtype.py`.
*   **Coverage**:
    *   Construction (from array, from list, with units).
    *   Metadata persistence.
    *   Slicing behavior (key feature).
    *   Arithmetic operations.
    *   Collection behavior.

## Key Considerations

*   **Quantity Compatibility**: gwexpy types are often subclasses of `astropy.units.Quantity`. Ensure `unit` handling works.
*   **Axis Management**: If managing axes, use `AxisApiMixin` or look at `Array3D`/`Array4D` for how to sync separate axis properties with the array shape.
*   **Documentation**: Always provide both English and Japanese API references.
