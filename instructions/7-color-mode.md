# Chapter 7: Color Mode Migration - From Vanilla JS to React Context

## Introduction

In this final chapter, we'll migrate your existing color mode functionality from vanilla JavaScript to a React-based implementation. Building on the hooks knowledge from Chapter 5, we'll create a robust theme system using React Context, custom hooks, and proper TypeScript typing.

## Current Implementation Analysis

Let's examine your current `color-mode.js` implementation:

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

### Issues with Current Approach:
1. **Direct DOM Manipulation**: Not React-friendly
2. **No State Management**: State isn't tracked in React
3. **No Persistence**: Theme doesn't persist across sessions
4. **Manual Event Handling**: Using vanilla JS event listeners
5. **No Type Safety**: No TypeScript types
6. **Tight Coupling**: Hard to reuse in other components

## React-Based Implementation

### Benefits of React Approach:
- **State Management**: React state handles theme changes
- **Context API**: Share theme across all components
- **Hooks**: Clean, reusable logic
- **Persistence**: Save to localStorage automatically
- **Type Safety**: Full TypeScript support
- **Testability**: Easy to test and mock

## Step 1: Create Theme Types

Create `src/app/types/theme.ts`:

```typescript
export type Theme = 'light' | 'dark';

export interface ThemeContextType {
  theme: Theme;
  toggleTheme: () => void;
  setTheme: (theme: Theme) => void;
}

export interface ThemeProviderProps {
  children: React.ReactNode;
  defaultTheme?: Theme;
  storageKey?: string;
}
```

## Step 2: Create Theme Context

Create `src/app/contexts/ThemeContext.tsx`:

```typescript
'use client';

import React, { createContext, useContext, useState, useEffect } from 'react';
import { Theme, ThemeContextType, ThemeProviderProps } from '../types/theme';

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export function ThemeProvider({
  children,
  defaultTheme = 'light',
  storageKey = 'theme',
}: ThemeProviderProps) {
  const [theme, setThemeState] = useState<Theme>(defaultTheme);
  const [mounted, setMounted] = useState(false);

  // Handle hydration mismatch
  useEffect(() => {
    setMounted(true);
  }, []);

  // Load theme from localStorage on mount
  useEffect(() => {
    if (typeof window !== 'undefined') {
      try {
        const savedTheme = localStorage.getItem(storageKey) as Theme;
        if (savedTheme && (savedTheme === 'light' || savedTheme === 'dark')) {
          setThemeState(savedTheme);
        }
      } catch (error) {
        console.warn('Failed to load theme from localStorage:', error);
      }
    }
  }, [storageKey]);

  // Apply theme to document and save to localStorage
  useEffect(() => {
    if (mounted) {
      try {
        // Apply theme to document
        document.documentElement.setAttribute('data-theme', theme);
        document.body.setAttribute('data-theme', theme);
        
        // Save to localStorage
        localStorage.setItem(storageKey, theme);
        
        // Dispatch custom event for external listeners
        window.dispatchEvent(
          new CustomEvent('themeChange', { detail: { theme } })
        );
      } catch (error) {
        console.warn('Failed to apply theme:', error);
      }
    }
  }, [theme, mounted, storageKey]);

  const setTheme = (newTheme: Theme) => {
    setThemeState(newTheme);
  };

  const toggleTheme = () => {
    setThemeState(prevTheme => prevTheme === 'light' ? 'dark' : 'light');
  };

  const value: ThemeContextType = {
    theme,
    setTheme,
    toggleTheme,
  };

  // Prevent hydration mismatch by not rendering until mounted
  if (!mounted) {
    return <>{children}</>;
  }

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme(): ThemeContextType {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}
```

## Step 3: Create a Theme Hook with Advanced Features

Create `src/app/hooks/useTheme.ts`:

```typescript
'use client';

import { useEffect, useState } from 'react';
import { Theme } from '../types/theme';

interface UseThemeOptions {
  defaultTheme?: Theme;
  storageKey?: string;
  enableSystem?: boolean;
}

interface UseThemeReturn {
  theme: Theme;
  setTheme: (theme: Theme) => void;
  toggleTheme: () => void;
  systemTheme: Theme | null;
  resolvedTheme: Theme;
}

export function useAdvancedTheme({
  defaultTheme = 'light',
  storageKey = 'theme',
  enableSystem = true,
}: UseThemeOptions = {}): UseThemeReturn {
  const [theme, setThemeState] = useState<Theme>(defaultTheme);
  const [systemTheme, setSystemTheme] = useState<Theme | null>(null);
  const [mounted, setMounted] = useState(false);

  // Detect system theme preference
  useEffect(() => {
    if (!enableSystem || typeof window === 'undefined') return;

    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
    const updateSystemTheme = () => {
      setSystemTheme(mediaQuery.matches ? 'dark' : 'light');
    };

    updateSystemTheme();
    mediaQuery.addEventListener('change', updateSystemTheme);

    return () => mediaQuery.removeEventListener('change', updateSystemTheme);
  }, [enableSystem]);

  // Load saved theme
  useEffect(() => {
    setMounted(true);
    
    if (typeof window !== 'undefined') {
      try {
        const saved = localStorage.getItem(storageKey) as Theme;
        if (saved && (saved === 'light' || saved === 'dark')) {
          setThemeState(saved);
        }
      } catch (error) {
        console.warn('Failed to load theme:', error);
      }
    }
  }, [storageKey]);

  // Determine the resolved theme
  const resolvedTheme = theme;

  // Apply theme when it changes
  useEffect(() => {
    if (!mounted) return;

    try {
      document.documentElement.setAttribute('data-theme', resolvedTheme);
      document.body.setAttribute('data-theme', resolvedTheme);
      localStorage.setItem(storageKey, theme);
    } catch (error) {
      console.warn('Failed to apply theme:', error);
    }
  }, [resolvedTheme, theme, storageKey, mounted]);

  const setTheme = (newTheme: Theme) => {
    setThemeState(newTheme);
  };

  const toggleTheme = () => {
    setThemeState(prev => prev === 'light' ? 'dark' : 'light');
  };

  return {
    theme,
    setTheme,
    toggleTheme,
    systemTheme,
    resolvedTheme,
  };
}
```

## Step 4: Create Theme-Aware Components

### 1. Enhanced Switch Component

Update `src/app/components/ui/Switch/Switch.tsx`:

```typescript
'use client';

import React from 'react';
import { useTheme } from '../../contexts/ThemeContext';
import styles from './Switch.module.css';

interface SwitchProps {
  checked?: boolean;
  onChange?: (checked: boolean) => void;
  disabled?: boolean;
  className?: string;
  variant?: 'default' | 'theme';
  size?: 'small' | 'medium' | 'large';
}

export default function Switch({
  checked: controlledChecked,
  onChange,
  disabled = false,
  className = '',
  variant = 'default',
  size = 'medium',
}: SwitchProps) {
  const { theme, toggleTheme } = useTheme();
  
  // For theme variant, use theme context
  const isChecked = variant === 'theme' ? theme === 'dark' : controlledChecked;
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    if (variant === 'theme') {
      toggleTheme();
    } else if (onChange) {
      onChange(e.target.checked);
    }
  };

  const switchClasses = [
    styles.switch,
    styles[variant],
    styles[size],
    className,
  ].filter(Boolean).join(' ');

  return (
    <label className={switchClasses}>
      <input
        type="checkbox"
        checked={isChecked}
        onChange={handleChange}
        disabled={disabled}
        className={styles.input}
      />
      <div className={styles.toggle}>
        <div className={styles.iconLight}>
          <i className="fa-solid fa-moon"></i>
        </div>
        <div className={styles.iconDark}>
          <i className="fa-solid fa-sun"></i>
        </div>
      </div>
    </label>
  );
}
```

### 2. Theme Toggle Button Component

Create `src/app/components/ui/ThemeToggle/ThemeToggle.tsx`:

```typescript
'use client';

import React from 'react';
import { useTheme } from '../../contexts/ThemeContext';
import Button from '../Button/Button';
import Icon from '../Icon/Icon';
import styles from './ThemeToggle.module.css';

interface ThemeToggleProps {
  variant?: 'icon' | 'text' | 'switch';
  size?: 'small' | 'medium' | 'large';
  className?: string;
}

export default function ThemeToggle({
  variant = 'icon',
  size = 'medium',
  className = '',
}: ThemeToggleProps) {
  const { theme, toggleTheme } = useTheme();
  const isDark = theme === 'dark';

  if (variant === 'switch') {
    return (
      <div className={`${styles.switchContainer} ${className}`}>
        <Icon name="sun" size="small" />
        <label className={styles.switch}>
          <input
            type="checkbox"
            checked={isDark}
            onChange={toggleTheme}
            className={styles.switchInput}
          />
          <div className={styles.switchSlider}></div>
        </label>
        <Icon name="moon" size="small" />
      </div>
    );
  }

  if (variant === 'text') {
    return (
      <Button
        onClick={toggleTheme}
        variant="outline"
        size={size}
        className={`${styles.textButton} ${className}`}
      >
        <Icon name={isDark ? 'sun' : 'moon'} size="small" />
        Switch to {isDark ? 'Light' : 'Dark'} Mode
      </Button>
    );
  }

  return (
    <Button
      onClick={toggleTheme}
      variant="outline"
      size={size}
      className={`${styles.iconButton} ${className}`}
      aria-label={`Switch to ${isDark ? 'light' : 'dark'} mode`}
    >
      <Icon name={isDark ? 'sun' : 'moon'} />
    </Button>
  );
}
```

Create `src/app/components/ui/ThemeToggle/ThemeToggle.module.css`:

```css
.switchContainer {
  display: flex;
  align-items: center;
  gap: 8px;
}

.switch {
  position: relative;
  display: inline-block;
  width: 44px;
  height: 24px;
}

.switchInput {
  opacity: 0;
  width: 0;
  height: 0;
}

.switchSlider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  transition: 0.3s;
  border-radius: 24px;
}

.switchSlider:before {
  position: absolute;
  content: "";
  height: 18px;
  width: 18px;
  left: 3px;
  bottom: 3px;
  background-color: white;
  transition: 0.3s;
  border-radius: 50%;
}

.switchInput:checked + .switchSlider {
  background-color: #007bff;
}

.switchInput:checked + .switchSlider:before {
  transform: translateX(20px);
}

.textButton {
  display: flex;
  align-items: center;
  gap: 8px;
}

.iconButton {
  padding: 8px;
  min-width: auto;
}
```

## Step 5: Create Theme Status Component

Create `src/app/components/features/ThemeStatus/ThemeStatus.tsx`:

```typescript
'use client';

import React from 'react';
import { useTheme } from '../../contexts/ThemeContext';
import Icon from '../../ui/Icon/Icon';
import styles from './ThemeStatus.module.css';

interface ThemeStatusProps {
  showIcon?: boolean;
  showText?: boolean;
  className?: string;
}

export default function ThemeStatus({
  showIcon = true,
  showText = true,
  className = '',
}: ThemeStatusProps) {
  const { theme } = useTheme();
  const isDark = theme === 'dark';

  return (
    <div className={`${styles.status} ${className}`}>
      {showIcon && (
        <Icon
          name={isDark ? 'moon' : 'sun'}
          className={styles.icon}
        />
      )}
      {showText && (
        <span className={styles.text}>
          {isDark ? 'Dark' : 'Light'} Mode
        </span>
      )}
    </div>
  );
}
```

Create `src/app/components/features/ThemeStatus/ThemeStatus.module.css`:

```css
.status {
  display: flex;
  align-items: center;
  gap: 8px;
}

.icon {
  color: var(--theme-icon-color);
}

.text {
  font-size: 14px;
  font-weight: 500;
  color: var(--text-primary);
}
```

## Step 6: Update Your Layout with Theme Support

Update your `src/app/components/layout/Navbar/Navbar.tsx`:

```typescript
'use client';

import React from 'react';
import Profile from '../../features/Profile/Profile';
import Search from '../../features/Search/Search';
import NavigationItem from '../../features/Navigation/NavigationItem';
import ThemeToggle from '../../ui/ThemeToggle/ThemeToggle';
import ThemeStatus from '../../features/ThemeStatus/ThemeStatus';
import styles from './Navbar.module.css';

interface NavItem {
  href: string;
  icon: string;
  label: string;
}

interface NavbarProps {
  user: {
    name: string;
    email: string;
  };
  navigationItems: NavItem[];
  currentPath?: string;
  onSearch?: (query: string) => void;
  className?: string;
}

export default function Navbar({
  user,
  navigationItems,
  currentPath = '/',
  onSearch,
  className = '',
}: NavbarProps) {
  return (
    <nav className={`${styles.navbar} ${className}`}>
      <div className={styles.top}>
        <Profile
          name={user.name}
          email={user.email}
          className={styles.profile}
        />

        <Search
          placeholder="Search..."
          onSearch={onSearch}
          className={styles.search}
        />

        <ul className={styles.navList}>
          {navigationItems.map((item) => (
            <NavigationItem
              key={item.href}
              href={item.href}
              icon={item.icon}
              label={item.label}
              isActive={currentPath === item.href}
            />
          ))}
        </ul>
      </div>

      <div className={styles.bottom}>
        <ul className={styles.navList}>
          <NavigationItem
            href="/logout"
            icon="right-from-bracket"
            label="Logout"
          />
        </ul>

        <div className={styles.themeControl}>
          <ThemeStatus />
          <ThemeToggle variant="switch" />
        </div>
      </div>
    </nav>
  );
}
```

## Step 7: Create Root Layout with Theme Provider

Update `src/app/layout.tsx`:

```typescript
import type { Metadata } from 'next'
import { ThemeProvider } from './contexts/ThemeContext'
import './globals.css'
import './style.css'
import './styles/themes.css'

export const metadata: Metadata = {
  title: 'iva',
  description: 'Iva UI Dashboard',
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <head>
        <link
          rel="stylesheet"
          href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.7.2/css/all.min.css"
        />
      </head>
      <body>
        <ThemeProvider defaultTheme="light" storageKey="iva-theme">
          {children}
        </ThemeProvider>
      </body>
    </html>
  )
}
```

## Step 8: Enhanced CSS Variables for Theme System

Update `src/app/styles/themes.css`:

```css
:root {
  /* Color tokens */
  --color-primary-50: #eff6ff;
  --color-primary-500: #3b82f6;
  --color-primary-600: #2563eb;
  --color-primary-700: #1d4ed8;
  
  --color-gray-50: #f9fafb;
  --color-gray-100: #f3f4f6;
  --color-gray-200: #e5e7eb;
  --color-gray-300: #d1d5db;
  --color-gray-400: #9ca3af;
  --color-gray-500: #6b7280;
  --color-gray-600: #4b5563;
  --color-gray-700: #374151;
  --color-gray-800: #1f2937;
  --color-gray-900: #111827;

  /* Light theme semantic tokens */
  --background-primary: var(--color-gray-50);
  --background-secondary: #ffffff;
  --background-elevated: #ffffff;
  --text-primary: var(--color-gray-900);
  --text-secondary: var(--color-gray-600);
  --text-tertiary: var(--color-gray-500);
  --border-primary: var(--color-gray-200);
  --border-secondary: var(--color-gray-300);
  
  /* Component-specific tokens */
  --navbar-bg: var(--background-secondary);
  --main-bg: var(--background-primary);
  --border-color: var(--border-primary);
  --theme-control-bg: var(--background-elevated);
  --theme-icon-color: var(--text-secondary);
  --nav-text-color: var(--text-primary);
  --nav-hover-bg: var(--color-gray-100);
  --nav-hover-text: var(--text-primary);
  --nav-active-bg: var(--color-primary-500);
  --nav-active-text: #ffffff;
  --profile-icon-color: var(--text-secondary);
  
  /* Transition */
  color-scheme: light;
  transition: background-color 0.3s ease, color 0.3s ease;
}

[data-theme="dark"] {
  /* Dark theme semantic tokens */
  --background-primary: var(--color-gray-900);
  --background-secondary: var(--color-gray-800);
  --background-elevated: var(--color-gray-700);
  --text-primary: var(--color-gray-50);
  --text-secondary: var(--color-gray-300);
  --text-tertiary: var(--color-gray-400);
  --border-primary: var(--color-gray-700);
  --border-secondary: var(--color-gray-600);
  
  /* Component-specific tokens */
  --navbar-bg: var(--background-secondary);
  --main-bg: var(--background-primary);
  --border-color: var(--border-primary);
  --theme-control-bg: var(--background-elevated);
  --theme-icon-color: var(--text-secondary);
  --nav-text-color: var(--text-primary);
  --nav-hover-bg: var(--color-gray-700);
  --nav-hover-text: var(--text-primary);
  --nav-active-bg: var(--color-primary-600);
  --nav-active-text: #ffffff;
  --profile-icon-color: var(--text-secondary);
  
  /* Color scheme */
  color-scheme: dark;
}

/* Smooth transitions for theme changes */
* {
  transition: background-color 0.3s ease, 
              border-color 0.3s ease, 
              color 0.3s ease;
}

/* Prevent transition on page load */
.no-transition * {
  transition: none !important;
}
```

## Step 9: Create Theme Persistence Hook

Create `src/app/hooks/useThemePersistence.ts`:

```typescript
'use client';

import { useEffect } from 'react';
import { Theme } from '../types/theme';

interface UseThemePersistenceOptions {
  theme: Theme;
  storageKey?: string;
  onLoad?: (theme: Theme) => void;
}

export function useThemePersistence({
  theme,
  storageKey = 'theme',
  onLoad,
}: UseThemePersistenceOptions) {
  // Save theme to localStorage
  useEffect(() => {
    try {
      localStorage.setItem(storageKey, theme);
    } catch (error) {
      console.warn('Failed to save theme to localStorage:', error);
    }
  }, [theme, storageKey]);

  // Load theme from localStorage on mount
  useEffect(() => {
    try {
      const savedTheme = localStorage.getItem(storageKey) as Theme;
      if (savedTheme && onLoad) {
        onLoad(savedTheme);
      }
    } catch (error) {
      console.warn('Failed to load theme from localStorage:', error);
    }
  }, [storageKey, onLoad]);

  // Sync across tabs
  useEffect(() => {
    const handleStorageChange = (e: StorageEvent) => {
      if (e.key === storageKey && e.newValue && onLoad) {
        const newTheme = e.newValue as Theme;
        if (newTheme === 'light' || newTheme === 'dark') {
          onLoad(newTheme);
        }
      }
    };

    window.addEventListener('storage', handleStorageChange);
    return () => window.removeEventListener('storage', handleStorageChange);
  }, [storageKey, onLoad]);
}
```

## Step 10: Update Your Pages

Update `src/app/page.tsx`:

```typescript
'use client';

import React from 'react';
import Layout from './components/layout/Layout/Layout';
import ThemeToggle from './components/ui/ThemeToggle/ThemeToggle';
import { useTheme } from './contexts/ThemeContext';

export default function HomePage() {
  const { theme } = useTheme();

  const user = {
    name: 'iva UI',
    email: 'ivagamilec@gmail.com',
  };

  return (
    <Layout user={user} currentPath="/">
      <div>
        <h1>Welcome to the Dashboard</h1>
        <p>Current theme: <strong>{theme}</strong></p>
        
        <div style={{ marginTop: '2rem', display: 'flex', gap: '1rem', flexWrap: 'wrap' }}>
          <ThemeToggle variant="icon" />
          <ThemeToggle variant="text" />
          <ThemeToggle variant="switch" />
        </div>
        
        <p style={{ marginTop: '2rem' }}>
          The theme automatically persists across page reloads and browser sessions.
          Try switching themes and refreshing the page!
        </p>
      </div>
    </Layout>
  );
}
```

## Step 11: Testing Your Theme Implementation

Create a test file `src/app/components/contexts/ThemeContext.test.tsx`:

```typescript
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import { ThemeProvider, useTheme } from './ThemeContext';

// Test component that uses the theme context
function TestComponent() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <div>
      <span data-testid="theme">{theme}</span>
      <button onClick={toggleTheme} data-testid="toggle">
        Toggle Theme
      </button>
    </div>
  );
}

// Mock localStorage
const localStorageMock = {
  getItem: jest.fn(),
  setItem: jest.fn(),
  clear: jest.fn(),
};
global.localStorage = localStorageMock as any;

describe('ThemeContext', () => {
  beforeEach(() => {
    localStorageMock.getItem.mockClear();
    localStorageMock.setItem.mockClear();
  });

  it('provides default theme', () => {
    render(
      <ThemeProvider defaultTheme="light">
        <TestComponent />
      </ThemeProvider>
    );

    expect(screen.getByTestId('theme')).toHaveTextContent('light');
  });

  it('toggles theme correctly', () => {
    render(
      <ThemeProvider defaultTheme="light">
        <TestComponent />
      </ThemeProvider>
    );

    const toggleButton = screen.getByTestId('toggle');
    const themeDisplay = screen.getByTestId('theme');

    expect(themeDisplay).toHaveTextContent('light');

    fireEvent.click(toggleButton);
    expect(themeDisplay).toHaveTextContent('dark');

    fireEvent.click(toggleButton);
    expect(themeDisplay).toHaveTextContent('light');
  });

  it('saves theme to localStorage', () => {
    render(
      <ThemeProvider defaultTheme="light">
        <TestComponent />
      </ThemeProvider>
    );

    const toggleButton = screen.getByTestId('toggle');
    fireEvent.click(toggleButton);

    expect(localStorageMock.setItem).toHaveBeenCalledWith('theme', 'dark');
  });

  it('throws error when used outside provider', () => {
    // Suppress console.error for this test
    const consoleSpy = jest.spyOn(console, 'error').mockImplementation();

    expect(() => {
      render(<TestComponent />);
    }).toThrow('useTheme must be used within a ThemeProvider');

    consoleSpy.mockRestore();
  });
});
```

## Step 12: Performance Optimizations

### 1. Prevent Theme Flash

Create `src/app/components/ui/ThemeScript/ThemeScript.tsx`:

```typescript
export default function ThemeScript() {
  const script = `
    (function() {
      try {
        var theme = localStorage.getItem('iva-theme');
        if (theme === 'dark' || theme === 'light') {
          document.documentElement.setAttribute('data-theme', theme);
          document.body.setAttribute('data-theme', theme);
        }
      } catch (e) {}
    })();
  `;

  return <script dangerouslySetInnerHTML={{ __html: script }} />;
}
```

Add to your `src/app/layout.tsx`:

```typescript
import ThemeScript from './components/ui/ThemeScript/ThemeScript';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <head>
        <ThemeScript />
        <link
          rel="stylesheet"
          href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.7.2/css/all.min.css"
        />
      </head>
      <body>
        <ThemeProvider defaultTheme="light" storageKey="iva-theme">
          {children}
        </ThemeProvider>
      </body>
    </html>
  )
}
```

## Step 13: Comparison: Before vs After

### Before (Vanilla JS):
```javascript
// Direct DOM manipulation
const body = document.body;
const switchInput = document.querySelector('.switch .switch__input');

// Manual event handling
switchInput.addEventListener("click", () => {
  const theme = body.dataset.theme;
  body.dataset.theme = theme === "light" ? "dark" : "light";
});
```

### After (React):
```typescript
// State management with Context
const { theme, toggleTheme } = useTheme();

// Component-based approach
<ThemeToggle variant="switch" />

// Type safety
interface ThemeContextType {
  theme: Theme;
  toggleTheme: () => void;
}
```

## Benefits of the React Implementation

### 1. **Declarative**: Components describe what they should look like
### 2. **Type Safe**: Full TypeScript support prevents errors
### 3. **Reusable**: Theme logic can be used anywhere
### 4. **Testable**: Easy to test with React Testing Library
### 5. **Persistent**: Automatically saves and loads from localStorage
### 6. **Performant**: Only re-renders when theme changes
### 7. **Accessible**: Proper ARIA labels and keyboard support

## Advanced Features

### 1. System Theme Detection
```typescript
const prefersDark = window.matchMedia('(prefers-color-scheme: dark)');
```

### 2. Multiple Theme Support
```typescript
type Theme = 'light' | 'dark' | 'auto' | 'high-contrast';
```

### 3. Theme Animations
```css
@media (prefers-reduced-motion: no-preference) {
  * {
    transition: background-color 0.3s ease, color 0.3s ease;
  }
}
```

## Summary

You've successfully migrated from vanilla JavaScript to a robust React-based theme system that includes:

- **React Context** for global state management
- **TypeScript** for type safety
- **localStorage** persistence
- **Multiple toggle variants**
- **Smooth transitions**
- **Testing support**
- **Performance optimizations**

## Next Steps

Your migration from HTML/CSS/JS to Next.js is now complete! You have:

1. ✅ Set up Next.js with proper configuration
2. ✅ Configured ESLint for code quality
3. ✅ Set up Prettier for consistent formatting
4. ✅ Added TypeScript for type safety
5. ✅ Extracted components with proper organization
6. ✅ Implemented a robust theme system

### Recommended Next Steps:
- Add unit tests for your components
- Implement Storybook for component documentation
- Add accessibility improvements
- Set up continuous integration/deployment
- Consider adding a component library like Chakra UI or Material-UI
- Explore Next.js features like API routes and middleware

Congratulations on completing your migration to Next.js! 🎉