# Chapter 4: TypeScript Introduction for Next.js

## What is TypeScript?

TypeScript is a strongly typed programming language that builds on JavaScript by adding static type definitions. It's developed by Microsoft and compiles to plain JavaScript that runs anywhere JavaScript runs.

### Key Benefits:
- **Type Safety**: Catch errors at compile time instead of runtime
- **Better IDE Support**: Enhanced autocomplete, refactoring, and navigation
- **Self-Documenting Code**: Types serve as inline documentation
- **Easier Refactoring**: Confidently change code with compiler validation
- **Team Collaboration**: Clear contracts between different parts of your code

### TypeScript vs JavaScript:
```javascript
// JavaScript - No type checking
function greet(name) {
  return "Hello, " + name;
}

greet(123); // Works, but might not be what you intended
```

```typescript
// TypeScript - Type checking
function greet(name: string): string {
  return "Hello, " + name;
}

greet(123); // Error: Argument of type 'number' is not assignable to parameter of type 'string'
```

## Why TypeScript with Next.js?

1. **Built-in Support**: Next.js has excellent TypeScript support out of the box
2. **React Integration**: Perfect for React components with prop validation
3. **API Routes**: Type-safe API endpoints
4. **Development Experience**: Catch bugs before they reach production
5. **Scalability**: Essential for larger applications

## Step 1: Basic TypeScript Concepts

### 1. Primitive Types

```typescript
// Basic types
let name: string = "Iva";
let age: number = 25;
let isActive: boolean = true;
let items: number[] = [1, 2, 3];
let user: string[] = ["John", "Jane"];

// Alternative array syntax
let scores: Array<number> = [95, 87, 92];
```

### 2. Object Types

```typescript
// Object type
let user: {
  name: string;
  age: number;
  email: string;
} = {
  name: "Iva",
  age: 25,
  email: "iva@example.com"
};

// Optional properties
let profile: {
  name: string;
  age?: number; // Optional
  email: string;
} = {
  name: "John",
  email: "john@example.com"
  // age is optional, so it's not required
};
```

### 3. Function Types

```typescript
// Function with typed parameters and return type
function add(a: number, b: number): number {
  return a + b;
}

// Function type as variable
let multiply: (x: number, y: number) => number = (x, y) => x * y;

// Function with optional parameters
function greet(name: string, greeting?: string): string {
  return `${greeting || "Hello"}, ${name}!`;
}

// Function with default parameters
function createUser(name: string, role: string = "user"): object {
  return { name, role };
}
```

## Step 2: Interfaces and Types

### Interfaces
```typescript
// Interface definition
interface User {
  id: number;
  name: string;
  email: string;
  age?: number; // Optional
  readonly createdAt: Date; // Read-only
}

// Using the interface
const user: User = {
  id: 1,
  name: "Iva",
  email: "iva@example.com",
  createdAt: new Date()
};

// Interface for functions
interface SearchFunction {
  (source: string, subString: string): boolean;
}

let mySearch: SearchFunction = function(source: string, subString: string) {
  return source.search(subString) > -1;
};
```

### Type Aliases
```typescript
// Type alias
type UserRole = "admin" | "user" | "moderator"; // Union type
type UserID = string | number;

// Object type alias
type Product = {
  id: UserID;
  name: string;
  price: number;
  category: string;
};

// Function type alias
type EventHandler = (event: Event) => void;
```

## Step 3: Advanced TypeScript Concepts

### 1. Union Types
```typescript
// Union types - can be one of several types
type Status = "loading" | "success" | "error";
type ID = string | number;

function displayStatus(status: Status): string {
  switch (status) {
    case "loading":
      return "Loading...";
    case "success":
      return "Success!";
    case "error":
      return "Error occurred";
    default:
      return "Unknown status";
  }
}
```

### 2. Generics
```typescript
// Generic function
function identity<T>(arg: T): T {
  return arg;
}

let output1 = identity<string>("myString");
let output2 = identity<number>(100);

// Generic interface
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

// Usage
const userResponse: ApiResponse<User> = {
  data: { id: 1, name: "Iva", email: "iva@example.com", createdAt: new Date() },
  status: 200,
  message: "Success"
};
```

### 3. Utility Types
```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

// Partial - makes all properties optional
type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; age?: number; }

// Pick - select specific properties
type UserSummary = Pick<User, "id" | "name">;
// { id: number; name: string; }

// Omit - exclude specific properties
type CreateUser = Omit<User, "id">;
// { name: string; email: string; age: number; }

// Required - makes all properties required
type RequiredUser = Required<PartialUser>;
// { id: number; name: string; email: string; age: number; }
```

## Step 4: TypeScript with React Components

### 1. Function Components

```typescript
// src/app/components/UserCard.tsx
interface UserCardProps {
  user: {
    id: number;
    name: string;
    email: string;
    avatar?: string;
  };
  onEdit?: (id: number) => void;
  className?: string;
}

export default function UserCard({ user, onEdit, className }: UserCardProps) {
  const handleEdit = () => {
    if (onEdit) {
      onEdit(user.id);
    }
  };

  return (
    <div className={`user-card ${className || ''}`}>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      {onEdit && (
        <button onClick={handleEdit}>Edit</button>
      )}
    </div>
  );
}
```

### 2. Props with Complex Types

```typescript
// src/app/components/ProductCard.tsx
interface Product {
  id: number;
  name: string;
  price: number;
  category: string;
  inStock: boolean;
  tags: string[];
}

interface ProductCardProps {
  product: Product;
  onAddToCart?: (productId: number) => void;
  onViewDetails?: (product: Product) => void;
  showActions?: boolean;
  className?: string;
}

export default function ProductCard({ 
  product, 
  onAddToCart, 
  onViewDetails, 
  showActions = true,
  className = ''
}: ProductCardProps) {
  const handleAddToCart = () => {
    if (onAddToCart) {
      onAddToCart(product.id);
    }
  };

  const handleViewDetails = () => {
    if (onViewDetails) {
      onViewDetails(product);
    }
  };

  return (
    <div className={`product-card ${className}`}>
      <h3>{product.name}</h3>
      <p>Price: ${product.price}</p>
      <p>Category: {product.category}</p>
      <p>Status: {product.inStock ? 'In Stock' : 'Out of Stock'}</p>
      <div>
        Tags: {product.tags.join(', ')}
      </div>
      {showActions && (
        <div>
          <button onClick={handleViewDetails}>View Details</button>
          {product.inStock && (
            <button onClick={handleAddToCart}>Add to Cart</button>
          )}
        </div>
      )}
    </div>
  );
}
```

### 3. Simple Event Handlers

```typescript
// src/app/components/LoginForm.tsx
interface LoginFormProps {
  onLogin: (username: string, password: string) => void;
  isLoading?: boolean;
}

export default function LoginForm({ onLogin, isLoading = false }: LoginFormProps) {
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    
    const formData = new FormData(e.currentTarget);
    const username = formData.get('username') as string;
    const password = formData.get('password') as string;
    
    if (username && password) {
      onLogin(username, password);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="username">Username:</label>
        <input
          id="username"
          name="username"
          type="text"
          required
          disabled={isLoading}
        />
      </div>
      
      <div>
        <label htmlFor="password">Password:</label>
        <input
          id="password"
          name="password"
          type="password"
          required
          disabled={isLoading}
        />
      </div>
      
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}
```

## Step 5: TypeScript with Next.js Specific Features

### 1. API Routes

```typescript
// src/app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';

interface User {
  id: number;
  name: string;
  email: string;
}

const users: User[] = [
  { id: 1, name: "John Doe", email: "john@example.com" },
  { id: 2, name: "Jane Smith", email: "jane@example.com" }
];

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const query = searchParams.get('search');

  let filteredUsers = users;
  
  if (query) {
    filteredUsers = users.filter(user => 
      user.name.toLowerCase().includes(query.toLowerCase())
    );
  }

  return NextResponse.json({ users: filteredUsers });
}

export async function POST(request: NextRequest) {
  try {
    const body: Omit<User, 'id'> = await request.json();
    
    const newUser: User = {
      id: users.length + 1,
      ...body
    };
    
    users.push(newUser);
    
    return NextResponse.json({ user: newUser }, { status: 201 });
  } catch (error) {
    return NextResponse.json(
      { error: 'Invalid request body' },
      { status: 400 }
    );
  }
}
```

### 2. Page Components

```typescript
// src/app/users/page.tsx
interface User {
  id: number;
  name: string;
  email: string;
}

async function getUsers(): Promise<User[]> {
  const res = await fetch('http://localhost:3000/api/users', {
    cache: 'no-store'
  });
  
  if (!res.ok) {
    throw new Error('Failed to fetch users');
  }
  
  const data = await res.json();
  return data.users;
}

export default async function UsersPage() {
  const users = await getUsers();

  return (
    <div>
      <h1>Users</h1>
      <ul>
        {users.map((user: User) => (
          <li key={user.id}>
            {user.name} - {user.email}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## Step 6: Practical Exercises

### Exercise 1: Create Basic Component Types

Create type definitions for common UI components:

```typescript
// src/app/types/components.ts
export interface ButtonProps {
  variant: 'primary' | 'secondary' | 'outline';
  size: 'small' | 'medium' | 'large';
  disabled?: boolean;
  onClick?: () => void;
  children: React.ReactNode;
}

export interface InputProps {
  type: 'text' | 'email' | 'password' | 'number';
  value: string;
  onChange: (value: string) => void;
  placeholder?: string;
  required?: boolean;
  disabled?: boolean;
}

export interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title?: string;
  children: React.ReactNode;
}
```

### Exercise 2: Create a Notification System

```typescript
// src/app/types/notifications.ts
export type NotificationType = 'success' | 'error' | 'warning' | 'info';

export interface Notification {
  id: string;
  type: NotificationType;
  title: string;
  message: string;
  timestamp: Date;
  read?: boolean;
}

export interface NotificationProps {
  notification: Notification;
  onMarkAsRead?: (id: string) => void;
  onDismiss?: (id: string) => void;
}

// Usage example
export default function NotificationItem({ 
  notification, 
  onMarkAsRead, 
  onDismiss 
}: NotificationProps) {
  return (
    <div className={`notification notification--${notification.type}`}>
      <h4>{notification.title}</h4>
      <p>{notification.message}</p>
      <small>{notification.timestamp.toLocaleTimeString()}</small>
      {onMarkAsRead && !notification.read && (
        <button onClick={() => onMarkAsRead(notification.id)}>
          Mark as Read
        </button>
      )}
      {onDismiss && (
        <button onClick={() => onDismiss(notification.id)}>
          Dismiss
        </button>
      )}
    </div>
  );
}
```

## Step 7: Practice Exercises

### Exercise 1: Update Your Navbar Component
Convert your existing navbar component to TypeScript with proper types:

**Your Task:**
1. Add proper TypeScript types to your navbar component
2. Create interfaces for navigation items
3. Add event handler types for the theme toggle

### Exercise 2: Create a Product Interface
Based on your products page, create proper TypeScript interfaces:

```typescript
interface Product {
  id: number;
  name: string;
  price: number;
  category: string;
  description?: string;
  inStock: boolean;
  tags: string[];
}

// Create a ProductCard component that uses this interface
// Add functions to filter products by category
// Create a shopping cart with proper types
```

### Exercise 3: Build a Settings Page
Create a settings page with TypeScript:

```typescript
interface UserSettings {
  theme: 'light' | 'dark';
  notifications: {
    email: boolean;
    push: boolean;
    newsletter: boolean;
  };
  language: 'en' | 'es' | 'fr';
  timezone: string;
}

// Create a form to edit these settings
// Add validation with proper error types
// Save settings to localStorage with type safety
```

## Step 8: Common TypeScript Patterns in React

### 1. Conditional Rendering with Types
```typescript
interface LoadingProps {
  isLoading: boolean;
  error?: string;
  children: React.ReactNode;
}

function LoadingWrapper({ isLoading, error, children }: LoadingProps) {
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  return <>{children}</>;
}
```

### 2. Polymorphic Components
```typescript
interface ButtonProps<T extends React.ElementType = 'button'> {
  as?: T;
  children: React.ReactNode;
  variant?: 'primary' | 'secondary';
}

function Button<T extends React.ElementType = 'button'>({
  as,
  children,
  variant = 'primary',
  ...props
}: ButtonProps<T> & Omit<React.ComponentPropsWithoutRef<T>, keyof ButtonProps<T>>) {
  const Component = as || 'button';
  
  return (
    <Component className={`btn btn-${variant}`} {...props}>
      {children}
    </Component>
  );
}

// Usage
<Button>Click me</Button>
<Button as="a" href="/link">Link Button</Button>
```

## Step 9: TypeScript Configuration for Next.js

Your `tsconfig.json` should look like this:

```json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "es6"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

## Step 10: Debugging TypeScript

### Common Error Messages and Solutions:

1. **Property does not exist on type**
```typescript
// Error: Property 'foo' does not exist on type '{}'
const obj = {};
obj.foo = 'bar'; // Error

// Solution: Define the type
const obj: { foo?: string } = {};
obj.foo = 'bar'; // OK
```

2. **Argument of type 'X' is not assignable to parameter of type 'Y'**
```typescript
// Error
function greet(name: string) {
  console.log(`Hello, ${name}`);
}
greet(123); // Error

// Solution: Use correct type
greet("John"); // OK
```

3. **Object is possibly 'undefined'**
```typescript
// Error
function process(user?: User) {
  console.log(user.name); // Error: user might be undefined
}

// Solution: Check for undefined
function process(user?: User) {
  if (user) {
    console.log(user.name); // OK
  }
  // Or use optional chaining
  console.log(user?.name); // OK
}
```

## Benefits of TypeScript in Your Project

1. **Catch Errors Early**: Find bugs at compile time
2. **Better Refactoring**: Rename and restructure with confidence
3. **Enhanced IDE Support**: Better autocomplete and navigation
4. **Self-Documenting**: Types serve as documentation
5. **Team Collaboration**: Clear contracts between code modules

## Next Steps

In the next chapter, we'll introduce React Hooks - powerful functions that let you add state and lifecycle features to functional components. You'll learn about useState, useEffect, and other essential hooks needed for building interactive React applications with proper TypeScript support.

## Useful Resources

- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
- [Next.js TypeScript docs](https://nextjs.org/docs/basic-features/typescript)

## Summary

TypeScript adds powerful type safety to your Next.js application. Start with basic types and gradually adopt more advanced features. The initial learning curve pays off with better code quality, fewer bugs, and improved developer experience.