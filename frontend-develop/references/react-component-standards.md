# React Component Development Standards

## Component Splitting Structure

Break large pages into small components + hooks:

```
UserPage/
├── index.tsx          # Main component, composes children
├── UserTable.tsx      # Table
├── UserFormModal.tsx  # Form modal
├── useUserData.ts     # Data fetching
└── useUserForm.ts     # Form logic
```

**Signals to split:**
- Component exceeds 200 lines
- Multiple unrelated state values in a single component
- More than 3 conditional rendering branches

## UI & Logic Separation

Hooks manage logic, components handle rendering:

```tsx
// useUserData.ts
export function useUserData(id: string) {
  const [user, setUser] = useState<User | null>(null)
  useEffect(() => { fetchUser(id).then(setUser) }, [id])
  return { user, loading: user === null }
}

// UserProfile.tsx — pure rendering
function UserProfile({ id }: { id: string }) {
  const { user, loading } = useUserData(id)
  if (loading) return <Skeleton />
  return <UserCard user={user} />
}
```

## Props Design

- Every component must have an explicit `interface` definition
- Boolean props use `is`/`has` prefix (`isLoading`, `hasError`)
- Callbacks use `on` prefix (`onClick`, `onSave`)

## JSX Readability

- Extract child components when nesting exceeds 4 levels
- Use early return instead of nested ternaries for complex conditions
- Use unique IDs for list keys, never index
