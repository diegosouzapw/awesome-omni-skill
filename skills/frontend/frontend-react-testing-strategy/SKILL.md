---
name: frontend-react-testing-strategy
description: Standardized guidelines and patterns for Frontend React Testing Strategy.
---

# Frontend React Testing Strategy

## When to use this skill
- Writing tests for React components
- Setting up testing infrastructure
- Improving test coverage
- Debugging flaky tests
- Code reviews for test quality

## Workflow
- [ ] Choose what to test (behavior, not implementation)
- [ ] Write tests with React Testing Library
- [ ] Follow AAA pattern (Arrange, Act, Assert)
- [ ] Test user interactions, not internals
- [ ] Mock external dependencies

## Instructions

### Testing Philosophy

**Test behavior, not implementation:**
```typescript
// ❌ Bad - testing implementation details
expect(component.state.count).toBe(1);

// ✅ Good - testing user-visible behavior
expect(screen.getByText('Count: 1')).toBeInTheDocument();
```

### Basic Component Test

```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from './Button';

describe('Button', () => {
  it('renders with correct text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
  });

  it('calls onClick when clicked', async () => {
    const handleClick = jest.fn();
    const user = userEvent.setup();
    
    render(<Button onClick={handleClick}>Click</Button>);
    
    await user.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });
});
```

### Form Testing

```typescript
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  it('submits form with email and password', async () => {
    const onSubmit = jest.fn();
    const user = userEvent.setup();
    
    render(<LoginForm onSubmit={onSubmit} />);
    
    // Fill form
    await user.type(screen.getByLabelText(/email/i), 'user@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    
    // Submit
    await user.click(screen.getByRole('button', { name: /log in/i }));
    
    // Assert
    expect(onSubmit).toHaveBeenCalledWith({
      email: 'user@example.com',
      password: 'password123'
    });
  });

  it('shows validation error for invalid email', async () => {
    const user = userEvent.setup();
    
    render(<LoginForm onSubmit={jest.fn()} />);
    
    await user.type(screen.getByLabelText(/email/i), 'invalid');
    await user.click(screen.getByRole('button', { name: /log in/i }));
    
    expect(await screen.findByText(/invalid email/i)).toBeInTheDocument();
  });
});
```

### Async Testing

```typescript
import { render, screen, waitFor } from '@testing-library/react';
import { UserProfile } from './UserProfile';

// Mock fetch
global.fetch = jest.fn();

describe('UserProfile', () => {
  beforeEach(() => {
    (fetch as jest.Mock).mockClear();
  });

  it('displays user data after loading', async () => {
    (fetch as jest.Mock).mockResolvedValueOnce({
      json: async () => ({ name: 'John Doe', email: 'john@example.com' })
    });

    render(<UserProfile userId="123" />);

    // Initially loading
    expect(screen.getByText(/loading/i)).toBeInTheDocument();

    // Wait for data
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
    });

    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });

  it('displays error message on fetch failure', async () => {
    (fetch as jest.Mock).mockRejectedValueOnce(new Error('Failed'));

    render(<UserProfile userId="123" />);

    expect(await screen.findByText(/error/i)).toBeInTheDocument();
  });
});
```

### Hook Testing

```typescript
import { renderHook, waitFor } from '@testing-library/react';
import { useUser } from './useUser';

// Mock API
jest.mock('./api', () => ({
  fetchUser: jest.fn()
}));

import { fetchUser } from './api';

describe('useUser', () => {
  it('fetches user data', async () => {
    (fetchUser as jest.Mock).mockResolvedValue({
      id: '1',
      name: 'John'
    });

    const { result } = renderHook(() => useUser('1'));

    expect(result.current.isLoading).toBe(true);

    await waitFor(() => {
      expect(result.current.isLoading).toBe(false);
    });

    expect(result.current.data).toEqual({ id: '1', name: 'John' });
  });

  it('refetches data when called', async () => {
    (fetchUser as jest.Mock).mockResolvedValue({ id: '1', name: 'John' });

    const { result } = renderHook(() => useUser('1'));

    await waitFor(() => {
      expect(result.current.data).toBeTruthy();
    });

    (fetchUser as jest.Mock).mockResolvedValue({ id: '1', name: 'Jane' });

    result.current.refetch();

    await waitFor(() => {
      expect(result.current.data?.name).toBe('Jane');
    });
  });
});
```

### Mocking Context

```typescript
import { render, screen } from '@testing-library/react';
import { AuthContext } from './AuthContext';
import { ProtectedRoute } from './ProtectedRoute';

const renderWithAuth = (component: React.ReactElement, isAuthenticated = false) => {
  return render(
    <AuthContext.Provider value={{ isAuthenticated, user: null }}>
      {component}
    </AuthContext.Provider>
  );
};

describe('ProtectedRoute', () => {
  it('shows content when authenticated', () => {
    renderWithAuth(<ProtectedRoute>Secret</ProtectedRoute>, true);
    expect(screen.getByText('Secret')).toBeInTheDocument();
  });

  it('redirects when not authenticated', () => {
    renderWithAuth(<ProtectedRoute>Secret</ProtectedRoute>, false);
    expect(screen.getByText(/login/i)).toBeInTheDocument();
  });
});
```

### Query Priority

```typescript
// Priority order (use the highest you can):
// 1. getByRole
expect(screen.getByRole('button', { name: /submit/i }));

// 2. getByLabelText (forms)
expect(screen.getByLabelText(/email/i));

// 3. getByPlaceholderText
expect(screen.getByPlaceholderText(/enter email/i));

// 4. getByText
expect(screen.getByText(/welcome/i));

// 5. getByTestId (last resort)
expect(screen.getByTestId('custom-element'));
```

### Snapshot Testing (Use Sparingly)

```typescript
import { render } from '@testing-library/react';
import { Card } from './Card';

it('matches snapshot', () => {
  const { container } = render(
    <Card title="Test" description="Description" />
  );
  expect(container).toMatchSnapshot();
});
```

## Resources
- Use React Testing Library, not Enzyme
- Query by role/label, not test IDs
- Test user behavior, not component state
- Mock network requests
- Use `userEvent` over `fireEvent`
