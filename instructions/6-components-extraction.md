# Chapter 6: Component Extraction and Organization

## Introduction

Component extraction is the process of breaking down your monolithic HTML structure into smaller, reusable React components. This chapter builds on the hooks knowledge from Chapter 5 to show you how to organize your codebase with proper file structure, component collocation, and separation of concerns.

## Why Extract Components?

### Benefits:
- **Reusability**: Use components across different pages
- **Maintainability**: Easier to update and debug smaller pieces
- **Testability**: Test individual components in isolation
- **Collaboration**: Team members can work on different components
- **Performance**: Better code splitting and lazy loading opportunities
- **Readability**: Smaller, focused components are easier to understand

### Component Thinking:
Instead of thinking in terms of pages, think in terms of components that can be composed together.

## Step 1: Analyzing Your Current Structure

Let's look at your current HTML structure and identify potential components:

```html
<!-- Your current index.html structure -->
<body data-theme="light">
  <nav class="navbar">
    <!-- Navigation content -->
  </nav>
</body>
```

### Potential Components:
1. **Layout Components**: `Navbar`, `Sidebar`, `Footer`
2. **UI Components**: `Button`, `Input`, `Icon`, `Switch`
3. **Business Components**: `Profile`, `SearchBox`, `NavigationItem`
4. **Page Components**: `Dashboard`, `Products`, `Analytics`

## Step 2: Component Hierarchy and Organization

### Recommended Folder Structure:

```
src/
├── app/
│   ├── components/           # Shared components
│   │   ├── ui/              # Basic UI components
│   │   │   ├── Button/
│   │   │   │   ├── Button.tsx
│   │   │   │   ├── Button.module.css
│   │   │   │   └── index.ts
│   │   │   ├── Input/
│   │   │   └── Switch/
│   │   ├── layout/          # Layout components
│   │   │   ├── Navbar/
│   │   │   ├── Sidebar/
│   │   │   └── Footer/
│   │   └── features/        # Business logic components
│   │       ├── Profile/
│   │       ├── Search/
│   │       └── Navigation/
│   ├── hooks/               # Custom hooks
│   ├── contexts/            # React contexts
│   ├── types/               # TypeScript types
│   ├── utils/               # Utility functions
│   └── styles/              # Global styles
```

## Step 3: The Collocation Principle

**Collocation** means keeping related files close together. Each component should have its own folder containing:

- `Component.tsx` - The React component
- `Component.module.css` - Component-specific styles
- `index.ts` - Export barrel
- `Component.test.tsx` - Tests (optional)
- `Component.stories.tsx` - Storybook stories (optional)

### Example: Button Component

```
src/app/components/ui/Button/
├── Button.tsx
├── Button.module.css
├── index.ts
└── Button.test.tsx
```

## Step 4: Creating Basic UI Components

### 1. Button Component

Create `src/app/components/ui/Button/Button.tsx`:

```typescript
import React from 'react';
import styles from './Button.module.css';

interface ButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  onClick?: () => void;
  type?: 'button' | 'submit' | 'reset';
  className?: string;
}

export default function Button({
  children,
  variant = 'primary',
  size = 'medium',
  disabled = false,
  onClick,
  type = 'button',
  className = '',
}: ButtonProps) {
  const buttonClasses = [
    styles.button,
    styles[variant],
    styles[size],
    className,
  ].filter(Boolean).join(' ');

  return (
    <button
      type={type}
      className={buttonClasses}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
}
```

Create `src/app/components/ui/Button/Button.module.css`:

```css
.button {
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.2s ease-in-out;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  text-decoration: none;
}

.button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* Variants */
.primary {
  background-color: #007bff;
  color: white;
}

.primary:hover:not(:disabled) {
  background-color: #0056b3;
}

.secondary {
  background-color: #6c757d;
  color: white;
}

.secondary:hover:not(:disabled) {
  background-color: #545b62;
}

.outline {
  background-color: transparent;
  color: #007bff;
  border: 2px solid #007bff;
}

.outline:hover:not(:disabled) {
  background-color: #007bff;
  color: white;
}

/* Sizes */
.small {
  padding: 8px 16px;
  font-size: 14px;
}

.medium {
  padding: 12px 24px;
  font-size: 16px;
}

.large {
  padding: 16px 32px;
  font-size: 18px;
}
```

Create `src/app/components/ui/Button/index.ts`:

```typescript
export { default } from './Button';
export type { ButtonProps } from './Button';
```

### 2. Icon Component

Create `src/app/components/ui/Icon/Icon.tsx`:

```typescript
import React from 'react';
import styles from './Icon.module.css';

interface IconProps {
  name: string;
  size?: 'small' | 'medium' | 'large';
  className?: string;
  onClick?: () => void;
}

export default function Icon({
  name,
  size = 'medium',
  className = '',
  onClick,
}: IconProps) {
  const iconClasses = [
    'fa-solid',
    `fa-${name}`,
    styles.icon,
    styles[size],
    className,
  ].filter(Boolean).join(' ');

  return (
    <i
      className={iconClasses}
      onClick={onClick}
      role={onClick ? "button" : undefined}
      tabIndex={onClick ? 0 : undefined}
    />
  );
}
```

Create `src/app/components/ui/Icon/Icon.module.css`:

```css
.icon {
  display: inline-block;
  cursor: default;
}

.icon[role="button"] {
  cursor: pointer;
  transition: color 0.2s ease;
}

.icon[role="button"]:hover {
  color: #007bff;
}

.small {
  font-size: 14px;
}

.medium {
  font-size: 16px;
}

.large {
  font-size: 20px;
}
```

### 3. Switch Component

Create `src/app/components/ui/Switch/Switch.tsx`:

```typescript
import React from 'react';
import styles from './Switch.module.css';

interface SwitchProps {
  checked: boolean;
  onChange: (checked: boolean) => void;
  disabled?: boolean;
  className?: string;
}

export default function Switch({
  checked,
  onChange,
  disabled = false,
  className = '',
}: SwitchProps) {
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    onChange(e.target.checked);
  };

  return (
    <label className={`${styles.switch} ${className}`}>
      <input
        type="checkbox"
        checked={checked}
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

Create `src/app/components/ui/Switch/Switch.module.css`:

```css
.switch {
  position: relative;
  display: inline-block;
  width: 60px;
  height: 30px;
  cursor: pointer;
}

.input {
  opacity: 0;
  width: 0;
  height: 0;
}

.toggle {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  border-radius: 30px;
  transition: 0.2s;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 8px;
}

.toggle:before {
  content: "";
  position: absolute;
  height: 22px;
  width: 22px;
  left: 4px;
  bottom: 4px;
  background-color: white;
  border-radius: 50%;
  transition: 0.2s;
}

.input:checked + .toggle {
  background-color: #007bff;
}

.input:checked + .toggle:before {
  transform: translateX(30px);
}

.iconLight,
.iconDark {
  font-size: 12px;
  z-index: 1;
  transition: opacity 0.2s;
}

.iconLight {
  color: #333;
}

.iconDark {
  color: white;
  opacity: 0;
}

.input:checked + .toggle .iconLight {
  opacity: 0;
}

.input:checked + .toggle .iconDark {
  opacity: 1;
}
```

## Step 5: Extracting Layout Components

### 1. Profile Component

Create `src/app/components/features/Profile/Profile.tsx`:

```typescript
import React from 'react';
import Icon from '../../ui/Icon';
import styles from './Profile.module.css';

interface ProfileProps {
  name: string;
  email: string;
  avatarIcon?: string;
  className?: string;
}

export default function Profile({
  name,
  email,
  avatarIcon = 'otter',
  className = '',
}: ProfileProps) {
  return (
    <div className={`${styles.profile} ${className}`}>
      <div className={styles.avatar}>
        <Icon name={avatarIcon} size="large" className={styles.avatarIcon} />
      </div>
      <div className={styles.content}>
        <p className={styles.name}>{name}</p>
        <p className={styles.email}>{email}</p>
      </div>
    </div>
  );
}
```

Create `src/app/components/features/Profile/Profile.module.css`:

```css
.profile {
  display: flex;
  align-items: center;
  gap: 12px;
}

.avatar {
  flex-shrink: 0;
}

.avatarIcon {
  color: var(--profile-icon-color, #666);
}

.content {
  min-width: 0;
}

.name {
  font-weight: 600;
  margin: 0 0 4px 0;
  font-size: 16px;
  color: var(--text-primary);
}

.email {
  margin: 0;
  font-size: 14px;
  color: var(--text-secondary);
  word-break: break-word;
}
```

### 2. Search Component

Create `src/app/components/features/Search/Search.tsx`:

```typescript
import React, { useState } from 'react';
import Icon from '../../ui/Icon';
import styles from './Search.module.css';

interface SearchProps {
  placeholder?: string;
  onSearch?: (query: string) => void;
  className?: string;
}

export default function Search({
  placeholder = 'Search...',
  onSearch,
  className = '',
}: SearchProps) {
  const [query, setQuery] = useState('');

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;
    setQuery(value);
    onSearch?.(value);
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSearch?.(query);
  };

  return (
    <form className={`${styles.search} ${className}`} onSubmit={handleSubmit}>
      <input
        type="search"
        value={query}
        onChange={handleChange}
        placeholder={placeholder}
        className={styles.input}
      />
      <div className={styles.icon}>
        <Icon name="magnifying-glass" />
      </div>
    </form>
  );
}
```

### 3. Navigation Item Component

Create `src/app/components/features/Navigation/NavigationItem.tsx`:

```typescript
import React from 'react';
import Link from 'next/link';
import Icon from '../../ui/Icon';
import styles from './NavigationItem.module.css';

interface NavigationItemProps {
  href: string;
  icon: string;
  label: string;
  isActive?: boolean;
  className?: string;
}

export default function NavigationItem({
  href,
  icon,
  label,
  isActive = false,
  className = '',
}: NavigationItemProps) {
  const itemClasses = [
    styles.item,
    isActive ? styles.active : '',
    className,
  ].filter(Boolean).join(' ');

  return (
    <li className={itemClasses}>
      <Link href={href} className={styles.link}>
        <div className={styles.iconWrapper}>
          <Icon name={icon} />
        </div>
        <span className={styles.label}>{label}</span>
      </Link>
    </li>
  );
}
```

Create `src/app/components/features/Navigation/NavigationItem.module.css`:

```css
.item {
  list-style: none;
}

.link {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  text-decoration: none;
  color: var(--nav-text-color);
  border-radius: 8px;
  transition: all 0.2s ease;
}

.link:hover {
  background-color: var(--nav-hover-bg);
  color: var(--nav-hover-text);
}

.active .link {
  background-color: var(--nav-active-bg);
  color: var(--nav-active-text);
}

.iconWrapper {
  flex-shrink: 0;
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.label {
  font-weight: 500;
}
```

## Step 6: Creating the Main Navigation Component

Create `src/app/components/layout/Navbar/Navbar.tsx`:

```typescript
'use client';

import React from 'react';
import Profile from '../../features/Profile/Profile';
import Search from '../../features/Search/Search';
import NavigationItem from '../../features/Navigation/NavigationItem';
import Switch from '../../ui/Switch/Switch';
import Icon from '../../ui/Icon/Icon';
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
  theme: 'light' | 'dark';
  onThemeToggle: () => void;
  onSearch?: (query: string) => void;
  className?: string;
}

export default function Navbar({
  user,
  navigationItems,
  currentPath = '/',
  theme,
  onThemeToggle,
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
          <div className={styles.themeIcon}>
            <Icon name="sun" />
          </div>
          <span className={styles.themeLabel}>
            {theme === 'light' ? 'Light mode' : 'Dark mode'}
          </span>
          <Switch
            checked={theme === 'dark'}
            onChange={onThemeToggle}
            className={styles.themeSwitch}
          />
        </div>
      </div>
    </nav>
  );
}
```

Create `src/app/components/layout/Navbar/Navbar.module.css`:

```css
.navbar {
  display: flex;
  flex-direction: column;
  height: 100vh;
  width: 280px;
  background-color: var(--navbar-bg);
  border-right: 1px solid var(--border-color);
  padding: 24px;
  justify-content: space-between;
}

.top {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.profile {
  padding-bottom: 16px;
  border-bottom: 1px solid var(--border-color);
}

.search {
  /* Search component styles are handled in its own module */
}

.navList {
  display: flex;
  flex-direction: column;
  gap: 4px;
  margin: 0;
  padding: 0;
}

.bottom {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.themeControl {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  background-color: var(--theme-control-bg);
  border-radius: 8px;
}

.themeIcon {
  flex-shrink: 0;
  color: var(--theme-icon-color);
}

.themeLabel {
  flex: 1;
  font-size: 14px;
  font-weight: 500;
  color: var(--text-primary);
}

.themeSwitch {
  flex-shrink: 0;
}
```

## Step 7: Creating a Layout Component

Create `src/app/components/layout/Layout/Layout.tsx`:

```typescript
'use client';

import React from 'react';
import Navbar from '../Navbar/Navbar';
import styles from './Layout.module.css';

interface LayoutProps {
  children: React.ReactNode;
  user: {
    name: string;
    email: string;
  };
  currentPath?: string;
  theme: 'light' | 'dark';
  onThemeToggle: () => void;
}

const navigationItems = [
  { href: '/', icon: 'house-crack', label: 'Dashboard' },
  { href: '/revenue', icon: 'chart-column', label: 'Revenue' },
  { href: '/products', icon: 'bell', label: 'Notification' },
  { href: '/analytics', icon: 'chart-pie', label: 'Analytics' },
  { href: '/inventory', icon: 'box', label: 'Inventory' },
];

export default function Layout({
  children,
  user,
  currentPath,
  theme,
  onThemeToggle,
}: LayoutProps) {
  return (
    <div className={styles.layout} data-theme={theme}>
      <Navbar
        user={user}
        navigationItems={navigationItems}
        currentPath={currentPath}
        theme={theme}
        onThemeToggle={onThemeToggle}
      />
      <main className={styles.main}>
        {children}
      </main>
    </div>
  );
}
```

Create `src/app/components/layout/Layout/Layout.module.css`:

```css
.layout {
  display: flex;
  min-height: 100vh;
}

.main {
  flex: 1;
  padding: 24px;
  background-color: var(--main-bg);
  overflow-y: auto;
}
```

## Step 8: CSS Custom Properties for Theming

Create `src/app/styles/themes.css`:

```css
:root {
  /* Light theme variables */
  --navbar-bg: #ffffff;
  --main-bg: #f8f9fa;
  --border-color: #e9ecef;
  --text-primary: #212529;
  --text-secondary: #6c757d;
  --theme-control-bg: #f8f9fa;
  --theme-icon-color: #6c757d;
  --nav-text-color: #495057;
  --nav-hover-bg: #e9ecef;
  --nav-hover-text: #212529;
  --nav-active-bg: #007bff;
  --nav-active-text: #ffffff;
  --profile-icon-color: #6c757d;
}

[data-theme="dark"] {
  /* Dark theme variables */
  --navbar-bg: #1a1a1a;
  --main-bg: #121212;
  --border-color: #333333;
  --text-primary: #ffffff;
  --text-secondary: #b0b0b0;
  --theme-control-bg: #2a2a2a;
  --theme-icon-color: #b0b0b0;
  --nav-text-color: #e0e0e0;
  --nav-hover-bg: #333333;
  --nav-hover-text: #ffffff;
  --nav-active-bg: #0066cc;
  --nav-active-text: #ffffff;
  --profile-icon-color: #b0b0b0;
}
```

## Step 9: Using the Components

Update your `src/app/layout.tsx`:

```typescript
import type { Metadata } from 'next'
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
        {children}
      </body>
    </html>
  )
}
```

Create `src/app/providers/ThemeProvider.tsx`:

```typescript
'use client';

import React, { createContext, useContext, useState, useEffect } from 'react';

type Theme = 'light' | 'dark';

interface ThemeContextType {
  theme: Theme;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light');

  useEffect(() => {
    // Load theme from localStorage on mount
    const savedTheme = localStorage.getItem('theme') as Theme;
    if (savedTheme) {
      setTheme(savedTheme);
    }
  }, []);

  useEffect(() => {
    // Save theme to localStorage and apply to document
    localStorage.setItem('theme', theme);
    document.body.setAttribute('data-theme', theme);
  }, [theme]);

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
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
```

Update `src/app/page.tsx`:

```typescript
'use client';

import React from 'react';
import Layout from './components/layout/Layout/Layout';
import { ThemeProvider, useTheme } from './providers/ThemeProvider';

function HomePage() {
  const { theme, toggleTheme } = useTheme();

  const user = {
    name: 'iva UI',
    email: 'ivagamilec@gmail.com',
  };

  return (
    <Layout
      user={user}
      currentPath="/"
      theme={theme}
      onThemeToggle={toggleTheme}
    >
      <div>
        <h1>Welcome to the Dashboard</h1>
        <p>Your components have been successfully extracted!</p>
      </div>
    </Layout>
  );
}

export default function Page() {
  return (
    <ThemeProvider>
      <HomePage />
    </ThemeProvider>
  );
}
```

## Step 10: Component Testing

Create a simple test for your Button component at `src/app/components/ui/Button/Button.test.tsx`:

```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import Button from './Button';

describe('Button', () => {
  it('renders children correctly', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('calls onClick when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    
    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('applies correct variant class', () => {
    render(<Button variant="secondary">Click me</Button>);
    const button = screen.getByText('Click me');
    expect(button).toHaveClass('secondary');
  });

  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>);
    const button = screen.getByText('Click me');
    expect(button).toBeDisabled();
  });
});
```

## Step 11: Benefits of This Approach

### 1. Maintainability
- Each component has a single responsibility
- Changes to one component don't affect others
- Easy to locate and fix issues

### 2. Reusability
- Use Button component across different pages
- Profile component can be used in multiple layouts
- Navigation items are consistent everywhere

### 3. Testability
- Test components in isolation
- Mock dependencies easily
- Write focused unit tests

### 4. Developer Experience
- TypeScript provides excellent autocomplete
- Clear component APIs through props
- Easy to understand component hierarchy

### 5. Performance
- Code splitting at component level
- Lazy loading opportunities
- Better caching strategies

## Step 12: Best Practices

### 1. Component Naming
- Use PascalCase for component names
- Be descriptive but concise
- Avoid generic names like `Item` or `Component`

### 2. Props Design
- Keep props simple and focused
- Use TypeScript interfaces for complex props
- Provide sensible defaults
- Use optional props judiciously

### 3. File Organization
- One component per file
- Colocate related files
- Use index files for clean imports
- Group by feature, not by file type

### 4. CSS Organization
- Use CSS Modules for component styles
- Avoid global styles in components
- Use CSS custom properties for theming
- Keep styles close to components

### 5. Component Composition
- Prefer composition over inheritance
- Make components configurable through props
- Use children prop for flexible layouts
- Avoid deep prop drilling

## Summary

Component extraction transforms your monolithic HTML into a modular, maintainable React application. By following the collocation principle and organizing components by features, you create a scalable architecture that grows with your application.

## Next Steps

In the next chapter, we'll implement the color mode functionality using React hooks and context, replacing your vanilla JavaScript implementation with a more robust React-based solution.

## Key Takeaways

1. **Think in Components**: Break UI into small, focused pieces
2. **Collocate Files**: Keep related files together
3. **Use TypeScript**: Leverage type safety for better APIs
4. **Organize by Features**: Group by business logic, not file types
5. **Test Components**: Write tests for complex logic
6. **Compose, Don't Inherit**: Use composition for flexibility