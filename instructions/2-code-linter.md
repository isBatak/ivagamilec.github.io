# Chapter 2: ESLint - Code Linting for Next.js

## What is ESLint?

ESLint is a static analysis tool that identifies and reports on patterns found in ECMAScript/JavaScript code. It helps you:

- **Find bugs**: Catch common errors before they reach production
- **Maintain consistency**: Enforce coding standards across your team
- **Improve code quality**: Follow best practices and modern JavaScript patterns
- **Save time**: Automatically fix many issues

ESLint analyzes your code and reports issues like:
- Unused variables
- Missing semicolons
- Unreachable code
- Inconsistent spacing and formatting
- Potential runtime errors

## ESLint in Next.js

Next.js comes with ESLint built-in and pre-configured with sensible defaults. When you created your Next.js project in Chapter 1, ESLint was automatically set up if you chose "Yes" during the setup process.

## Step 1: Verify ESLint Installation

Check if ESLint is already installed by looking at your `package.json`:

```json
{
  "devDependencies": {
    "eslint": "^8",
    "eslint-config-next": "14.0.0"
  }
}
```

If it's not installed, add it:

```bash
npm install --save-dev eslint eslint-config-next
```

## Step 2: ESLint Configuration File

Next.js automatically creates an `.eslintrc.json` file in your project root:

```json
{
  "extends": ["next/core-web-vitals"]
}
```

This configuration includes:
- **React rules**: React-specific linting rules
- **React Hooks rules**: Rules for proper hook usage
- **Next.js rules**: Next.js specific optimizations
- **Core Web Vitals**: Performance-related rules

## Step 3: Understanding the Next.js ESLint Config

The `next/core-web-vitals` configuration includes several rule sets:

### Core Rules Include:
- `react/no-unescaped-entities`: Prevents unescaped HTML entities
- `react/jsx-no-target-blank`: Requires `rel="noopener"` for external links
- `@next/next/no-img-element`: Prefers Next.js Image component over `<img>`
- `@next/next/no-page-custom-font`: Prevents custom fonts in pages

## Step 4: Running ESLint

You can run ESLint in several ways:

### Command Line
```bash
# Lint all files
npm run lint

# Lint specific files
npx eslint src/app/page.tsx

# Auto-fix issues where possible
npm run lint -- --fix
```

### In Your Code Editor
Most editors have ESLint extensions that show errors in real-time:

- **VS Code**: Install the "ESLint" extension
- **WebStorm**: Built-in ESLint support
- **Sublime Text**: SublimeLinter-eslint package

## Step 5: Customizing ESLint Rules

Let's customize the ESLint configuration for your project. Update `.eslintrc.json`:

```json
{
  "extends": [
    "next/core-web-vitals"
  ],
  "rules": {
    // Enforce semicolons
    "semi": ["error", "always"],
    
    // Enforce consistent quotes
    "quotes": ["error", "single"],
    
    // Prevent unused variables
    "no-unused-vars": "error",
    
    // Enforce consistent indentation
    "indent": ["error", 2],
    
    // Require trailing commas in multiline
    "comma-dangle": ["error", "always-multiline"],
    
    // Enforce consistent spacing
    "object-curly-spacing": ["error", "always"],
    
    // Prevent console.log in production
    "no-console": "warn",
    
    // Enforce proper React practices
    "react/prop-types": "off", // We'll use TypeScript instead
    "react/react-in-jsx-scope": "off", // Not needed in Next.js
    
    // Next.js specific rules
    "@next/next/no-img-element": "error"
  },
  "env": {
    "browser": true,
    "es2021": true,
    "node": true
  }
}
```

## Step 6: Rule Severity Levels

ESLint has three severity levels:

- **"off" or 0**: Turn the rule off
- **"warn" or 1**: Show as warning (yellow in editors)
- **"error" or 2**: Show as error (red in editors, fails build)

```json
{
  "rules": {
    "no-console": "warn",        // Warning: shows but doesn't break build
    "no-unused-vars": "error",   // Error: breaks build
    "prefer-const": "off"        // Disabled: rule is ignored
  }
}
```

## Step 7: ESLint with TypeScript

If you're using TypeScript, install the TypeScript ESLint parser:

```bash
npm install --save-dev @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

Update your `.eslintrc.json`:

```json
{
  "extends": [
    "next/core-web-vitals",
    "@typescript-eslint/recommended"
  ],
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint"],
  "rules": {
    // TypeScript specific rules
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/explicit-function-return-type": "off",
    "@typescript-eslint/explicit-module-boundary-types": "off",
    "@typescript-eslint/no-explicit-any": "warn"
  }
}
```

## Step 8: Ignoring Files

Create `.eslintignore` to exclude files from linting:

```
# Dependencies
node_modules/

# Build outputs
.next/
out/

# Environment files
.env*

# Other
*.config.js
```

## Step 9: Integrating ESLint with Your Workflow

### Pre-commit Hooks
Install husky and lint-staged for automatic linting before commits:

```bash
npm install --save-dev husky lint-staged
```

Add to `package.json`:

```json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "git add"
    ]
  },
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  }
}
```

### GitHub Actions
Create `.github/workflows/lint.yml`:

```yaml
name: Lint

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run lint
```

## Step 10: Common ESLint Errors and Fixes

### 1. Unused Variables
```javascript
// ❌ Error: 'unusedVar' is defined but never used
const unusedVar = 'hello';
const usedVar = 'world';
console.log(usedVar);

// ✅ Fix: Remove unused variable
const usedVar = 'world';
console.log(usedVar);
```

### 2. Missing Dependencies in useEffect
```javascript
// ❌ Error: React Hook useEffect has a missing dependency
useEffect(() => {
  fetchData(userId);
}, []);

// ✅ Fix: Add dependency
useEffect(() => {
  fetchData(userId);
}, [userId]);
```

### 3. Incorrect HTML attributes in JSX
```javascript
// ❌ Error: 'class' should be 'className'
<div class="container">

// ✅ Fix: Use className
<div className="container">
```

## Step 11: VS Code Integration

For optimal VS Code integration, create `.vscode/settings.json`:

```json
{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "editor.formatOnSave": true,
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact"
  ]
}
```

## Step 12: Testing Your ESLint Setup

Create a test file `src/app/test-eslint.tsx` with intentional errors:

```tsx
import React from 'react'

// This will trigger several ESLint errors
export default function TestComponent() {
  const unusedVariable = 'test'  // unused-vars error
  const name = "John"           // quotes error (should be single)
  
  console.log('Debug message')  // no-console warning
  
  return (
    <div class="test">           {/* class should be className */}
      <img src="/test.jpg" />    {/* should use Next.js Image */}
      Hello {name}
    </div>
  )
}
```

Run ESLint:
```bash
npm run lint
```

You should see errors. Fix them and run again to verify.

## Benefits of ESLint in Your Project

1. **Consistency**: All team members follow the same coding standards
2. **Error Prevention**: Catch bugs before they reach production
3. **Performance**: Next.js specific rules help optimize your app
4. **Accessibility**: Built-in a11y rules improve user experience
5. **Maintainability**: Cleaner, more readable code

## Next Steps

In the next chapter, we'll set up Prettier for code formatting, which works perfectly alongside ESLint to maintain both code quality and consistent formatting.

## Troubleshooting

**Common Issues:**

1. **ESLint not running**: Make sure the ESLint extension is installed and enabled
2. **Rules not applying**: Check your `.eslintrc.json` syntax
3. **TypeScript errors**: Ensure TypeScript ESLint packages are installed
4. **Performance issues**: Add files to `.eslintignore` if needed

**Useful Commands:**
```bash
npm run lint           # Run ESLint
npm run lint -- --fix  # Auto-fix issues
npx eslint --init      # Reconfigure ESLint
npx eslint --print-config file.js  # Show config for file
```