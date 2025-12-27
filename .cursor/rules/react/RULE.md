---
description: React and TypeScript conventions for frontend applications
globs:
  - "**/*.tsx"
  - "**/*.ts"
  - "**/*.jsx"
  - "**/*.js"
alwaysApply: false
---

# React & TypeScript Standards

## Project Structure

```
src/
  ├── components/        # Reusable UI components
  │   └── Button/
  │       ├── Button.tsx
  │       ├── Button.test.tsx
  │       └── index.ts
  ├── features/          # Feature-based modules
  │   └── auth/
  │       ├── components/
  │       ├── hooks/
  │       ├── api.ts
  │       └── types.ts
  ├── hooks/             # Shared custom hooks
  ├── lib/               # Utilities and helpers
  ├── types/             # Global type definitions
  └── App.tsx
```

## Component Patterns

### Functional Components with TypeScript
```tsx
interface UserCardProps {
  user: User;
  onSelect?: (user: User) => void;
  className?: string;
}

export function UserCard({ user, onSelect, className }: UserCardProps) {
  return (
    <div className={cn("user-card", className)} onClick={() => onSelect?.(user)}>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
}
```

### Use Named Exports
```tsx
// Good: Named export
export function Button() { ... }

// Avoid: Default export (harder to refactor)
export default function Button() { ... }
```

## Hooks

### Custom Hook Pattern
```tsx
function useUser(userId: string) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let cancelled = false;
    
    async function fetchUser() {
      try {
        setIsLoading(true);
        const data = await api.getUser(userId);
        if (!cancelled) setUser(data);
      } catch (err) {
        if (!cancelled) setError(err as Error);
      } finally {
        if (!cancelled) setIsLoading(false);
      }
    }
    
    fetchUser();
    return () => { cancelled = true; };
  }, [userId]);

  return { user, isLoading, error };
}
```

### Hook Rules
- Only call hooks at the top level
- Only call hooks from React functions
- Prefix custom hooks with `use`

## State Management

### Local State First
```tsx
// Prefer useState for component-local state
const [isOpen, setIsOpen] = useState(false);

// Use useReducer for complex state logic
const [state, dispatch] = useReducer(reducer, initialState);
```

### Server State with React Query / TanStack Query
```tsx
function UserList() {
  const { data: users, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  });

  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;
  
  return <ul>{users.map(user => <UserItem key={user.id} user={user} />)}</ul>;
}
```

## Performance

### Memoization (use sparingly, measure first)
```tsx
// Memoize expensive computations
const sortedItems = useMemo(
  () => items.sort((a, b) => a.name.localeCompare(b.name)),
  [items]
);

// Memoize callbacks passed to children
const handleClick = useCallback((id: string) => {
  setSelected(id);
}, []);

// Memoize components that receive complex props
const MemoizedList = memo(function List({ items }: ListProps) {
  return <ul>{items.map(item => <li key={item.id}>{item.name}</li>)}</ul>;
});
```

### Lazy Loading
```tsx
const Dashboard = lazy(() => import('./features/dashboard/Dashboard'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <Dashboard />
    </Suspense>
  );
}
```

## Testing

### React Testing Library
```tsx
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('Button', () => {
  it('calls onClick when clicked', async () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    
    await userEvent.click(screen.getByRole('button', { name: /click me/i }));
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});
```

### Testing Principles
- Query by role, label, or text (accessibility-first)
- Avoid testing implementation details
- Test behavior, not structure

## TypeScript

### Strict Mode Always
```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true
  }
}
```

### Type Utilities
```tsx
// Pick specific props
type ButtonSize = Pick<ButtonProps, 'size'>;

// Omit unwanted props
type InputProps = Omit<React.InputHTMLAttributes<HTMLInputElement>, 'size'>;

// Make props optional
type PartialUser = Partial<User>;
```

## Styling

### CSS Modules or Tailwind
```tsx
// CSS Modules
import styles from './Button.module.css';
<button className={styles.primary}>Click</button>

// Tailwind with cn helper
import { cn } from '@/lib/utils';
<button className={cn("px-4 py-2 bg-blue-500", className)}>Click</button>
```

## Error Boundaries
```tsx
class ErrorBoundary extends Component<Props, State> {
  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }
  
  render() {
    if (this.state.hasError) {
      return <ErrorFallback error={this.state.error} />;
    }
    return this.props.children;
  }
}
```

