# Understanding useCallback in React

## What is useCallback?

`useCallback` is a React Hook that returns a **memoized version** of a function. It prevents the function from being recreated on every render unless its dependencies change.

```typescript
const memoizedFunction = useCallback(() => {
  // Your function logic
}, [dependencies])
```

---

## When to Use useCallback

### 1. Passing Functions to Child Components (Most Common)

**Problem:** Without `useCallback`, functions are recreated on every render, causing child components to re-render unnecessarily.

```typescript
// WITHOUT useCallback - Child re-renders every time
const ParentComponent = () => {
  const handleClick = () => {
    console.log('Clicked!')
  }
  
  return <ChildComponent onClick={handleClick} />
}

// WITH useCallback - Child only re-renders when needed
const ParentComponent = () => {
  const handleClick = useCallback(() => {
    console.log('Clicked!')
  }, []) // Empty array = function never changes
  
  return <ChildComponent onClick={handleClick} />
}

// Child component (memoized with React.memo)
const ChildComponent = React.memo(({ onClick }) => {
  return <button onClick={onClick}>Click Me</button>
})
```

**When to use:**

- Passing functions as props to memoized child components
- Child component is wrapped in `React.memo()`
- Optimizing performance in large component trees

---

### 2. Function in useEffect Dependencies

**Problem:** Functions defined in component body are recreated every render. If used in `useEffect` dependencies, this causes infinite loops.

```typescript
// WITHOUT useCallback - INFINITE LOOP!
const MyComponent = () => {
  const [data, setData] = useState([])
  
  const fetchData = async () => {
    const response = await api.get('/data')
    setData(response.data)
  }
  
  useEffect(() => {
    fetchData()  // ← Calls function
  }, [fetchData])  // ← fetchData changes every render = infinite loop!
  
  return <div>{data}</div>
}

// WITH useCallback - Works correctly
const MyComponent = () => {
  const [data, setData] = useState([])
  
  const fetchData = useCallback(async () => {
    const response = await api.get('/data')
    setData(response.data)
  }, []) // ← Function only created once
  
  useEffect(() => {
    fetchData()
  }, [fetchData]) // ← fetchData reference stays the same
  
  return <div>{data}</div>
}
```

**When to use:**

- Function is used in `useEffect` dependencies
- Need to call the function from multiple places (like a refresh button)

---

### 3. Alternative: Define Function Inside useEffect (Simpler)

**If the function is ONLY used in one `useEffect`, you can define it inline:**

```typescript
// SIMPLER - No useCallback needed
const MyComponent = () => {
  const [data, setData] = useState([])
  const [page, setPage] = useState(1)
  
  useEffect(() => {
    const fetchData = async () => {
      const response = await api.get(`/data?page=${page}`)
      setData(response.data)
    }
    
    fetchData()
  }, [page]) // ← Only page in dependencies
  
  return <div>{data}</div>
}
```

**When to use:**

- Function only used in this specific `useEffect`
- Don't need to call it elsewhere (no refresh button, etc.)
- Simpler and cleaner code

---

## Real-World Example: Sales Page

### With useCallback (When you need the function elsewhere)

```typescript
export const Sales = () => {
  const [sales, setSales] = useState([])
  const [currentPage, setCurrentPage] = useState(1)

  // useCallback - can reuse this function
  const fetchSales = useCallback(async () => {
    const response = await salesService.getSales({ page: currentPage })
    setSales(response.sales)
  }, [currentPage])

  useEffect(() => {
    fetchSales()
  }, [fetchSales])

  return (
    <div>
      <button onClick={fetchSales}>Refresh</button>  {/* ← Can reuse! */}
      {/* Sales table */}
    </div>
  )
}
```

### Without useCallback (Inline - simpler if only used once)

```typescript
export const Sales = () => {
  const [sales, setSales] = useState([])
  const [currentPage, setCurrentPage] = useState(1)

  useEffect(() => {
    const fetchSales = async () => {
      const response = await salesService.getSales({ page: currentPage })
      setSales(response.sales)
    }
    
    fetchSales()
  }, [currentPage])

  return (
    <div>
      {/* No refresh button - can't reuse fetchSales */}
      {/* Sales table */}
    </div>
  )
}
```

---

## When NOT to Use useCallback

### Don't use for every function

```typescript
// Overkill - unnecessary optimization
const handleClick = useCallback(() => {
  console.log('Clicked')
}, [])

return <button onClick={handleClick}>Click</button>
```

**Why not needed:**

- Button is a native HTML element (not a custom component)
- No performance benefit
- Adds complexity for no gain

### Don't use if function has no dependencies

```typescript
// Pointless - function never changes anyway
const logMessage = useCallback(() => {
  console.log('Static message')
}, [])
```

**Better:**

```typescript
// Just define it normally
const logMessage = () => {
  console.log('Static message')
}
```

---

## Decision Tree

```text

## Common Patterns

### Pattern 1: Fetch Data with Filters

```typescript
const [filters, setFilters] = useState({ search: '', category: '' })

const fetchData = useCallback(async () => {
  const response = await api.get('/data', { params: filters })
  setData(response.data)
}, [filters]) // ← Recreate when filters change

useEffect(() => {
  fetchData()
}, [fetchData])
```

### Pattern 2: Event Handlers for Memoized Children

```typescript
const handleDelete = useCallback((id: string) => {
  // Delete logic
}, []) // ← No dependencies, function never changes

return (
  <MemoizedList items={items} onDelete={handleDelete} />
)
```

### Pattern 3: Debounced Search

```typescript
const debouncedSearch = useCallback(
  debounce((query: string) => {
    // Search API
  }, 500),
  []
)
```

---

## Key Takeaways

1. **useCallback** memoizes functions (keeps same reference across renders)

2. **Use it when:**
   - Passing to memoized child components
   - Function is in useEffect dependencies AND used elsewhere

3. **Don't use it when:**
   - Passing to native HTML elements
   - Function only used in one useEffect (use inline instead)
   - Premature optimization

4. **Alternative:** Define function inside useEffect if only used there

5. **Dependencies matter:** Include all values the function uses from component scope

---

## TypeScript with useCallback

```typescript
// Explicit return type
const fetchData = useCallback(async (): Promise<void> => {
  const response = await api.get('/data')
  setData(response.data)
}, [])

// With parameters
const handleUpdate = useCallback(async (id: string): Promise<void> => {
  await api.patch(`/items/${id}`)
}, [])
```

---

## Further Reading

- [React Docs: useCallback](https://react.dev/reference/react/useCallback)
- [When to useMemo and useCallback](https://kentcdodds.com/blog/usememo-and-usecallback)
- [React Performance Optimization](https://react.dev/learn/render-and-commit)
