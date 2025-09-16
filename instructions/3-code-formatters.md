# Chapter 3: Prettier - Code Formatting for Next.js

## What is Prettier?

Prettier is an opinionated code formatter that automatically formats your code to ensure consistent style across your entire project. Unlike ESLint which focuses on code quality and potential bugs, Prettier focuses purely on code formatting and style.

### Key Benefits:
- **Consistency**: Eliminates style debates in your team
- **Automatic**: Formats code on save or during build
- **Language Support**: Works with JavaScript, TypeScript, CSS, HTML, JSON, and more
- **Integration**: Works seamlessly with ESLint and VS Code

### What Prettier Handles:
- Indentation and spacing
- Line length and wrapping
- Quote style (single vs double)
- Semicolons
- Trailing commas
- Bracket spacing

## Why Use Prettier with Next.js?

1. **Team Consistency**: Everyone's code looks the same
2. **Focus on Logic**: Stop worrying about formatting
3. **Reduced PR Reviews**: No more formatting discussions
4. **Better Readability**: Consistent, clean code
5. **Integration**: Works with existing ESLint setup

## Step 1: Install Prettier

Install Prettier as a development dependency:

```bash
npm install --save-dev prettier
```

For better ESLint integration, also install:

```bash
npm install --save-dev eslint-config-prettier eslint-plugin-prettier
```

## Step 2: Create Prettier Configuration

Create `.prettierrc.json` in your project root:

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "bracketSameLine": false,
  "arrowParens": "avoid"
}
```

### Configuration Options Explained:

- **semi**: Add semicolons at the end of statements
- **trailingComma**: Add trailing commas where valid in ES5 (objects, arrays, etc.)
- **singleQuote**: Use single quotes instead of double quotes
- **printWidth**: Maximum line length before wrapping
- **tabWidth**: Number of spaces per indentation level
- **useTabs**: Use tabs instead of spaces for indentation
- **bracketSpacing**: Print spaces between brackets in object literals
- **bracketSameLine**: Put the `>` of a multi-line JSX element at the end of the last line
- **arrowParens**: Avoid parentheses around arrow function parameters when possible

## Step 3: Create Prettier Ignore File

Create `.prettierignore` to exclude files from formatting:

```
# Dependencies
node_modules/

# Build outputs
.next/
out/
build/

# Environment files
.env*

# Package files
package-lock.json
yarn.lock

# Other
*.min.js
*.min.css
coverage/
```

## Step 4: Integrate Prettier with ESLint

Update your `.eslintrc.json` to work with Prettier:

```json
{
  "extends": [
    "next/core-web-vitals",
    "prettier"
  ],
  "plugins": ["prettier"],
  "rules": {
    "prettier/prettier": "error",
    "semi": ["error", "always"],
    "quotes": ["error", "single"],
    "no-unused-vars": "error",
    "comma-dangle": ["error", "es5"],
    "object-curly-spacing": ["error", "always"],
    "no-console": "warn"
  }
}
```

The key changes:
- Added `"prettier"` to extends (must be last)
- Added `"prettier"` to plugins
- Added `"prettier/prettier": "error"` rule

## Step 5: Add NPM Scripts

Add formatting scripts to your `package.json`:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "lint:fix": "next lint --fix",
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "format:staged": "prettier --write"
  }
}
```

### Script Explanations:
- **format**: Format all files in the project
- **format:check**: Check if files are formatted (useful for CI)
- **format:staged**: Format only staged files (for pre-commit hooks)
- **lint:fix**: Run ESLint with auto-fix

## Step 6: VS Code Integration

Create or update `.vscode/settings.json`:

```json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": true
  },
  "editor.rulers": [80],
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[javascriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[css]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

Make sure to install the Prettier VS Code extension: `esbenp.prettier-vscode`

## Step 7: Format Your Existing Code

Format all your existing code:

```bash
npm run format
```

This will reformat all files according to your Prettier configuration.

## Step 8: Pre-commit Hooks with Husky and lint-staged

Set up automatic formatting before commits:

```bash
npm install --save-dev husky lint-staged
```

Initialize husky:

```bash
npx husky install
```

Add a pre-commit hook:

```bash
npx husky add .husky/pre-commit "npx lint-staged"
```

Update `package.json` to include lint-staged configuration:

```json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,css,md}": [
      "prettier --write"
    ]
  }
}
```

Add the prepare script to `package.json`:

```json
{
  "scripts": {
    "prepare": "husky install"
  }
}
```

## Step 9: Example Before and After

### Before Prettier:
```javascript
import React,{useState,useEffect}from 'react';
import{Navbar}from'./components/navbar'

export default function Home(){
const[theme,setTheme]=useState('light')
const[isLoading,setIsLoading]=useState(true)

useEffect(()=>{
if(typeof window!=='undefined'){
const savedTheme=localStorage.getItem('theme')||'light'
setTheme(savedTheme)
setIsLoading(false)
}
},[])

return(<main>
<Navbar theme={theme}/>
<div className="content">
<h1>Welcome to Iva UI</h1>
{isLoading?<p>Loading...</p>:<p>Ready!</p>}
</div>
</main>)
}
```

### After Prettier:
```javascript
import React, { useState, useEffect } from 'react';
import { Navbar } from './components/navbar';

export default function Home() {
  const [theme, setTheme] = useState('light');
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    if (typeof window !== 'undefined') {
      const savedTheme = localStorage.getItem('theme') || 'light';
      setTheme(savedTheme);
      setIsLoading(false);
    }
  }, []);

  return (
    <main>
      <Navbar theme={theme} />
      <div className="content">
        <h1>Welcome to Iva UI</h1>
        {isLoading ? <p>Loading...</p> : <p>Ready!</p>}
      </div>
    </main>
  );
}
```

## Step 10: Prettier with Different File Types

### JavaScript/TypeScript Files
Prettier handles:
- Import/export formatting
- Function parameter alignment
- Object and array formatting
- JSX formatting

### CSS Files
Prettier formats:
- Property ordering
- Selector formatting
- Value spacing
- Media query formatting

### JSON Files
Prettier formats:
- Key ordering (alphabetical)
- Consistent indentation
- Trailing comma removal

### Markdown Files
Prettier formats:
- Table alignment
- Link formatting
- Code block formatting

## Step 11: Team Workflow

### For New Team Members:
1. Clone the repository
2. Run `npm install`
3. Install VS Code extensions: ESLint and Prettier
4. Code will automatically format on save

### For Code Reviews:
1. Formatting is automatic, focus on logic
2. No more "fix spacing" comments
3. Consistent diffs in git

### For CI/CD:
Add to your GitHub Actions workflow:

```yaml
name: Code Quality

on: [push, pull_request]

jobs:
  lint-and-format:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run lint
      - run: npm run format:check
```

## Step 12: Common Configuration Examples

### For Teams That Prefer Tabs:
```json
{
  "useTabs": true,
  "tabWidth": 4
}
```

### For Longer Line Lengths:
```json
{
  "printWidth": 120
}
```

### For No Semicolons:
```json
{
  "semi": false
}
```

### For Different Quote Styles:
```json
{
  "singleQuote": false,
  "jsxSingleQuote": true
}
```

## Step 13: Troubleshooting

### Common Issues:

**1. Prettier and ESLint Conflicts**
- Make sure `eslint-config-prettier` is installed
- Ensure `"prettier"` is the last item in `extends` array

**2. Format on Save Not Working**
- Check VS Code Prettier extension is installed
- Verify `editor.formatOnSave` is `true`
- Check file is not in `.prettierignore`

**3. Different Formatting in Different Environments**
- Ensure same Prettier version across team
- Use exact versions in `package.json`
- Check for different `.prettierrc` files

## Step 14: Test Your Setup

Create a test file `src/app/test-prettier.tsx`:

```typescript
// This code is intentionally poorly formatted
import React,{useState}from 'react'
import{type}from './types'

export default function TestComponent(){
const[count,setCount]=useState(0)
const handleClick=()=>{setCount(count+1)}
return(<div className="test"><button onClick={handleClick}>Count: {count}</button></div>)
}
```

Save the file (if VS Code is configured correctly, it should auto-format), or run:

```bash
npm run format
```

The file should be reformatted to:

```typescript
import React, { useState } from 'react';
import { type } from './types';

export default function TestComponent() {
  const [count, setCount] = useState(0);
  const handleClick = () => {
    setCount(count + 1);
  };
  
  return (
    <div className="test">
      <button onClick={handleClick}>Count: {count}</button>
    </div>
  );
}
```

## Benefits Summary

1. **Consistency**: Code looks the same regardless of who wrote it
2. **Productivity**: No time wasted on formatting decisions
3. **Collaboration**: Reduced friction in code reviews
4. **Quality**: Focus on logic instead of style
5. **Integration**: Works seamlessly with existing tools

## Next Steps

In the next chapter, we'll introduce TypeScript, which will provide type safety and better development experience for your Next.js project. TypeScript works great with both ESLint and Prettier!

## Useful Commands Reference

```bash
# Format all files
npm run format

# Check formatting without changing files
npm run format:check

# Format specific files
npx prettier --write src/app/page.tsx

# Format and lint
npm run lint:fix && npm run format

# Check what files would be formatted
npx prettier --list-different .
```