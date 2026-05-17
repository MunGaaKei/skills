# Frontend Code Standards (General)

## Project Structure

```
src/
  features/        # Business modules
    auth/
    dashboard/
    settings/
  components/      # Global UI components
  hooks/           # Global hooks
  services/        # API request layer
  utils/           # Utility functions
```

**Rules:**
- Only put something in global directories when it's reused across multiple features
- Feature-specific components/hooks go inside the feature directory, not in global
- Utility/helper functions go in a local `utils/` dir (if only used in that directory) or root `utils/` (if reused globally)

## API Layer

Centralize all requests in `services/`. Don't call fetch/axios directly inside components:

```ts
// services/user.ts
export async function fetchUser(id: string): Promise<User> {
  const res = await fetch(`/api/users/${id}`)
  if (!res.ok) throw new Error('Failed to fetch user')
  return res.json()
}
```

## Error Handling Pattern

```ts
function useData<T>(fetcher: () => Promise<T>) {
  const [state, setState] = useState<{
    data: T | null
    loading: boolean
    error: string | null
  }>({ data: null, loading: true, error: null })

  useEffect(() => {
    fetcher()
      .then(data => setState({ data, loading: false, error: null }))
      .catch(err => setState({ data: null, loading: false, error: err.message }))
  }, [])

  return state
}
```

## Constants

Replace magic values with named constants:

```ts
// ❌
if (status === 3) {}

// ✅
const STATUS_ACTIVE = 3
if (status === STATUS_ACTIVE) {}
```
