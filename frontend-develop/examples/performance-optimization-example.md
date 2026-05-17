# Performance Optimization Example

## Scenario

An order list page with search, filter, and sorting features is lagging with large datasets.

## Original Code

```tsx
import { useState, useEffect } from 'react'

function OrderList() {
  const [orders, setOrders] = useState<Order[]>([])
  const [search, setSearch] = useState('')
  const [sortBy, setSortBy] = useState<'date' | 'amount'>('date')

  useEffect(() => {
    fetchOrders().then(setOrders)
  }, [])

  const handleSearch = (e: React.ChangeEvent<HTMLInputElement>) => {
    setSearch(e.target.value)
  }

  return (
    <div>
      <input value={search} onChange={handleSearch} />
      <button onClick={() => setSortBy('date')}>By Date</button>
      <button onClick={() => setSortBy('amount')}>By Amount</button>
      <div>
        {orders
          .filter(o => o.name.includes(search))
          .sort((a, b) => a[sortBy] > b[sortBy] ? -1 : 1)
          .map(o => (
            <OrderCard key={o.id} order={o} />
          ))
        }
      </div>
    </div>
  )
}

function OrderCard({ order }: { order: Order }) {
  return (
    <div style={{ border: '1px solid #ddd', padding: 8 }}>
      <h4>{order.name}</h4>
      <p>Amount: ¥{order.amount}</p>
      <p>Date: {order.date}</p>
    </div>
  )
}
```

## Analysis Process

### Issue 1: Filtering/Sorting Recalculated on Every Render

```tsx
// ❌ Re-filters and re-sorts all data on every render
{orders
  .filter(o => o.name.includes(search))
  .sort((a, b) => a[sortBy] > b[sortBy] ? -1 : 1)
  .map(o => ...)
}
```

### Issue 2: OrderCard Has No React.memo

When search or sortBy changes, all OrderCards re-render even if their data hasn't changed.

### Issue 3: handleSearch Recreated on Every Render

```tsx
// ✅ Only meaningful when paired with a memo'd child component
```

## Optimized Code

```tsx
import { useState, useEffect, useMemo, useCallback } from 'react'

// Main component doesn't need memo — it has its own state
function OrderList() {
  const [orders, setOrders] = useState<Order[]>([])
  const [search, setSearch] = useState('')
  const [sortBy, setSortBy] = useState<'date' | 'amount'>('date')

  useEffect(() => {
    fetchOrders().then(setOrders)
  }, [])

  // ✅ Optimized: cache filter/sort results with useMemo
  const visibleOrders = useMemo(() => {
    console.log('recompute visibleOrders')
    return orders
      .filter(o => o.name.includes(search))
      .sort((a, b) => a[sortBy] > b[sortBy] ? -1 : 1)
  }, [orders, search, sortBy])

  // ✅ Optimized: stabilize callback reference with useCallback
  const handleSearch = useCallback(
    (e: React.ChangeEvent<HTMLInputElement>) => setSearch(e.target.value),
    []
  )

  return (
    <div>
      <SearchInput value={search} onChange={handleSearch} />
      <SortBar sortBy={sortBy} onSort={setSortBy} />
      <div>
        {visibleOrders.map(o => (
          <OrderCard key={o.id} order={o} />
        ))}
      </div>
    </div>
  )
}

// ✅ Optimized: React.memo to avoid unnecessary re-renders
const OrderCard = React.memo(function OrderCard({ order }: { order: Order }) {
  return (
    <div style={{ border: '1px solid #ddd', padding: 8 }}>
      <h4>{order.name}</h4>
      <p>Amount: ¥{order.amount}</p>
      <p>Date: {order.date}</p>
    </div>
  )
})

// ✅ Optimized: extract child components to isolate re-render scope
interface SearchInputProps {
  value: string
  onChange: (e: React.ChangeEvent<HTMLInputElement>) => void
}

const SearchInput = React.memo(function SearchInput({ value, onChange }: SearchInputProps) {
  return <input value={value} onChange={onChange} placeholder="Search orders..." />
})

interface SortBarProps {
  sortBy: string
  onSort: (sort: 'date' | 'amount') => void
}

const SortBar = React.memo(function SortBar({ sortBy, onSort }: SortBarProps) {
  return (
    <div>
      <button disabled={sortBy === 'date'} onClick={() => onSort('date')}>By Date</button>
      <button disabled={sortBy === 'amount'} onClick={() => onSort('amount')}>By Amount</button>
    </div>
  )
})
```

## Optimization Comparison

| Metric | Before | After |
|--------|--------|-------|
| Re-render on search input | All components | Only list content |
| Computation on sort toggle | Full data traversal | Once (cached) |
| OrderCard re-render | All cards | Only new/changed cards |
