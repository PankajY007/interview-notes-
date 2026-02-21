# 11 — React Basics (Interview Ready)

> **Level:** 3-4 Years Experience (Backend dev with React exposure) | **Priority:** 🟡 Important  
> Covers: Components, JSX, State, Props, Hooks, Lifecycle, API Integration, Routing

---

## 📌 Table of Contents

1. [React Fundamentals](#1-react-fundamentals)
2. [Components & JSX](#2-components--jsx)
3. [Props & State](#3-props--state)
4. [Hooks (Deep Dive)](#4-hooks)
5. [Component Lifecycle](#5-component-lifecycle)
6. [Event Handling & Forms](#6-event-handling--forms)
7. [API Integration with Backend](#7-api-integration)
8. [React Router](#8-react-router)
9. [State Management Basics](#9-state-management)
10. [Interview Questions & Answers](#10-interview-questions--answers)

---

## 1. React Fundamentals

### What is React?
- **JavaScript library** for building UIs (not a framework)
- Created by **Facebook (Meta)**
- Uses **Virtual DOM** for efficient updates
- **Component-based** architecture
- **Declarative** — describe what UI should look like, React handles the updates

### Virtual DOM
```
1. State changes → React creates new Virtual DOM tree
2. React DIFFS new Virtual DOM with previous one
3. Calculates minimal set of changes needed
4. Applies only those changes to Real DOM (efficient!)
```

---

## 2. Components & JSX

### Functional Component (Modern Standard)
```jsx
// Simple component
function Greeting({ name }) {
    return <h1>Hello, {name}!</h1>;
}

// Arrow function (equivalent)
const Greeting = ({ name }) => <h1>Hello, {name}!</h1>;

// Usage
<Greeting name="John" />
```

### JSX — JavaScript XML
```jsx
// JSX is syntactic sugar for React.createElement()
const element = <h1 className="title">Hello</h1>;
// Compiles to:
const element = React.createElement('h1', { className: 'title' }, 'Hello');

// JSX Rules:
// 1. Return single root element (use <> fragment if needed)
// 2. Use className instead of class
// 3. Use camelCase for attributes (onClick, onChange)
// 4. All tags must be closed (<img /> not <img>)
// 5. JavaScript expressions in {} curly braces

const App = () => {
    const items = ['Java', 'Spring', 'React'];
    const isLoggedIn = true;

    return (
        <>
            {/* Conditional rendering */}
            {isLoggedIn ? <Dashboard /> : <Login />}
            {isLoggedIn && <WelcomeMessage />}

            {/* List rendering */}
            <ul>
                {items.map((item, index) => (
                    <li key={index}>{item}</li>  // key is required!
                ))}
            </ul>
        </>
    );
};
```

---

## 3. Props & State

### Props (Read-only, parent → child)
```jsx
// Parent passes data
<UserCard name="John" age={30} onDelete={handleDelete} />

// Child receives and uses
const UserCard = ({ name, age, onDelete }) => (
    <div>
        <h2>{name} (Age: {age})</h2>
        <button onClick={onDelete}>Delete</button>
    </div>
);

// Default props
const UserCard = ({ name = "Anonymous", age = 0 }) => { ... };

// Children prop
const Card = ({ children, title }) => (
    <div className="card">
        <h2>{title}</h2>
        {children}  {/* whatever is nested inside <Card>...</Card> */}
    </div>
);

<Card title="Profile">
    <p>This is the card content</p>
</Card>
```

### State (Mutable, managed within component)
```jsx
import { useState } from 'react';

const Counter = () => {
    const [count, setCount] = useState(0); // [value, setter]

    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={() => setCount(count + 1)}>+</button>
            <button onClick={() => setCount(prev => prev - 1)}>-</button>
            {/* ↑ Use callback form when new state depends on old state */}
        </div>
    );
};

// Object state
const [user, setUser] = useState({ name: '', email: '' });
setUser(prev => ({ ...prev, name: 'John' })); // spread to preserve other fields
```

---

## 4. Hooks

### useState
```jsx
const [state, setState] = useState(initialValue);
// initialValue can be: primitive, object, array, or function
```

### useEffect — Side effects (API calls, subscriptions, DOM manipulation)
```jsx
import { useEffect } from 'react';

const UserProfile = ({ userId }) => {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        // Runs after every render (if no dependency array)
        fetchUser(userId)
            .then(data => setUser(data))
            .finally(() => setLoading(false));

        // Cleanup function (runs on unmount or before next effect)
        return () => {
            // cancel subscriptions, timers, etc.
        };
    }, [userId]); // Dependency array — re-run only when userId changes

    if (loading) return <p>Loading...</p>;
    return <p>{user.name}</p>;
};

// useEffect patterns:
useEffect(() => { }, []);      // ← runs ONCE on mount (componentDidMount)
useEffect(() => { }, [x, y]);  // ← runs when x or y changes
useEffect(() => { });          // ← runs on EVERY render (rarely needed)
```

### useContext — Avoid prop drilling
```jsx
const ThemeContext = React.createContext('light');

// Provider (top level)
const App = () => (
    <ThemeContext.Provider value="dark">
        <Dashboard />
    </ThemeContext.Provider>
);

// Consumer (any nested level)
const Sidebar = () => {
    const theme = useContext(ThemeContext); // "dark"
    return <div className={theme}>Sidebar</div>;
};
```

### useRef — Access DOM elements / persist values
```jsx
const InputFocus = () => {
    const inputRef = useRef(null);

    const handleClick = () => {
        inputRef.current.focus(); // directly access DOM
    };

    return (
        <>
            <input ref={inputRef} />
            <button onClick={handleClick}>Focus Input</button>
        </>
    );
};
```

### useMemo & useCallback — Performance optimization
```jsx
// useMemo: memoize computed VALUE
const sortedList = useMemo(() => {
    return items.sort((a, b) => a.name.localeCompare(b.name));
}, [items]); // recompute only when items change

// useCallback: memoize FUNCTION reference
const handleDelete = useCallback((id) => {
    setItems(prev => prev.filter(item => item.id !== id));
}, []); // stable reference, won't cause re-renders in child
```

---

## 5. Component Lifecycle

```
Mounting:     constructor → render → DOM update → useEffect(,[])
Updating:     new props/state → render → DOM update → useEffect
Unmounting:   useEffect cleanup function
```

---

## 6. Event Handling & Forms

```jsx
const LoginForm = () => {
    const [formData, setFormData] = useState({ email: '', password: '' });
    const [errors, setErrors] = useState({});

    const handleChange = (e) => {
        const { name, value } = e.target;
        setFormData(prev => ({ ...prev, [name]: value }));
    };

    const handleSubmit = async (e) => {
        e.preventDefault(); // prevent page refresh
        if (!formData.email) {
            setErrors({ email: 'Email is required' });
            return;
        }
        const response = await fetch('/api/login', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(formData),
        });
        // handle response...
    };

    return (
        <form onSubmit={handleSubmit}>
            <input
                name="email"
                value={formData.email}
                onChange={handleChange}
                placeholder="Email"
            />
            {errors.email && <span className="error">{errors.email}</span>}

            <input
                name="password"
                type="password"
                value={formData.password}
                onChange={handleChange}
            />
            <button type="submit">Login</button>
        </form>
    );
};
```

---

## 7. API Integration

```jsx
// Custom hook for API calls
const useApi = (url) => {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    useEffect(() => {
        const controller = new AbortController();

        fetch(url, { signal: controller.signal })
            .then(res => {
                if (!res.ok) throw new Error(`HTTP ${res.status}`);
                return res.json();
            })
            .then(setData)
            .catch(err => {
                if (err.name !== 'AbortError') setError(err.message);
            })
            .finally(() => setLoading(false));

        return () => controller.abort(); // cleanup on unmount
    }, [url]);

    return { data, loading, error };
};

// Usage
const EmployeeList = () => {
    const { data: employees, loading, error } = useApi('/api/v1/employees');

    if (loading) return <Spinner />;
    if (error) return <Alert message={error} />;

    return (
        <ul>
            {employees.map(emp => <li key={emp.id}>{emp.name}</li>)}
        </ul>
    );
};
```

---

## 8. React Router

```jsx
import { BrowserRouter, Routes, Route, Link, useParams, useNavigate } from 'react-router-dom';

const App = () => (
    <BrowserRouter>
        <nav>
            <Link to="/">Home</Link>
            <Link to="/employees">Employees</Link>
        </nav>
        <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/employees" element={<EmployeeList />} />
            <Route path="/employees/:id" element={<EmployeeDetail />} />
            <Route path="*" element={<NotFound />} />
        </Routes>
    </BrowserRouter>
);

const EmployeeDetail = () => {
    const { id } = useParams();          // get URL params
    const navigate = useNavigate();       // programmatic navigation
    return (
        <div>
            <h1>Employee {id}</h1>
            <button onClick={() => navigate('/employees')}>Back</button>
        </div>
    );
};
```

---

## 9. State Management

### When to use what?
| State Type | Tool |
|-----------|------|
| Local component state | `useState` |
| Shared between few components | Lift state up + props |
| Global app state | Context API / Redux / Zustand |
| Server state (API data) | React Query / SWR |

---

## 10. Interview Questions & Answers

### Q1: What is the Virtual DOM?
**A:** A lightweight in-memory representation of the real DOM. When state changes, React creates a new Virtual DOM, diffs it with the previous one (reconciliation), and applies only the minimal changes to the real DOM. This makes updates efficient.

### Q2: What are React Hooks? Why were they introduced?
**A:** Hooks let you use state and lifecycle features in functional components (previously only in class components). Introduced to avoid class complexity, share stateful logic via custom hooks, and reduce wrapper hell (HOCs, render props).

### Q3: What is the difference between `useMemo` and `useCallback`?
**A:** `useMemo` memoizes a **computed value** (result of a function). `useCallback` memoizes a **function reference** itself. Both prevent unnecessary recalculations/re-renders based on dependency arrays.

### Q4: What is prop drilling and how do you avoid it?
**A:** Passing props through multiple intermediate components that don't use them. Solutions: Context API (`useContext`), state management libraries (Redux/Zustand), component composition.

### Q5: Why do we need `key` in lists?
**A:** Keys help React identify which items changed, were added, or removed. Without keys (or with index as key), React can't efficiently update the list and may cause bugs (wrong items updated). Use unique stable IDs.

---

> **Next:** [12_System_Design_Basics.md](12_System_Design_Basics.md) →
