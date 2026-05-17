# Component Structure Refactoring Example

## Scenario

A "User Dashboard" page with all code in a single 500+ line component that is difficult to maintain.

## Original Code (Simplified)

```tsx
import { useState, useEffect } from 'react'

function Dashboard() {
  const [users, setUsers] = useState([])
  const [stats, setStats] = useState(null)
  const [selectedUser, setSelectedUser] = useState(null)
  const [editingUser, setEditingUser] = useState(null)
  const [formData, setFormData] = useState({})
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    Promise.all([
      fetchUsers(),
      fetchStats()
    ]).then(([usersData, statsData]) => {
      setUsers(usersData)
      setStats(statsData)
      setLoading(false)
    })
  }, [])

  const handleEdit = (user) => {
    setEditingUser(user)
    setFormData({ name: user.name, email: user.email, role: user.role })
  }

  const handleSave = async () => {
    await updateUser(editingUser.id, formData)
    const updated = await fetchUsers()
    setUsers(updated)
    setEditingUser(null)
  }

  if (loading) return <p>Loading...</p>

  return (
    <div>
      <header>
        <h1>Dashboard</h1>
        {stats && (
          <div>
            <p>Total Users: {stats.totalUsers}</p>
            <p>Active Users: {stats.activeUsers}</p>
            <p>New Today: {stats.newToday}</p>
          </div>
        )}
      </header>

      <table>
        {users.map(user => (
          <tr key={user.id}>
            <td>{user.name}</td>
            <td>{user.email}</td>
            <td>{user.role}</td>
            <td><button onClick={() => handleEdit(user)}>Edit</button></td>
          </tr>
        ))}
      </table>

      {editingUser && (
        <div className="modal">
          <h2>Edit User</h2>
          <input value={formData.name} onChange={e => setFormData({...formData, name: e.target.value})} />
          <input value={formData.email} onChange={e => setFormData({...formData, email: e.target.value})} />
          <select value={formData.role} onChange={e => setFormData({...formData, role: e.target.value})}>
            <option value="admin">Admin</option>
            <option value="user">User</option>
          </select>
          <button onClick={handleSave}>Save</button>
          <button onClick={() => setEditingUser(null)}>Cancel</button>
        </div>
      )}
    </div>
  )
}
```

## Refactored Structure

```
Dashboard/
├── index.tsx              # Composes child components, manages state
├── useDashboardData.ts    # Data fetching logic
├── useUserForm.ts         # Form editing logic
├── StatsCard.tsx          # Stats display
├── UserTable.tsx          # User table
├── UserFormModal.tsx      # Edit user modal
└── types.ts               # Type definitions
```

## Refactored Code

### Type Definitions `types.ts`

```ts
export interface User {
  id: string
  name: string
  email: string
  role: 'admin' | 'user'
}

export interface DashboardStats {
  totalUsers: number
  activeUsers: number
  newToday: number
}
```

### Data Hook `useDashboardData.ts`

```ts
import { useState, useEffect } from 'react'
import type { User, DashboardStats } from './types'

export function useDashboardData() {
  const [users, setUsers] = useState<User[]>([])
  const [stats, setStats] = useState<DashboardStats | null>(null)
  const [loading, setLoading] = useState(true)

  const loadData = () => {
    setLoading(true)
    return Promise.all([
      fetchUsers(),
      fetchStats()
    ]).then(([usersData, statsData]) => {
      setUsers(usersData)
      setStats(statsData)
      setLoading(false)
    })
  }

  useEffect(() => { loadData() }, [])

  return { users, stats, loading, refresh: loadData }
}
```

### Form Hook `useUserForm.ts`

```ts
import { useState } from 'react'
import type { User } from './types'

interface FormData {
  name: string
  email: string
  role: 'admin' | 'user'
}

export function useUserForm(onSave: () => void) {
  const [editingUser, setEditingUser] = useState<User | null>(null)
  const [formData, setFormData] = useState<FormData>({ name: '', email: '', role: 'user' })

  const startEdit = (user: User) => {
    setEditingUser(user)
    setFormData({ name: user.name, email: user.email, role: user.role })
  }

  const updateField = (field: keyof FormData, value: string) => {
    setFormData(prev => ({ ...prev, [field]: value }))
  }

  const save = async () => {
    if (!editingUser) return
    await updateUser(editingUser.id, formData)
    onSave()
    setEditingUser(null)
  }

  const cancel = () => setEditingUser(null)

  return { editingUser, formData, startEdit, updateField, save, cancel }
}
```

### Main Component `index.tsx`

```tsx
import { useDashboardData } from './useDashboardData'
import { useUserForm } from './useUserForm'
import { StatsCard } from './StatsCard'
import { UserTable } from './UserTable'
import { UserFormModal } from './UserFormModal'

export function Dashboard() {
  const { users, stats, loading, refresh } = useDashboardData()
  const form = useUserForm(refresh)

  if (loading) return <Loading />

  return (
    <div>
      <header>
        <h1>Dashboard</h1>
        {stats && <StatsCard stats={stats} />}
      </header>
      <UserTable users={users} onEdit={form.startEdit} />
      {form.editingUser && (
        <UserFormModal
          formData={form.formData}
          onFieldChange={form.updateField}
          onSave={form.save}
          onCancel={form.cancel}
        />
      )}
    </div>
  )
}
```

### Child Component `StatsCard.tsx`

```tsx
import type { DashboardStats } from './types'

export function StatsCard({ stats }: { stats: DashboardStats }) {
  return (
    <div className="stats">
      <p>Total Users: {stats.totalUsers}</p>
      <p>Active Users: {stats.activeUsers}</p>
      <p>New Today: {stats.newToday}</p>
    </div>
  )
}
```

### Child Component `UserTable.tsx`

```tsx
import type { User } from './types'

interface UserTableProps {
  users: User[]
  onEdit: (user: User) => void
}

export function UserTable({ users, onEdit }: UserTableProps) {
  return (
    <table>
      {users.map(user => (
        <tr key={user.id}>
          <td>{user.name}</td>
          <td>{user.email}</td>
          <td>{user.role}</td>
          <td><button onClick={() => onEdit(user)}>Edit</button></td>
        </tr>
      ))}
    </table>
  )
}
```

### Child Component `UserFormModal.tsx`

```tsx
import type { User } from './types'

interface UserFormModalProps {
  formData: { name: string; email: string; role: string }
  onFieldChange: (field: string, value: string) => void
  onSave: () => void
  onCancel: () => void
}

export function UserFormModal({ formData, onFieldChange, onSave, onCancel }: UserFormModalProps) {
  return (
    <div className="modal">
      <h2>Edit User</h2>
      <input value={formData.name} onChange={e => onFieldChange('name', e.target.value)} />
      <input value={formData.email} onChange={e => onFieldChange('email', e.target.value)} />
      <select value={formData.role} onChange={e => onFieldChange('role', e.target.value)}>
        <option value="admin">Admin</option>
        <option value="user">User</option>
      </select>
      <button onClick={onSave}>Save</button>
      <button onClick={onCancel}>Cancel</button>
    </div>
  )
}
```

## Refactoring Summary

| Problem | Refactoring | Benefit |
|---------|-------------|---------|
| Component > 500 lines | Split into 5 files | Each file < 100 lines |
| Data and UI mixed | Extracted useDashboardData hook | Logic reusable and testable |
| Form state scattered | Extracted useUserForm hook | State logic centralized |
| Stats display inline | Extracted StatsCard component | Independent rendering |
| User list inline | Extracted UserTable component | Can use memo for optimization |
| Edit modal inline | Extracted UserFormModal component | Clear responsibility |
