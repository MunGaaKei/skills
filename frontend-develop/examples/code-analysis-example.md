# Code Analysis Example

## Scenario

A user submitted the following component code that needs to be analyzed for issues.

## Original Code

```tsx
import { useState, useEffect } from 'react'
import { fetchOrders } from '../services/api'

function OrderPage(props) {
  const [data, setData] = useState()
  const [filter, setFilter] = useState('all')

  useEffect(() => {
    fetchOrders(props.userId).then(res => {
      setData(res)
    })
  }, [])

  const filterData = () => {
    if (filter === 'all') return data
    return data.filter(item => item.status === filter)
  }

  return (
    <div>
      <div>
        <button onClick={() => setFilter('all')}>All</button>
        <button onClick={() => setFilter('pending')}>Pending</button>
        <button onClick={() => setFilter('done')}>Done</button>
      </div>
      {data ? (
        <table>
          {filterData().map((item, index) => (
            <tr key={index}>
              <td>{item.id}</td>
              <td>{item.name}</td>
              <td>{item.status}</td>
            </tr>
          ))}
        </table>
      ) : <p>Loading...</p>}
    </div>
  )
}
```

## Analysis Process

### Issue 1: Props Missing Type Definition

```tsx
// ❌ props: any
function OrderPage(props) {
  // props.userId
}

// ✅ Define an interface
interface OrderPageProps {
  userId: string
}
```

### Issue 2: useEffect Missing Dependencies

```tsx
// ❌ userId missing from dependency array
useEffect(() => {
  fetchOrders(props.userId)
}, []) // Won't re-fetch if userId changes

// ✅ Complete dependencies
useEffect(() => {
  fetchOrders(props.userId)
}, [props.userId])
```

### Issue 3: Using Index as Key

```tsx
// ❌ Using index
{items.map((item, index) => <tr key={index} />)}

// ✅ Using unique id
{items.map(item => <tr key={item.id} />)}
```

### Issue 4: Conditional Rendering Can Be Simplified with Early Return

```tsx
// ❌ Nested ternary
{data ? <Table /> : <p>Loading...</p>}

// ✅ Early return
if (!data) return <Loading />
return <Table data={data} />
```

### Issue 5: Event Handlers Recreated on Every Render

```tsx
// ✅ Extract stable reference
<FilterBar onFilter={setFilter} />
```

## Refactored Code

```tsx
import { useState, useEffect } from 'react'
import { fetchOrders } from '../services/api'
import type { Order } from '../types'

interface OrderPageProps {
  userId: string
}

function OrderPage({ userId }: OrderPageProps) {
  const [data, setData] = useState<Order[] | null>(null)
  const [filter, setFilter] = useState<string>('all')

  useEffect(() => {
    fetchOrders(userId).then(setData)
  }, [userId])

  if (!data) return <Loading />

  const filtered = filter === 'all'
    ? data
    : data.filter(item => item.status === filter)

  return (
    <div>
      <FilterBar current={filter} onChange={setFilter} />
      <OrderTable data={filtered} />
    </div>
  )
}

// Child component
interface FilterBarProps {
  current: string
  onChange: (filter: string) => void
}

const FILTERS = [
  { key: 'all', label: 'All' },
  { key: 'pending', label: 'Pending' },
  { key: 'done', label: 'Done' },
] as const

function FilterBar({ current, onChange }: FilterBarProps) {
  return (
    <div>
      {FILTERS.map(f => (
        <button
          key={f.key}
          onClick={() => onChange(f.key)}
          disabled={current === f.key}
        >
          {f.label}
        </button>
      ))}
    </div>
  )
}

// Use React.memo to avoid unnecessary re-renders
const OrderTable = React.memo(function OrderTable({ data }: { data: Order[] }) {
  return (
    <table>
      {data.map(item => (
        <tr key={item.id}>
          <td>{item.id}</td>
          <td>{item.name}</td>
          <td>{item.status}</td>
        </tr>
      ))}
    </table>
  )
})
```
