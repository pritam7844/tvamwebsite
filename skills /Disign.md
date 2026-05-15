# React Design Rules — Deep Guide
>
> Responsive · CSS Libraries · Reusable Architecture · Common Patterns

---

## Table of Contents

1. [Project Structure](#1-project-structure)
2. [Component Design Rules](#2-component-design-rules)
3. [Reusability Principles](#3-reusability-principles)
4. [Responsive Design Rules](#4-responsive-design-rules)
5. [CSS Library Usage Rules](#5-css-library-usage-rules)
6. [Theming & Design Tokens](#6-theming--design-tokens)
7. [Props & API Design](#7-props--api-design)
8. [Common Reusable Components](#8-common-reusable-components)
9. [State Management Rules](#9-state-management-rules)
10. [Performance Rules](#10-performance-rules)
11. [Accessibility Rules](#11-accessibility-rules)
12. [Naming Conventions](#12-naming-conventions)
13. [Anti-Patterns to Avoid](#13-anti-patterns-to-avoid)

---

## 1. Project Structure

Always separate concerns into clear folders. A flat structure becomes unmanageable after 20+ files.

```
src/
├── components/          # Reusable UI components (dumb/presentational)
│   ├── ui/              # Atomic elements: Button, Input, Card, Badge
│   ├── layout/          # Grid, Container, Stack, Flex, Divider
│   └── common/          # Composed: Navbar, Footer, Sidebar, Modal
│
├── features/            # Feature-scoped components + logic
│   ├── auth/
│   │   ├── LoginForm.jsx
│   │   ├── useAuth.js
│   │   └── authSlice.js
│   └── dashboard/
│
├── hooks/               # Custom reusable hooks
├── context/             # React Context providers
├── utils/               # Pure helper functions
├── constants/           # App-wide constants (breakpoints, colors, etc.)
├── styles/              # Global CSS, tokens, reset
│   ├── tokens.css       # Design tokens (CSS variables)
│   ├── reset.css
│   └── global.css
├── pages/               # Route-level components (Next.js / React Router)
└── assets/              # Images, icons, fonts
```

**Rules:**

- `components/ui/` — zero business logic, pure presentational
- `features/` — everything domain-specific lives here
- Never import from `features/` into `components/` (one-way dependency)
- Each folder should have an `index.js` barrel export

---

## 2. Component Design Rules

### 2.1 Single Responsibility

Every component does ONE thing. If you're writing "and" in its description, split it.

```jsx
// ❌ BAD — doing too much
function UserDashboard() {
  const [users, setUsers] = useState([]);
  // fetches data + formats data + renders table + handles pagination
}

// ✅ GOOD — each piece separated
function UserDashboard() {
  return (
    <PageLayout>
      <UserTable />
      <Pagination />
    </PageLayout>
  );
}
```

### 2.2 Component Size Limit

- **Presentational components** → under 80 lines
- **Container/feature components** → under 150 lines
- If longer → extract into sub-components or custom hooks

### 2.3 Composition Over Configuration

Prefer composing small pieces over passing dozens of props to one mega-component.

```jsx
// ❌ BAD — prop soup
<Card
  title="Hello"
  subtitle="World"
  image="/img.png"
  footer="Footer text"
  showBorder
  showShadow
  headerColor="blue"
/>

// ✅ GOOD — composable
<Card>
  <Card.Header>
    <Card.Title>Hello</Card.Title>
    <Card.Subtitle>World</Card.Subtitle>
  </Card.Header>
  <Card.Image src="/img.png" />
  <Card.Footer>Footer text</Card.Footer>
</Card>
```

### 2.4 Controlled vs Uncontrolled

- Form inputs — prefer **controlled** (value + onChange)
- For simple local UI (toggle, dropdown) — **uncontrolled** with `useRef` is fine
- Always expose both modes in reusable inputs via `defaultValue` + `value` support

---

## 3. Reusability Principles

### 3.1 The 3-Times Rule

Write specific code once. Write it twice — note the pattern. On the third time — extract it into a reusable component or hook.

### 3.2 Separation of Logic and UI

```jsx
// ✅ Hook handles logic
function useCounter(initial = 0) {
  const [count, setCount] = useState(initial);
  const increment = () => setCount(c => c + 1);
  const decrement = () => setCount(c => c - 1);
  const reset = () => setCount(initial);
  return { count, increment, decrement, reset };
}

// ✅ Component handles UI only
function Counter() {
  const { count, increment, decrement } = useCounter(0);
  return (
    <div>
      <button onClick={decrement}>-</button>
      <span>{count}</span>
      <button onClick={increment}>+</button>
    </div>
  );
}
```

### 3.3 Render Props & Children as Functions

Use for maximum flexibility in reusable components.

```jsx
// Reusable data-fetcher
function DataFetcher({ url, children }) {
  const { data, loading, error } = useFetch(url);
  return children({ data, loading, error });
}

// Usage
<DataFetcher url="/api/users">
  {({ data, loading }) => loading ? <Spinner /> : <UserList users={data} />}
</DataFetcher>
```

### 3.4 Compound Components Pattern

For related components that share implicit state (Tabs, Accordion, Dropdown):

```jsx
const TabContext = createContext();

function Tabs({ children, defaultTab }) {
  const [active, setActive] = useState(defaultTab);
  return (
    <TabContext.Provider value={{ active, setActive }}>
      <div className="tabs">{children}</div>
    </TabContext.Provider>
  );
}

Tabs.List = function TabList({ children }) {
  return <div className="tabs__list">{children}</div>;
};

Tabs.Tab = function Tab({ id, children }) {
  const { active, setActive } = useContext(TabContext);
  return (
    <button
      className={`tab ${active === id ? 'tab--active' : ''}`}
      onClick={() => setActive(id)}
    >
      {children}
    </button>
  );
};

Tabs.Panel = function TabPanel({ id, children }) {
  const { active } = useContext(TabContext);
  return active === id ? <div className="tab-panel">{children}</div> : null;
};
```

---

## 4. Responsive Design Rules

### 4.1 Mobile-First Always

Write styles for mobile, then scale up with `min-width` media queries. Never write desktop-first and override downward.

```css
/* ✅ CORRECT — mobile first */
.card {
  padding: 1rem;
  flex-direction: column;
}

@media (min-width: 768px) {
  .card {
    padding: 2rem;
    flex-direction: row;
  }
}

/* ❌ WRONG — desktop first */
.card {
  padding: 2rem;
  flex-direction: row;
}

@media (max-width: 767px) {
  .card {
    padding: 1rem;
    flex-direction: column;
  }
}
```

### 4.2 Standard Breakpoints

Define breakpoints as constants — never hardcode them inline.

```js
// constants/breakpoints.js
export const BREAKPOINTS = {
  xs: '480px',
  sm: '640px',
  md: '768px',
  lg: '1024px',
  xl: '1280px',
  '2xl': '1536px',
};
```

```css
/* styles/tokens.css */
:root {
  --bp-xs: 480px;
  --bp-sm: 640px;
  --bp-md: 768px;
  --bp-lg: 1024px;
  --bp-xl: 1280px;
}
```

### 4.3 useBreakpoint Hook

```js
function useBreakpoint() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handler = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handler);
    return () => window.removeEventListener('resize', handler);
  }, []);

  return {
    isMobile: width < 768,
    isTablet: width >= 768 && width < 1024,
    isDesktop: width >= 1024,
    width,
  };
}
```

### 4.4 Fluid Typography

Use `clamp()` for font sizes that scale smoothly between breakpoints:

```css
:root {
  --text-sm:   clamp(0.75rem,  1vw, 0.875rem);
  --text-base: clamp(0.875rem, 1.5vw, 1rem);
  --text-lg:   clamp(1rem,     2vw, 1.25rem);
  --text-xl:   clamp(1.25rem,  3vw, 2rem);
  --text-2xl:  clamp(1.5rem,   4vw, 3rem);
}
```

### 4.5 Responsive Layout Components

```jsx
// Reusable Stack component (vertical/horizontal with gap)
function Stack({ direction = 'column', gap = '1rem', children, className }) {
  return (
    <div
      style={{
        display: 'flex',
        flexDirection: direction,
        gap,
      }}
      className={className}
    >
      {children}
    </div>
  );
}

// Reusable Grid
function Grid({ cols = { base: 1, md: 2, lg: 3 }, gap = '1.5rem', children }) {
  return (
    <div
      className="grid"
      style={{ '--cols-base': cols.base, '--cols-md': cols.md, '--cols-lg': cols.lg, gap }}
    >
      {children}
    </div>
  );
}
```

```css
.grid {
  display: grid;
  grid-template-columns: repeat(var(--cols-base, 1), 1fr);
}

@media (min-width: 768px) {
  .grid { grid-template-columns: repeat(var(--cols-md, 2), 1fr); }
}

@media (min-width: 1024px) {
  .grid { grid-template-columns: repeat(var(--cols-lg, 3), 1fr); }
}
```

---

## 5. CSS Library Usage Rules

### 5.1 Tailwind CSS

**Rules when using Tailwind:**

```jsx
// ✅ Extract repeated class combos into a variable or component
const buttonBase = 'rounded-md font-semibold transition-colors duration-200 focus:outline-none focus:ring-2';
const variants = {
  primary: 'bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-400',
  secondary: 'bg-gray-100 text-gray-800 hover:bg-gray-200 focus:ring-gray-300',
  danger: 'bg-red-600 text-white hover:bg-red-700 focus:ring-red-400',
};

function Button({ variant = 'primary', size = 'md', children, ...props }) {
  const sizes = { sm: 'px-3 py-1.5 text-sm', md: 'px-4 py-2 text-base', lg: 'px-6 py-3 text-lg' };
  return (
    <button className={`${buttonBase} ${variants[variant]} ${sizes[size]}`} {...props}>
      {children}
    </button>
  );
}
```

- Use `clsx` or `cn()` (shadcn helper) to conditionally join classes — never string concatenation with ternaries inside JSX
- Never use arbitrary values (`w-[347px]`) when a standard token works
- Create a `cn()` utility:

```js
// utils/cn.js
import { clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs) {
  return twMerge(clsx(inputs));
}
```

### 5.2 shadcn/ui

- Use shadcn components as the **base layer** — copy into your project, don't treat as a black box
- Always extend via `className` prop, never override with `!important`
- Customize through CSS variables in `globals.css`, not by forking components

```css
/* Theming shadcn via variables */
:root {
  --primary: 221 83% 53%;
  --primary-foreground: 210 40% 98%;
  --radius: 0.5rem;
}
```

### 5.3 CSS Modules

Use CSS Modules for component-scoped styles that need more power than Tailwind provides.

```jsx
// Button.module.css
.button { ... }
.button--primary { ... }
.button--loading { opacity: 0.7; pointer-events: none; }

// Button.jsx
import styles from './Button.module.css';
import { cn } from '@/utils/cn';

function Button({ loading, variant, className }) {
  return (
    <button className={cn(
      styles.button,
      styles[`button--${variant}`],
      loading && styles['button--loading'],
      className
    )}>
      ...
    </button>
  );
}
```

### 5.4 Styled Components / Emotion

- Define styled components **outside** the render function — never inside
- Use theme via `ThemeProvider`, never hardcode color values
- Extract variants using `.attrs()` or prop maps

```jsx
// ❌ BAD — defined inside render = recreated every render
function MyComp() {
  const Box = styled.div`color: red`; // new component each render!
  return <Box />;
}

// ✅ GOOD — defined at module level
const Box = styled.div`
  color: ${({ theme }) => theme.colors.primary};
  padding: ${({ size }) => size === 'lg' ? '2rem' : '1rem'};
`;
```

---

## 6. Theming & Design Tokens

### 6.1 CSS Custom Properties (Tokens)

All design values must live in CSS variables — never hardcoded in components.

```css
/* styles/tokens.css */
:root {
  /* Colors */
  --color-primary-50:  #eff6ff;
  --color-primary-500: #3b82f6;
  --color-primary-900: #1e3a8a;

  --color-neutral-0:   #ffffff;
  --color-neutral-100: #f5f5f5;
  --color-neutral-900: #111111;

  /* Semantic aliases */
  --color-bg:          var(--color-neutral-0);
  --color-text:        var(--color-neutral-900);
  --color-accent:      var(--color-primary-500);
  --color-border:      var(--color-neutral-100);

  /* Spacing scale */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-6: 1.5rem;
  --space-8: 2rem;
  --space-12: 3rem;
  --space-16: 4rem;

  /* Typography */
  --font-sans: 'Inter Variable', sans-serif;
  --font-mono: 'JetBrains Mono', monospace;

  /* Radius */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-full: 9999px;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
  --shadow-md: 0 4px 12px rgba(0,0,0,0.08);
  --shadow-lg: 0 8px 24px rgba(0,0,0,0.12);

  /* Transitions */
  --transition-fast: 150ms ease;
  --transition-base: 250ms ease;
  --transition-slow: 400ms ease;

  /* Z-index scale */
  --z-below:   -1;
  --z-base:     0;
  --z-dropdown: 100;
  --z-sticky:   200;
  --z-overlay:  300;
  --z-modal:    400;
  --z-toast:    500;
}

/* Dark mode */
[data-theme="dark"] {
  --color-bg:   var(--color-neutral-900);
  --color-text: var(--color-neutral-0);
  --color-border: #2a2a2a;
}
```

### 6.2 JS Theme Object

Mirror tokens in JS for use in styled-components or dynamic styles:

```js
// styles/theme.js
export const theme = {
  colors: {
    primary: 'var(--color-accent)',
    bg: 'var(--color-bg)',
    text: 'var(--color-text)',
  },
  space: [0, '0.25rem', '0.5rem', '0.75rem', '1rem', '1.5rem', '2rem', '3rem', '4rem'],
  radii: {
    sm: 'var(--radius-sm)',
    md: 'var(--radius-md)',
    lg: 'var(--radius-lg)',
    full: 'var(--radius-full)',
  },
};
```

---

## 7. Props & API Design

### 7.1 Prop Typing Rules

Always use PropTypes or TypeScript. Every reusable component must document its API.

```tsx
// TypeScript (preferred)
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'ghost' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
}
```

### 7.2 Spread Native Props

Always spread native HTML attributes so consumers aren't blocked:

```jsx
function Input({ label, error, className, ...props }) {
  return (
    <div className="field">
      {label && <label>{label}</label>}
      <input className={cn('input', error && 'input--error', className)} {...props} />
      {error && <span className="error-msg">{error}</span>}
    </div>
  );
}
```

### 7.3 Default Props Strategy

```jsx
// Use default parameters, not defaultProps (deprecated in React 19)
function Card({ variant = 'default', padding = 'md', shadow = true, children }) { ... }
```

### 7.4 Boolean Props Convention

```jsx
// ✅ Boolean props — just the name = true
<Button disabled />
<Button loading />
<Input required />

// ❌ Don't do this
<Button disabled={true} />
```

---

## 8. Common Reusable Components

Every project needs these. Build once, use everywhere.

### 8.1 Button

```jsx
function Button({
  children,
  variant = 'primary',
  size = 'md',
  loading = false,
  leftIcon,
  rightIcon,
  className,
  ...props
}) {
  return (
    <button
      className={cn('btn', `btn--${variant}`, `btn--${size}`, loading && 'btn--loading', className)}
      disabled={loading || props.disabled}
      {...props}
    >
      {loading && <Spinner size="sm" />}
      {!loading && leftIcon && <span className="btn__icon btn__icon--left">{leftIcon}</span>}
      <span className="btn__label">{children}</span>
      {rightIcon && <span className="btn__icon btn__icon--right">{rightIcon}</span>}
    </button>
  );
}
```

### 8.2 Container / Layout Wrapper

```jsx
function Container({ children, maxWidth = 'lg', className }) {
  const widths = {
    sm: '640px', md: '768px', lg: '1024px', xl: '1280px', full: '100%'
  };
  return (
    <div
      style={{ maxWidth: widths[maxWidth], margin: '0 auto', padding: '0 1rem' }}
      className={className}
    >
      {children}
    </div>
  );
}
```

### 8.3 Text / Typography

```jsx
const tags = { h1: 'h1', h2: 'h2', h3: 'h3', h4: 'h4', body: 'p', small: 'span', label: 'label' };

function Text({ variant = 'body', as, color, weight, align, truncate, children, className }) {
  const Tag = as || tags[variant] || 'p';
  return (
    <Tag
      className={cn(`text--${variant}`, truncate && 'text--truncate', className)}
      style={{ color, fontWeight: weight, textAlign: align }}
    >
      {children}
    </Tag>
  );
}
```

### 8.4 Spinner / Loader

```jsx
function Spinner({ size = 'md', color = 'currentColor' }) {
  const sizes = { sm: '1rem', md: '1.5rem', lg: '2.5rem' };
  return (
    <svg
      width={sizes[size]} height={sizes[size]}
      viewBox="0 0 24 24" fill="none"
      className="spinner"
      aria-label="Loading"
      role="status"
    >
      <circle cx="12" cy="12" r="10" stroke={color} strokeWidth="3" strokeDasharray="40 60" />
    </svg>
  );
}
```

```css
.spinner { animation: spin 0.8s linear infinite; }
@keyframes spin { to { transform: rotate(360deg); } }
```

### 8.5 Modal

```jsx
function Modal({ isOpen, onClose, title, children, size = 'md' }) {
  useEffect(() => {
    const handler = (e) => e.key === 'Escape' && onClose();
    if (isOpen) document.addEventListener('keydown', handler);
    return () => document.removeEventListener('keydown', handler);
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return createPortal(
    <div className="modal-overlay" onClick={onClose} role="dialog" aria-modal="true">
      <div className={`modal modal--${size}`} onClick={e => e.stopPropagation()}>
        <div className="modal__header">
          <h2 className="modal__title">{title}</h2>
          <button className="modal__close" onClick={onClose} aria-label="Close">✕</button>
        </div>
        <div className="modal__body">{children}</div>
      </div>
    </div>,
    document.body
  );
}
```

### 8.6 Toast / Notification System

```jsx
// hooks/useToast.js
const ToastContext = createContext();

export function ToastProvider({ children }) {
  const [toasts, setToasts] = useState([]);

  const addToast = useCallback(({ message, type = 'info', duration = 3000 }) => {
    const id = Date.now();
    setToasts(prev => [...prev, { id, message, type }]);
    setTimeout(() => setToasts(prev => prev.filter(t => t.id !== id)), duration);
  }, []);

  return (
    <ToastContext.Provider value={{ addToast }}>
      {children}
      <div className="toast-container">
        {toasts.map(t => (
          <div key={t.id} className={`toast toast--${t.type}`}>{t.message}</div>
        ))}
      </div>
    </ToastContext.Provider>
  );
}

export const useToast = () => useContext(ToastContext);
```

### 8.7 Error Boundary

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, info) {
    console.error('ErrorBoundary caught:', error, info);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          <button onClick={() => this.setState({ hasError: false })}>Try again</button>
        </div>
      );
    }
    return this.props.children;
  }
}
```

### 8.8 Common Custom Hooks

```js
// hooks/useFetch.js
function useFetch(url, options) {
  const [state, setState] = useState({ data: null, loading: true, error: null });

  useEffect(() => {
    let cancelled = false;
    setState({ data: null, loading: true, error: null });

    fetch(url, options)
      .then(r => r.json())
      .then(data => !cancelled && setState({ data, loading: false, error: null }))
      .catch(error => !cancelled && setState({ data: null, loading: false, error }));

    return () => { cancelled = true; };
  }, [url]);

  return state;
}

// hooks/useLocalStorage.js
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    try { return JSON.parse(localStorage.getItem(key)) ?? initialValue; }
    catch { return initialValue; }
  });

  const set = useCallback((val) => {
    setValue(val);
    localStorage.setItem(key, JSON.stringify(val));
  }, [key]);

  return [value, set];
}

// hooks/useDebounce.js
function useDebounce(value, delay = 300) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  return debounced;
}

// hooks/useClickOutside.js
function useClickOutside(ref, handler) {
  useEffect(() => {
    const listener = (e) => {
      if (!ref.current || ref.current.contains(e.target)) return;
      handler(e);
    };
    document.addEventListener('mousedown', listener);
    return () => document.removeEventListener('mousedown', listener);
  }, [ref, handler]);
}
```

---

## 9. State Management Rules

### 9.1 State Placement Rules

| State Type | Where to put it |
|---|---|
| UI state (open/closed, active tab) | Local `useState` |
| Shared between siblings | Lift to parent |
| Shared across a feature | React Context or Zustand slice |
| Global app state (user, theme) | Global Context or Zustand/Redux |
| Server state (API data) | React Query / SWR |
| URL state (filters, pagination) | URL params via `useSearchParams` |

### 9.2 Context Rules

```jsx
// ✅ Split contexts — don't put unrelated state together
// Separate UserContext, ThemeContext, CartContext

// ✅ Always split value and updater if value changes often
const ThemeValueContext = createContext();
const ThemeSetterContext = createContext();

// Consumers that only SET don't re-render when value changes
function ThemeToggle() {
  const setTheme = useContext(ThemeSetterContext); // won't re-render on value change
  return <button onClick={() => setTheme('dark')}>Dark</button>;
}
```

### 9.3 Server State (React Query)

```jsx
// Always use React Query for async/server data
function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then(r => r.json()),
    staleTime: 1000 * 60 * 5, // 5 minutes
  });
}

function useMutation_CreateUser() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data) => fetch('/api/users', { method: 'POST', body: JSON.stringify(data) }),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['users'] }),
  });
}
```

---

## 10. Performance Rules

### 10.1 Memoization — When to Use

```jsx
// useMemo — expensive computations only
const sorted = useMemo(() => heavySort(items), [items]);

// useCallback — stable refs for child components using React.memo
const handleClick = useCallback((id) => deleteItem(id), [deleteItem]);

// React.memo — only if parent re-renders frequently AND props rarely change
const UserCard = React.memo(function UserCard({ user }) { ... });

// ❌ Don't over-memoize simple values — it costs more than it saves
const doubled = useMemo(() => count * 2, [count]); // useless
```

### 10.2 Code Splitting

```jsx
// Split routes
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

### 10.3 List Rendering

```jsx
// ✅ Always use stable, unique keys — never index for dynamic lists
{users.map(user => <UserCard key={user.id} user={user} />)}

// ❌ Never use index as key for lists that reorder/filter
{users.map((user, i) => <UserCard key={i} user={user} />)}

// For very long lists — use virtualization
import { FixedSizeList } from 'react-window';
```

### 10.4 Image Optimization

```jsx
// Always specify dimensions to prevent layout shift
<img
  src={src}
  alt={alt}
  width={400}
  height={300}
  loading="lazy"
  decoding="async"
/>
```

---

## 11. Accessibility Rules

Every component must be accessible. Non-negotiable.

```jsx
// ✅ Semantic HTML first
<button onClick={handleClick}>Click me</button>  // not <div onClick>

// ✅ Keyboard navigation
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => (e.key === 'Enter' || e.key === ' ') && handleClick()}
>

// ✅ ARIA labels where text isn't obvious
<button aria-label="Close dialog">✕</button>
<input aria-label="Search users" type="search" />

// ✅ Live regions for dynamic content
<div aria-live="polite" aria-atomic="true">{statusMessage}</div>

// ✅ Focus management in modals
useEffect(() => {
  if (isOpen) modalRef.current?.focus();
}, [isOpen]);

// ✅ Color contrast — minimum 4.5:1 for normal text, 3:1 for large text
// ✅ Don't rely on color alone to convey meaning (add icons or text)
// ✅ All images need meaningful alt text; decorative images: alt=""
```

---

## 12. Naming Conventions

| Thing | Convention | Example |
|---|---|---|
| Components | PascalCase | `UserCard`, `NavbarMenu` |
| Hooks | camelCase, `use` prefix | `useAuth`, `useFetch` |
| Utilities | camelCase | `formatDate`, `debounce` |
| Constants | SCREAMING_SNAKE | `MAX_RETRIES`, `API_URL` |
| CSS classes | kebab-case | `card--active`, `btn--primary` |
| CSS variables | `--kebab-case` | `--color-primary`, `--space-4` |
| Event handlers | `handle` prefix | `handleClick`, `handleSubmit` |
| Boolean props | `is/has/can/should` prefix | `isLoading`, `hasError`, `canEdit` |
| Context files | `[Name]Context.jsx` | `AuthContext.jsx` |
| Type files | `[name].types.ts` | `user.types.ts` |

---

## 13. Anti-Patterns to Avoid

```jsx
// ❌ Don't derive state from props (causes sync bugs)
function Bad({ initialCount }) {
  const [count, setCount] = useState(initialCount); // breaks if prop changes
}
// ✅ Use the prop directly or memo
const count = useMemo(() => transform(initialCount), [initialCount]);

// ❌ Don't use useEffect for data that can be computed during render
useEffect(() => {
  setFullName(`${first} ${last}`); // unnecessary effect + render
}, [first, last]);
// ✅ Compute inline
const fullName = `${first} ${last}`;

// ❌ Don't mutate state directly
state.items.push(newItem); // React won't detect this
// ✅ Return new reference
setItems(prev => [...prev, newItem]);

// ❌ Don't put objects/arrays in useEffect deps without useMemo
useEffect(() => { ... }, [{ id: 1 }]); // new reference every render = infinite loop
// ✅ Stabilize the reference
const opts = useMemo(() => ({ id: 1 }), []);
useEffect(() => { ... }, [opts]);

// ❌ Don't fetch in useEffect in new codebases
useEffect(() => { fetchUsers().then(setUsers); }, []); // no cancellation, no caching
// ✅ Use React Query or SWR

// ❌ Don't style with inline objects on every render
<div style={{ color: 'red', marginTop: 20 }} /> // new object each render
// ✅ Use CSS classes or a stable style object defined outside

// ❌ Don't hardcode magic numbers
if (status === 3) ... // what is 3?
// ✅ Use named constants
const STATUS = { PENDING: 1, ACTIVE: 2, CLOSED: 3 };
if (status === STATUS.CLOSED) ...
```

---

## Quick Reference Card

```
Folder structure    → features/ + components/ui/ + hooks/ + styles/
Component size      → <80 lines presentational, <150 lines container
Styling             → CSS variables for all tokens, never hardcode values
Responsive          → Mobile-first, clamp() for type, custom breakpoints
CSS library         → Use cn() util, never string concat for classes
Reusability         → 3-times rule, hooks for logic, UI for display
State               → Local → lift → context → global → server (React Query)
Performance         → useMemo/useCallback only when needed, lazy() for routes
A11y                → Semantic HTML, ARIA labels, keyboard nav, focus mgmt
Naming              → PascalCase components, camelCase hooks, handle* events
Anti-patterns       → No derived state, no direct mutation, no magic numbers
```

---

*Last updated: 2026 · React 18+ · Applies to Tailwind, shadcn/ui, CSS Modules, Styled Components*
