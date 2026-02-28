---
name: test-cotton-components
description: 'Guide AI agents on testing Django Cotton components using django-cotton-bs5 pytest fixtures. Covers when to use cotton_render vs cotton_render_soup vs cotton_render_string vs cotton_render_string_soup, best practices for component testing, DOM assertions, context handling, and multi-component testing patterns. Use when writing or reviewing Cotton component tests.'
---

# Test Cotton Components

A comprehensive guide for AI agents on effectively using django-cotton-bs5 pytest fixtures to test Django Cotton components. This skill teaches fixture selection, testing patterns, and best practices.

## When to Use This Skill

- Writing new tests for Django Cotton components
- Testing Bootstrap 5 components wrapped in Cotton syntax
- Reviewing or refactoring existing Cotton component tests
- User asks: "How do I test Cotton components?", "Which fixture should I use?", "Test this component"
- Debugging test failures in Cotton-based templates

## Prerequisites

- Project has `django-cotton-bs5` installed (provides pytest fixtures automatically)
- Django and pytest configured in the project
- Basic understanding of Django templates and Cotton component syntax

## Available Fixtures

### Fixture Decision Matrix

| Scenario | Input | Fixture | Output |
|----------|-------|---------|--------|
| Test single component by name | `'cotton_bs5.alert'` | `cotton_render` | Raw HTML string |
| Test single component with DOM queries | `'cotton_bs5.button'` | `cotton_render_soup` | BeautifulSoup object |
| Test inline multi-component markup | `"<c-ul><c-li /></c-ul>"` | `cotton_render_string` | Raw HTML string |
| Test complex nested structure with DOM queries | `"<c-card><c-button /></c-card>"` | `cotton_render_string_soup` | BeautifulSoup object |

### Quick Rules

1. **Use `_soup` variants when you need DOM traversal** (`find`, `find_all`, `select`, etc.)
2. **Use `_string` variants when testing multiple components together** or inline markup
3. **Use plain variants for simple string assertions** ("does it contain X?")
4. **All fixtures inject request object automatically** — don't create one manually

## Testing Patterns

### Pattern 1: Single Component - Basic Assertions

**Use Case**: Verify a component renders with correct CSS classes and content.

**Fixture**: `cotton_render` (raw HTML)

```python
def test_alert_renders_with_variant(cotton_render):
    """Test alert component renders correct variant class."""
    html = cotton_render(
        'cotton_bs5.alert',
        message="Warning message",
        variant="warning"
    )

    # Simple string assertions
    assert 'alert-warning' in html
    assert 'Warning message' in html
    assert 'alert' in html
```

**When to use**:
- Testing CSS class presence
- Verifying text content appears
- Quick smoke tests
- Testing component attributes are rendered

---

### Pattern 2: Single Component - DOM Structure

**Use Case**: Verify component structure, hierarchy, or specific element attributes.

**Fixture**: `cotton_render_soup` (parsed HTML)

```python
def test_button_structure_and_attributes(cotton_render_soup):
    """Test button component renders correct HTML structure."""
    soup = cotton_render_soup(
        'cotton_bs5.button',
        text="Click Me",
        variant="primary",
        size="lg"
    )

    # DOM traversal and assertions
    button = soup.find('button')
    assert button is not None
    assert 'btn' in button['class']
    assert 'btn-primary' in button['class']
    assert 'btn-lg' in button['class']
    assert button.get_text().strip() == 'Click Me'
```

**When to use**:
- Verifying element attributes (id, class, data-*, aria-*)
- Testing DOM hierarchy (parent/child relationships)
- Checking element counts (`len(soup.find_all('li'))`)
- Extracting and validating specific text content

---

### Pattern 3: Multi-Component - Inline Markup

**Use Case**: Test how multiple components work together in a template string.

**Fixture**: `cotton_render_string` (compiles Cotton syntax → raw HTML)

```python
def test_list_with_multiple_items(cotton_render_string):
    """Test list group with multiple items renders correctly."""
    html = cotton_render_string("""
        <c-list_group>
            <c-list_group.item text="First Item" />
            <c-list_group.item text="Second Item" active="True" />
            <c-list_group.item text="Third Item" />
        </c-list_group>
    """)

    # Verify all items present
    assert 'First Item' in html
    assert 'Second Item' in html
    assert 'Third Item' in html
    assert 'active' in html  # Active state rendered
```

**When to use**:
- Testing component composition
- Verifying parent-child component relationships
- Testing slot content with nested components
- Quick inline component experiments

---

### Pattern 4: Multi-Component - Complex Structure Testing

**Use Case**: Test nested components with DOM queries and assertions on structure.

**Fixture**: `cotton_render_string_soup` (compiles + parses)

```python
def test_card_with_nested_components(cotton_render_string_soup):
    """Test card component with nested title, body, and buttons."""
    soup = cotton_render_string_soup("""
        <c-card>
            <c-card.title>User Profile</c-card.title>
            <c-card.body>
                <p>Manage your account settings</p>
                <c-button variant='primary'>Edit Profile</c-button>
                <c-button variant='secondary'>Cancel</c-button>
            </c-card.body>
        </c-card>
    """)

    # Test structure
    card = soup.find('div', class_='card')
    assert card is not None

    # Test title
    title = card.find('h5')
    assert title.get_text().strip() == 'User Profile'

    # Test buttons
    buttons = card.find_all('button')
    assert len(buttons) == 2
    assert 'btn-primary' in buttons[0]['class']
    assert buttons[0].get_text().strip() == 'Edit Profile'
    assert 'btn-secondary' in buttons[1]['class']
```

**When to use**:
- Testing complex component hierarchies
- Verifying nested component relationships
- Counting child elements
- Testing layouts (cards, modals, accordions with nested content)

---

### Pattern 5: Context Variables & Cotton Attributes

**Use Case**: Test that global context and component-level attributes work together without leakage.

**Fixture**: `cotton_render_string` or `cotton_render_string_soup` (with context parameter)

```python
def test_component_with_global_context(cotton_render_string_soup):
    """Test component uses both global context and Cotton attributes."""
    template = """
        <c-alert variant='{{ alert_type }}'>
            {{ message }}
        </c-alert>
        <c-button variant='primary'>{{ button_text }}</c-button>
    """

    soup = cotton_render_string_soup(template, context={
        'alert_type': 'success',
        'message': 'Operation completed',
        'button_text': 'Continue'
    })

    # Verify context variables applied
    alert = soup.find('div', class_='alert')
    assert 'alert-success' in alert['class']
    assert 'Operation completed' in alert.get_text()

    button = soup.find('button')
    assert 'Continue' in button.get_text()
```

**When to use**:
- Testing components with dynamic attributes
- Verifying template variable interpolation
- Testing context isolation (no leakage between components)
- Integration testing with Django views

---

## Best Practices

### ✅ DO

1. **Use the simplest fixture that meets your needs**
   - `cotton_render` for simple assertions → faster
   - `cotton_render_soup` when you need DOM queries

2. **Test one concern per test function**
   ```python
   # Good: focused test
   def test_button_variant_primary(cotton_render):
       html = cotton_render('cotton_bs5.button', variant='primary')
       assert 'btn-primary' in html

   # Good: separate test for size
   def test_button_size_large(cotton_render):
       html = cotton_render('cotton_bs5.button', size='lg')
       assert 'btn-lg' in html
   ```

3. **Use descriptive test names and docstrings**
   ```python
   def test_alert_dismissible_renders_close_button(cotton_render_soup):
       """Test that dismissible alerts include a close button with correct aria-label."""
       soup = cotton_render_soup('cotton_bs5.alert', dismissible=True)
       # ...
   ```

4. **Test accessibility attributes**
   ```python
   def test_modal_has_aria_labels(cotton_render_soup):
       """Verify modal has proper ARIA attributes for screen readers."""
       soup = cotton_render_soup('cotton_bs5.modal', id='myModal')
       modal = soup.find('div', class_='modal')
       assert modal.get('aria-labelledby') is not None
       assert modal.get('role') == 'dialog'
   ```

5. **Test negative cases (attributes NOT present)**
   ```python
   def test_button_without_icon_has_no_icon_element(cotton_render_soup):
       """Verify button without icon doesn't render icon element."""
       soup = cotton_render_soup('cotton_bs5.button', text='Click')
       icon = soup.find('i')  # or svg, depending on icon implementation
       assert icon is None
   ```

### ❌ DON'T

1. **Don't create request objects manually** — fixtures handle it
   ```python
   # Bad
   def test_component(cotton_render):
       request = RequestFactory().get("/")  # Unnecessary!
       # ...

   # Good
   def test_component(cotton_render):
       html = cotton_render('component_name')
       # ...
   ```

2. **Don't use `cotton_render_soup` for simple string checks**
   ```python
   # Inefficient - parsing overhead
   def test_alert_has_text(cotton_render_soup):
       soup = cotton_render_soup('cotton_bs5.alert', message='Hello')
       assert 'Hello' in str(soup)

   # Better - direct string assertion
   def test_alert_has_text(cotton_render):
       html = cotton_render('cotton_bs5.alert', message='Hello')
       assert 'Hello' in html
   ```

3. **Don't test Bootstrap CSS implementation details** — test your component's behavior
   ```python
   # Bad - too brittle, tests Bootstrap internals
   def test_button_has_exact_padding():
       assert 'padding: 0.375rem 0.75rem' in html

   # Good - test component renders correct classes
   def test_button_applies_size_class(cotton_render):
       html = cotton_render('cotton_bs5.button', size='sm')
       assert 'btn-sm' in html
   ```

4. **Don't duplicate tests across fixtures** unless testing different aspects
   ```python
   # Bad - redundant
   def test_alert_with_render(cotton_render):
       html = cotton_render('cotton_bs5.alert')
       assert 'alert' in html

   def test_alert_with_soup(cotton_render_soup):  # Same test!
       soup = cotton_render_soup('cotton_bs5.alert')
       assert 'alert' in str(soup)

   # Good - different concerns
   def test_alert_css_class(cotton_render):
       html = cotton_render('cotton_bs5.alert')
       assert 'alert' in html

   def test_alert_structure(cotton_render_soup):
       soup = cotton_render_soup('cotton_bs5.alert')
       alert_div = soup.find('div', class_='alert')
       assert alert_div.get('role') == 'alert'
   ```

---

## Common Testing Scenarios

### Testing Slots (Named and Default)

```python
def test_button_slot_content(cotton_render_string_soup):
    """Test button with slot content instead of text attribute."""
    soup = cotton_render_string_soup("""
        <c-button variant='primary'>
            <i class='icon-save'></i> Save Changes
        </c-button>
    """)

    button = soup.find('button')
    assert button.find('i', class_='icon-save') is not None
    assert 'Save Changes' in button.get_text()
```

### Testing Conditional Rendering

```python
def test_alert_dismissible_button(cotton_render_soup):
    """Test close button only appears when dismissible=True."""
    # With dismissible
    soup = cotton_render_soup('cotton_bs5.alert', dismissible=True)
    assert soup.find('button', class_='btn-close') is not None

    # Without dismissible
    soup = cotton_render_soup('cotton_bs5.alert', dismissible=False)
    assert soup.find('button', class_='btn-close') is None
```

### Testing Dynamic Classes

```python
def test_button_combines_variant_and_size(cotton_render_soup):
    """Test button correctly combines variant and size classes."""
    soup = cotton_render_soup(
        'cotton_bs5.button',
        variant='success',
        size='lg'
    )
    button = soup.find('button')
    classes = button['class']
    assert 'btn' in classes
    assert 'btn-success' in classes
    assert 'btn-lg' in classes
```

### Testing Data Attributes

```python
def test_modal_data_attributes(cotton_render_soup):
    """Test modal renders correct data-bs-* attributes."""
    soup = cotton_render_soup('cotton_bs5.modal', id='testModal')

    toggle_button = soup.find('button', attrs={'data-bs-toggle': 'modal'})
    assert toggle_button is not None
    assert toggle_button.get('data-bs-target') == '#testModal'
```

---

## Troubleshooting

### Issue: "Fixture not found"

**Problem**: Test can't find cotton fixtures.

**Solution**: Ensure `django-cotton-bs5` is installed and pytest can discover it:

```bash
pip install django-cotton-bs5
pytest --fixtures | grep cotton_render
```

All four fixtures should appear. If not, check `INSTALLED_APPS` includes `cotton_bs5`.

---

### Issue: "Template syntax error"

**Problem**: Cotton syntax not compiling in `cotton_render_string`.

**Solution**:
1. Check your Cotton component syntax is valid
2. Verify component is registered in Django
3. Test component works in actual template first
4. Use `cotton_render('component.name')` to isolate if issue is component vs string rendering

---

### Issue: "Can't find element with BeautifulSoup"

**Problem**: `soup.find()` returns `None`.

**Solution**:
1. Print the HTML to inspect structure: `print(soup.prettify())`
2. Check if component rendered at all
3. Verify CSS selector or attributes are correct
4. Component might use different tag than expected

```python
# Debug: print HTML structure
def test_debug_component(cotton_render_soup):
    soup = cotton_render_soup('cotton_bs5.card')
    print(soup.prettify())  # Inspect actual output
    # Now adjust assertions based on what you see
```

---

### Issue: "Context variable not rendering"

**Problem**: Template variables like `{{ variable }}` show as literal text.

**Solution**: Pass context dict to fixture:

```python
# Wrong - variable won't render
html = cotton_render_string("<c-alert>{{ msg }}</c-alert>")

# Right - pass context
html = cotton_render_string(
    "<c-alert>{{ msg }}</c-alert>",
    context={'msg': 'Hello'}
)
```

---

## Example: Complete Test Suite for a Component

```python
"""Test suite for Button component."""

import pytest


class TestButtonComponent:
    """Test cotton_bs5.button component."""

    def test_button_basic_rendering(self, cotton_render):
        """Test button renders with default classes."""
        html = cotton_render('cotton_bs5.button', text='Click')
        assert 'btn' in html
        assert 'Click' in html

    def test_button_variants(self, cotton_render):
        """Test button renders all Bootstrap variants."""
        variants = ['primary', 'secondary', 'success', 'danger', 'warning', 'info', 'light', 'dark']

        for variant in variants:
            html = cotton_render('cotton_bs5.button', variant=variant)
            assert f'btn-{variant}' in html

    def test_button_sizes(self, cotton_render_soup):
        """Test button renders with size classes."""
        # Small
        soup = cotton_render_soup('cotton_bs5.button', size='sm')
        assert 'btn-sm' in soup.find('button')['class']

        # Large
        soup = cotton_render_soup('cotton_bs5.button', size='lg')
        assert 'btn-lg' in soup.find('button')['class']

    def test_button_as_link(self, cotton_render_soup):
        """Test button renders as anchor tag when href provided."""
        soup = cotton_render_soup('cotton_bs5.button', href='/test/', text='Link')

        link = soup.find('a')
        assert link is not None
        assert link.get('href') == '/test/'
        assert 'btn' in link['class']

    def test_button_disabled(self, cotton_render_soup):
        """Test button disabled attribute."""
        soup = cotton_render_soup('cotton_bs5.button', disabled=True)
        button = soup.find('button')
        assert button.get('disabled') is not None

    def test_button_with_icon_slot(self, cotton_render_string_soup):
        """Test button with icon in slot content."""
        soup = cotton_render_string_soup("""
            <c-button variant='primary'>
                <i class='bi bi-save'></i> Save
            </c-button>
        """)

        button = soup.find('button')
        icon = button.find('i', class_='bi-save')
        assert icon is not None
        assert 'Save' in button.get_text()
```

---

## Summary

Use this skill to guide fixture selection and write comprehensive, maintainable tests for Django Cotton components. Remember:

- **Start simple**: Use `cotton_render` for basic tests
- **Use BeautifulSoup** (`_soup`) when you need DOM queries
- **Test multi-component scenarios** with `cotton_render_string` variants
- **Test accessibility** (ARIA attributes, roles, labels)
- **Follow one-concern-per-test** principle
- **Document expected behavior** in docstrings

The fixtures make testing Cotton components straightforward — choose the right tool for each testing need!
