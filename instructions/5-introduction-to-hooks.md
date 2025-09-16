# Chapter 5: Introduction to React Hooks

## What are React Hooks?

React Hooks are functions that let you use state and other React features in functional components. They were introduced in React 16.8 and have become the standard way to write React components. Hooks allow you to:

- **Add state** to functional components
- **Manage side effects** like API calls and subscriptions
- **Reuse stateful logic** between components
- **Optimize performance** with memoization
- **Access React features** without writing class components

### Why Hooks Matter for Your Project

In your migration from vanilla JavaScript to React, hooks will replace:
- Direct DOM manipulation with state management
- Event listeners with effect hooks
- Manual data persistence with custom hooks
- Complex lifecycle management with simpler hook patterns

## Step 1: useState - Managing Component State

The `useState` hook lets you add state to functional components.

### Basic useState Examples

```typescript
'use client';

import { useState } from 'react';

// Simple counter component
export default function Counter() {
  const [count, setCount] = useState<number>(0);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);
  const reset = () => setCount(0);

  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

### useState with Complex State

```typescript
'use client';

import { useState } from 'react';

interface User {
  name: string;
  email: string;
  preferences: {
    theme: 'light' | 'dark';
    notifications: boolean;
  };
}

export default function UserProfile() {
  const [user, setUser] = useState<User>({
    name: '',
    email: '',
    preferences: {
      theme: 'light',
      notifications: true
    }
  });

  const updateName = (name: string) => {
    setUser(prev => ({
      ...prev,
      name
    }));
  };

  const toggleTheme = () => {
    setUser(prev => ({
      ...prev,
      preferences: {
        ...prev.preferences,
        theme: prev.preferences.theme === 'light' ? 'dark' : 'light'
      }
    }));
  };

  return (
    <div>
      <input
        type="text"
        value={user.name}
        onChange={(e) => updateName(e.target.value)}
        placeholder="Enter your name"
      />
      <p>Current theme: {user.preferences.theme}</p>
      <button onClick={toggleTheme}>Toggle Theme</button>
    </div>
  );
}
```

### useState Best Practices

```typescript
// ✅ Good: Functional updates for dependent state changes
const [count, setCount] = useState(0);
const increment = () => setCount(prev => prev + 1);

// ❌ Avoid: Direct state access in updates
const increment = () => setCount(count + 1);

// ✅ Good: Separate state for independent values
const [name, setName] = useState('');
const [email, setEmail] = useState('');

// ❌ Avoid: Single state object for unrelated values
const [form, setForm] = useState({ name: '', email: '', unrelatedData: '' });
```

## Step 2: useEffect - Handling Side Effects

The `useEffect` hook lets you perform side effects in function components.

### Basic useEffect Examples

```typescript
'use client';

import { useState, useEffect } from 'react';

// Document title update
export default function DocumentTitle() {
  const [title, setTitle] = useState('My App');

  useEffect(() => {
    document.title = title;
  }, [title]); // Effect runs when title changes

  return (
    <div>
      <input
        type="text"
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Enter page title"
      />
    </div>
  );
}
```

### useEffect with Cleanup

```typescript
'use client';

import { useState, useEffect } from 'react';

export default function WindowSize() {
  const [windowSize, setWindowSize] = useState({
    width: 0,
    height: 0
  });

  useEffect(() => {
    // Function to update size
    const updateSize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };

    // Set initial size
    updateSize();

    // Add event listener
    window.addEventListener('resize', updateSize);

    // Cleanup function - removes event listener
    return () => {
      window.removeEventListener('resize', updateSize);
    };
  }, []); // Empty dependency array = runs once on mount

  return (
    <div>
      <p>Window size: {windowSize.width} x {windowSize.height}</p>
    </div>
  );
}
```

### useEffect Dependency Patterns

```typescript
'use client';

import { useState, useEffect } from 'react';

export default function EffectPatterns() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // No dependencies - runs after every render
  useEffect(() => {
    console.log('Runs after every render');
  });

  // Empty dependencies - runs once after mount
  useEffect(() => {
    console.log('Runs once after mount');
  }, []);

  // Specific dependencies - runs when dependencies change
  useEffect(() => {
    console.log('Count changed:', count);
  }, [count]);

  // Multiple dependencies
  useEffect(() => {
    console.log('Count or name changed:', { count, name });
  }, [count, name]);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Enter name"
      />
    </div>
  );
}
```

## Step 3: Theme Management with Hooks

Let's recreate your color mode functionality using hooks:

```typescript
'use client';

import { useState, useEffect } from 'react';

type Theme = 'light' | 'dark';

export function useTheme() {
  const [theme, setTheme] = useState<Theme>('light');

  // Load theme from localStorage on mount
  useEffect(() => {
    const savedTheme = localStorage.getItem('theme') as Theme;
    if (savedTheme === 'light' || savedTheme === 'dark') {
      setTheme(savedTheme);
    }
  }, []);

  // Apply theme to document and save to localStorage
  useEffect(() => {
    document.body.setAttribute('data-theme', theme);
    localStorage.setItem('theme', theme);
  }, [theme]);

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return { theme, setTheme, toggleTheme };
}

// Component using the theme hook
export default function ThemeToggle() {
  const { theme, toggleTheme } = useTheme();

  return (
    <div>
      <p>Current theme: {theme}</p>
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'} mode
      </button>
    </div>
  );
}
```

## Step 4: Data Fetching with useEffect

```typescript
'use client';

import { useState, useEffect } from 'react';

interface User {
  id: number;
  name: string;
  email: string;
}

interface FetchState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

export default function UserList() {
  const [state, setState] = useState<FetchState<User[]>>({
    data: null,
    loading: true,
    error: null
  });

  useEffect(() => {
    const fetchUsers = async () => {
      try {
        setState(prev => ({ ...prev, loading: true, error: null }));
        
        const response = await fetch('/api/users');
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const users = await response.json();
        setState({ data: users, loading: false, error: null });
      } catch (error) {
        setState({
          data: null,
          loading: false,
          error: error instanceof Error ? error.message : 'An error occurred'
        });
      }
    };

    fetchUsers();
  }, []);

  if (state.loading) return <div>Loading...</div>;
  if (state.error) return <div>Error: {state.error}</div>;
  if (!state.data) return <div>No data</div>;

  return (
    <ul>
      {state.data.map(user => (
        <li key={user.id}>
          {user.name} - {user.email}
        </li>
      ))}
    </ul>
  );
}
```

## Step 5: Custom Hooks

Custom hooks let you extract component logic into reusable functions.

### useLocalStorage Hook

```typescript
'use client';

import { useState, useEffect } from 'react';

function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') {
      return initialValue;
    }

    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      
      if (typeof window !== 'undefined') {
        window.localStorage.setItem(key, JSON.stringify(valueToStore));
      }
    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error);
    }
  };

  return [storedValue, setValue] as const;
}

// Usage example
export default function Settings() {
  const [name, setName] = useLocalStorage('userName', '');
  const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light');

  return (
    <div>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Enter your name"
      />
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Current theme: {theme}
      </button>
    </div>
  );
}
```

### useApi Hook

```typescript
'use client';

import { useState, useEffect } from 'react';

interface ApiState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

function useApi<T>(url: string): ApiState<T> & { refetch: () => void } {
  const [state, setState] = useState<ApiState<T>>({
    data: null,
    loading: true,
    error: null
  });

  const fetchData = async () => {
    try {
      setState(prev => ({ ...prev, loading: true, error: null }));
      
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const data = await response.json();
      setState({ data, loading: false, error: null });
    } catch (error) {
      setState({
        data: null,
        loading: false,
        error: error instanceof Error ? error.message : 'An error occurred'
      });
    }
  };

  useEffect(() => {
    fetchData();
  }, [url]);

  return { ...state, refetch: fetchData };
}

// Usage example
export default function ProductList() {
  const { data: products, loading, error, refetch } = useApi<Product[]>('/api/products');

  if (loading) return <div>Loading products...</div>;
  if (error) return <div>Error: {error} <button onClick={refetch}>Retry</button></div>;

  return (
    <div>
      <button onClick={refetch}>Refresh</button>
      <ul>
        {products?.map(product => (
          <li key={product.id}>{product.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

## Step 6: useContext Hook - Sharing State Across Components

The `useContext` hook lets you share state between components without passing props down through every level. This is perfect for themes, user authentication, and other global state.

### Basic useContext Example

```typescript
'use client';

import { createContext, useContext, useState, ReactNode } from 'react';

// 1. Create a context
interface UserContextType {
  user: string;
  setUser: (user: string) => void;
}

const UserContext = createContext<UserContextType | undefined>(undefined);

// 2. Create a provider component
interface UserProviderProps {
  children: ReactNode;
}

export function UserProvider({ children }: UserProviderProps) {
  const [user, setUser] = useState('Guest');

  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
}

// 3. Create a custom hook to use the context
export function useUser() {
  const context = useContext(UserContext);
  if (context === undefined) {
    throw new Error('useUser must be used within a UserProvider');
  }
  return context;
}

// 4. Use the context in components
function UserProfile() {
  const { user } = useUser();
  
  return <h2>Welcome, {user}!</h2>;
}

function LoginForm() {
  const { user, setUser } = useUser();
  const [inputValue, setInputValue] = useState('');

  const handleLogin = () => {
    setUser(inputValue);
    setInputValue('');
  };

  return (
    <div>
      <input
        type="text"
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        placeholder="Enter your name"
      />
      <button onClick={handleLogin}>Login</button>
      <p>Current user: {user}</p>
    </div>
  );
}

// 5. Wrap your app with the provider
export default function App() {
  return (
    <UserProvider>
      <UserProfile />
      <LoginForm />
    </UserProvider>
  );
}
```

### Theme Context for Your Project

Let's create a theme context that will replace your vanilla JavaScript color mode:

```typescript
'use client';

import { createContext, useContext, useState, useEffect, ReactNode } from 'react';

type Theme = 'light' | 'dark';

interface ThemeContextType {
  theme: Theme;
  toggleTheme: () => void;
  setTheme: (theme: Theme) => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

interface ThemeProviderProps {
  children: ReactNode;
}

export function ThemeProvider({ children }: ThemeProviderProps) {
  const [theme, setTheme] = useState<Theme>('light');

  // Load theme from localStorage on mount
  useEffect(() => {
    const savedTheme = localStorage.getItem('theme') as Theme;
    if (savedTheme === 'light' || savedTheme === 'dark') {
      setTheme(savedTheme);
    }
  }, []);

  // Apply theme to document and save to localStorage
  useEffect(() => {
    document.body.setAttribute('data-theme', theme);
    localStorage.setItem('theme', theme);
  }, [theme]);

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}

// Theme switch component
export function ThemeSwitch() {
  const { theme, toggleTheme } = useTheme();

  return (
    <label className="switch">
      <input
        type="checkbox"
        checked={theme === 'dark'}
        onChange={toggleTheme}
        className="switch__input"
      />
      <div className="switch__toggle">
        <div className="switch__icon switch__icon--light">
          <i className="fa-solid fa-moon"></i>
        </div>
        <div className="switch__icon switch__icon--dark">
          <i className="fa-solid fa-sun"></i>
        </div>
      </div>
    </label>
  );
}

// Any component can now access theme
export function Header() {
  const { theme } = useTheme();
  
  return (
    <header className={`header header--${theme}`}>
      <h1>My Website</h1>
      <ThemeSwitch />
    </header>
  );
}
```

### Multiple Contexts Example

You can have multiple contexts for different concerns:

```typescript
'use client';

import { createContext, useContext, useState, ReactNode } from 'react';

// Settings Context
interface Settings {
  language: 'en' | 'es' | 'fr';
  notifications: boolean;
}

interface SettingsContextType {
  settings: Settings;
  updateSetting: <K extends keyof Settings>(key: K, value: Settings[K]) => void;
}

const SettingsContext = createContext<SettingsContextType | undefined>(undefined);

export function SettingsProvider({ children }: { children: ReactNode }) {
  const [settings, setSettings] = useState<Settings>({
    language: 'en',
    notifications: true
  });

  const updateSetting = <K extends keyof Settings>(key: K, value: Settings[K]) => {
    setSettings(prev => ({ ...prev, [key]: value }));
  };

  return (
    <SettingsContext.Provider value={{ settings, updateSetting }}>
      {children}
    </SettingsContext.Provider>
  );
}

export function useSettings() {
  const context = useContext(SettingsContext);
  if (!context) {
    throw new Error('useSettings must be used within SettingsProvider');
  }
  return context;
}

// Component using both theme and settings
export function SettingsPanel() {
  const { theme } = useTheme();
  const { settings, updateSetting } = useSettings();

  return (
    <div className={`settings-panel settings-panel--${theme}`}>
      <h3>Settings</h3>
      
      <div>
        <label>Language:</label>
        <select 
          value={settings.language}
          onChange={(e) => updateSetting('language', e.target.value as Settings['language'])}
        >
          <option value="en">English</option>
          <option value="es">Spanish</option>
          <option value="fr">French</option>
        </select>
      </div>

      <div>
        <label>
          <input
            type="checkbox"
            checked={settings.notifications}
            onChange={(e) => updateSetting('notifications', e.target.checked)}
          />
          Enable notifications
        </label>
      </div>
    </div>
  );
}
```

## Step 7: useRef Hook

The `useRef` hook lets you access DOM elements directly and persist values across renders.

```typescript
'use client';

import { useState, useRef, useEffect } from 'react';

export default function SearchWithFocus() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState<string[]>([]);
  const inputRef = useRef<HTMLInputElement>(null);
  const searchCount = useRef(0);

  // Focus input on mount
  useEffect(() => {
    if (inputRef.current) {
      inputRef.current.focus();
    }
  }, []);

  const handleSearch = () => {
    searchCount.current += 1;
    console.log(`Search #${searchCount.current}: ${query}`);
    
    // Simulate search
    setResults([
      `Result 1 for "${query}"`,
      `Result 2 for "${query}"`,
      `Result 3 for "${query}"`
    ]);
  };

  const clearAndFocus = () => {
    setQuery('');
    setResults([]);
    if (inputRef.current) {
      inputRef.current.focus();
    }
  };

  return (
    <div>
      <div>
        <input
          ref={inputRef}
          type="text"
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          placeholder="Search..."
        />
        <button onClick={handleSearch}>Search</button>
        <button onClick={clearAndFocus}>Clear</button>
      </div>
      
      <ul>
        {results.map((result, index) => (
          <li key={index}>{result}</li>
        ))}
      </ul>
    </div>
  );
}
```

## Step 8: Converting Your Existing JavaScript

Let's convert your original color-mode.js to React hooks:

### Original JavaScript:

```javascript
export function colorMode() {
  const body = document.body;
  const switchInput = document.querySelector(
    '.switch .switch__input[type="checkbox"]'
  );
  switchInput.addEventListener("click", () => {
    const theme = body.dataset.theme;

    if (theme === "light") {
      body.dataset.theme = "dark";
    } else {
      body.dataset.theme = "light";
    }
    console.log(body, switchInput, body.dataset.theme);
  });
}
```

### React Hooks Version:

```typescript
'use client';

import { useState, useEffect } from 'react';

type Theme = 'light' | 'dark';

export function useColorMode() {
  const [theme, setTheme] = useState<Theme>('light');

  // Load saved theme on mount
  useEffect(() => {
    const savedTheme = localStorage.getItem('theme') as Theme;
    if (savedTheme === 'light' || savedTheme === 'dark') {
      setTheme(savedTheme);
    }
  }, []);

  // Apply theme to body and save to localStorage
  useEffect(() => {
    document.body.setAttribute('data-theme', theme);
    localStorage.setItem('theme', theme);
    console.log('Theme changed to:', theme);
  }, [theme]);

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return { theme, setTheme, toggleTheme };
}

// Switch component using the hook
export default function ThemeSwitch() {
  const { theme, toggleTheme } = useColorMode();

  return (
    <label className="switch">
      <input
        type="checkbox"
        checked={theme === 'dark'}
        onChange={toggleTheme}
        className="switch__input"
      />
      <div className="switch__toggle">
        <div className="switch__icon switch__icon--light">
          <i className="fa-solid fa-moon"></i>
        </div>
        <div className="switch__icon switch__icon--dark">
          <i className="fa-solid fa-sun"></i>
        </div>
      </div>
    </label>
  );
}
```

## Step 9: Hook Rules and Best Practices

### Rules of Hooks:

1. **Only call hooks at the top level** - Never inside loops, conditions, or nested functions
2. **Only call hooks from React functions** - Components or custom hooks

```typescript
// ✅ Good
function MyComponent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');
  
  useEffect(() => {
    // Effect logic
  }, []);

  return <div>{count}</div>;
}

// ❌ Bad - conditional hook
function MyComponent() {
  if (someCondition) {
    const [count, setCount] = useState(0); // Don't do this!
  }
}

// ❌ Bad - hook in loop
function MyComponent() {
  for (let i = 0; i < 5; i++) {
    const [count, setCount] = useState(0); // Don't do this!
  }
}
```

### Best Practices:

1. **Use custom hooks for reusable logic**
2. **Keep effects focused and specific**
3. **Always include dependencies in useEffect**
4. **Use TypeScript for better hook typing**

```typescript
// ✅ Good: Focused effect with proper dependencies
useEffect(() => {
  const fetchUser = async () => {
    const user = await api.getUser(userId);
    setUser(user);
  };
  
  fetchUser();
}, [userId]); // Include all dependencies

// ✅ Good: Custom hook for reusable logic
function useWindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 });
  
  useEffect(() => {
    const updateSize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };
    
    window.addEventListener('resize', updateSize);
    updateSize();
    
    return () => window.removeEventListener('resize', updateSize);
  }, []);
  
  return size;
}
```

## Benefits of Using Hooks

1. **Simpler than class components** - No need for `this` binding
2. **Reusable logic** - Custom hooks can be shared between components
3. **Better TypeScript support** - Easier to type than class components
4. **Smaller bundle size** - No class overhead
5. **Easier testing** - Logic can be extracted and tested independently

## Next Steps

In the next chapter, we'll use these hooks to extract your HTML components into reusable React components with proper state management, focusing on component organization and file collocation strategies.

## Summary

React Hooks provide a powerful way to add state and side effects to functional components. The hooks you learned in this chapter will replace most of your vanilla JavaScript logic:

- **useState** replaces direct DOM state manipulation
- **useEffect** replaces event listeners and lifecycle methods  
- **useRef** replaces direct DOM element access
- **useContext** enables global state sharing without prop drilling
- **Custom hooks** encapsulate and reuse complex logic

With these tools, you can build interactive, stateful components that are easier to test, maintain, and reason about than vanilla JavaScript alternatives.