# Chapter 1: Next.js Setup and Project Migration

## Introduction

This tutorial will guide you through migrating your existing HTML, CSS, and JavaScript project to Next.js. Next.js is a React framework that provides features like server-side rendering, static site generation, and automatic code splitting out of the box.

## Prerequisites

- Node.js 18.17 or later
- Basic knowledge of HTML, CSS, and JavaScript
- Familiarity with command line/terminal

## Step 1: Create a New Next.js Project

First, let's create a new Next.js project. Open your terminal and run:

```bash
npx create-next-app@latest iva-nextjs-app
```

You'll be prompted with several questions. For this migration, choose:

```
✔ Would you like to use TypeScript? … No / Yes  (Choose Yes for better development experience)
✔ Would you like to use ESLint? … No / Yes  (Choose Yes)
✔ Would you like to use Tailwind CSS? … No / Yes  (Choose No, we'll use our existing CSS)
✔ Would you like to use `src/` directory? … No / Yes  (Choose Yes for better organization)
✔ Would you like to use App Router? … No / Yes  (Choose Yes, it's the modern approach)
✔ Would you like to customize the default import alias (@/*)? … No / Yes  (Choose No)
```

## Step 2: Navigate to Your Project

```bash
cd iva-nextjs-app
```

## Step 3: Understand the Project Structure

Your new Next.js project will have this structure:

```
iva-nextjs-app/
├── public/                 # Static files (images, icons, etc.)
├── src/
│   └── app/               # App Router directory
│       ├── globals.css    # Global CSS styles
│       ├── layout.tsx     # Root layout component
│       ├── page.tsx       # Home page component
│       └── favicon.ico    # Favicon
├── next.config.js         # Next.js configuration
├── package.json          # Dependencies and scripts
├── tailwind.config.ts    # Tailwind configuration (if you chose it)
└── tsconfig.json         # TypeScript configuration
```

## Step 4: Clean Up Default Files

Let's remove the default styling and content to prepare for our migration:

1. **Clear the default page content** - Open `src/app/page.tsx`:

```tsx
export default function Home() {
  return (
    <main>
      <h1>Welcome to Iva UI</h1>
    </main>
  );
}
```

2. **Clear global styles** - Open `src/app/globals.css` and remove all content except:

```css
* {
  box-sizing: border-box;
  padding: 0;
  margin: 0;
}

html,
body {
  max-width: 100vw;
  overflow-x: hidden;
}
```

## Step 5: Copy Your Existing Assets

1. **Copy your CSS files** to the `src/app/` directory:
   - Copy `style.css` and `style2.css` to `src/app/`

2. **Copy any images or static assets** to the `public/` directory

## Step 6: Set Up Your Layout

Update `src/app/layout.tsx` to include your CSS and Font Awesome:

```tsx
import type { Metadata } from 'next'
import './globals.css'
import './style.css'
// import './style2.css' // Add if needed

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
      <body data-theme="light">
        {children}
      </body>
    </html>
  )
}
```

## Step 7: Create Your First Component

Now let's convert your navbar HTML to a React component. Create `src/app/components/navbar.tsx`:

```tsx
export default function Navbar() {
  return (
    <nav className="navbar">
      <div className="top">
        <div className="profile">
          <div className="profile__avatar">
            <i className="fa-solid fa-otter profile__avatar-icon"></i>
          </div>

          <div className="profile__content">
            <p className="profile__name">iva UI</p>
            <p className="profile__email">ivagamilec@gmail.com</p>
          </div>
        </div>

        <div className="search">
          <input className="search__input" type="search" placeholder="Search..." />
          <div className="search__icon">
            <i className="fa-solid fa-magnifying-glass"></i>
          </div>
        </div>

        <ul className="nav-list">
          <li className="nav-list__item">
            <a className="nav-list__item-link" href="">
              <div className="nav-list__item-link-icon">
                <i className="fa-solid fa-house-crack"></i>
              </div>
              <span className="nav-list__item-link-label">Dashboard</span>
            </a>
          </li>
          <li className="nav-list__item">
            <a className="nav-list__item-link" href="">
              <div className="nav-list__item-link-icon">
                <i className="fa-solid fa-chart-column"></i>
              </div>
              <span className="nav-list__item-link-label">Revenue</span>
            </a>
          </li>
          <li className="nav-list__item">
            <a className="nav-list__item-link" href="/products">
              <div className="nav-list__item-link-icon">
                <i className="fa-solid fa-lg fa-bell"></i>
              </div>
              <span className="nav-list__item-link-label">Notification</span>
            </a>
          </li>
          <li className="nav-list__item">
            <a className="nav-list__item-link" href="">
              <div className="nav-list__item-link-icon">
                <i className="fa-solid fa-chart-pie"></i>
              </div>
              <span className="nav-list__item-link-label">Analytics</span>
            </a>
          </li>
          <li className="nav-list__item">
            <a className="nav-list__item-link" href="">
              <div className="nav-list__item-link-icon">
                <i className="fa-solid fa-box"></i>
              </div>
              <span className="nav-list__item-link-label">Inventory</span>
            </a>
          </li>
        </ul>
      </div>
      
      <div className="bottom">
        <ul className="nav-list">
          <li className="nav-list__item">
            <a className="nav-list__item-link" href="">
              <div className="nav-list__item-link-icon">
                <i className="fa-solid fa-right-from-bracket"></i>
              </div>
              <span className="nav-list__item-link-label">Logout</span>
            </a>
          </li>
        </ul>
        
        <div className="light-dark-control">
          <div className="light-dark-control__icon">
            <i className="fa-regular fa-sun"></i>
          </div>
          <span className="light-dark-control__label">Light mode</span>
          <label className="light-dark-control__action switch">
            <input className="switch__input" type="checkbox" defaultChecked />
            <div className="switch__toggle">
              <div className="switch__icon switch__icon--light">
                <i className="fa-solid fa-moon"></i>
              </div>
              <div className="switch__icon switch__icon--dark">
                <i className="fa-solid fa-sun"></i>
              </div>
            </div>
          </label>
        </div>
      </div>
    </nav>
  );
}
```

## Step 8: Update Your Main Page

Update `src/app/page.tsx` to include the navbar:

```tsx
import Navbar from './components/navbar'

export default function Home() {
  return (
    <main>
      <Navbar />
      {/* Your main content will go here */}
    </main>
  );
}
```

## Step 9: Run Your Development Server

Start the development server:

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) to see your migrated application.

## Step 10: Create Additional Pages

Create a products page at `src/app/products/page.tsx`:

```tsx
import Navbar from '../components/navbar'

export default function Products() {
  return (
    <main>
      <Navbar />
      <div>
        <h1>Products Page</h1>
        {/* Your products content here */}
      </div>
    </main>
  );
}
```

## Key Differences from HTML to Next.js

1. **File Extensions**: `.html` becomes `.tsx` (TypeScript JSX)
2. **Attributes**: `class` becomes `className`, `for` becomes `htmlFor`
3. **Self-closing tags**: Must be properly closed (e.g., `<input />` instead of `<input>`)
4. **Components**: HTML sections become reusable React components
5. **Routing**: File-based routing instead of separate HTML files
6. **JavaScript Integration**: Logic is integrated within components using hooks

## Next Steps

- In the next chapter, we'll set up ESLint for code quality
- After that, we'll configure Prettier for code formatting
- Then we'll dive into TypeScript basics
- Finally, we'll extract more components and implement the color mode functionality

## Troubleshooting

**Common Issues:**

1. **CSS not loading**: Make sure CSS files are imported in the correct order in `layout.tsx`
2. **Font Awesome not working**: Ensure the CDN link is in the `<head>` section
3. **Build errors**: Check for unclosed JSX tags and proper attribute naming

**Useful Commands:**

```bash
npm run dev          # Start development server
npm run build        # Build for production
npm run start        # Start production server
npm run lint         # Run ESLint
```

This completes the basic Next.js setup and initial migration. Your project should now be running as a Next.js application with the same visual appearance as your original HTML project.
