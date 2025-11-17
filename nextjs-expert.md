---
description: Expert Next.js 15+ development with App Router, Server Components, and modern patterns
---

# Next.js Expert

Expert guidance for building Next.js 15+ applications using App Router, Server Components, Server Actions, and the latest best practices from official documentation.

**Latest Version**: Next.js 16.0.3 (as of documentation fetch)

## Core Principles

### 1. App Router Architecture

**File-System Based Routing**:
- Folders define route segments (e.g., `app/dashboard/settings/`)
- Special files define UI: `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`
- Dynamic routes: `[slug]/page.tsx` or `[...slug]/page.tsx` (catch-all)
- Route groups: `(auth)/login/page.tsx` (URL excluded)
- Parallel routes: `@modal/page.tsx` (simultaneous rendering)

**File Conventions**:
```
app/
├── layout.tsx          # Root layout (required, wraps all pages)
├── page.tsx            # Home page (/)
├── loading.tsx         # Loading UI (shown during navigation)
├── error.tsx           # Error boundary
├── not-found.tsx       # 404 page
├── global-error.tsx    # Global error handler
├── dashboard/
│   ├── layout.tsx      # Nested layout
│   ├── page.tsx        # /dashboard
│   └── [id]/
│       └── page.tsx    # /dashboard/123
└── (auth)/
    ├── login/
    │   └── page.tsx    # /login (not /auth/login)
    └── signup/
        └── page.tsx    # /signup
```

### 2. Server Components vs Client Components

**Server Components (Default)**:
- No `'use client'` directive needed
- Execute on the server only
- Can directly access databases, file systems, APIs
- Cannot use React hooks (useState, useEffect)
- Cannot use browser APIs
- Better for SEO and initial page load
- Reduce client bundle size

**When to Use Server Components**:
- ✅ Fetching data from APIs or databases
- ✅ Accessing backend resources
- ✅ Keeping sensitive information on server (API keys, tokens)
- ✅ Large dependencies that don't need client-side JavaScript
- ✅ Static content and layouts

**Client Components**:
- Require `'use client'` directive at top of file
- Execute on both server (initial render) and client (hydration)
- Can use React hooks and browser APIs
- Handle interactivity, event listeners, state

**When to Use Client Components**:
- ✅ Interactive UI (onClick, onChange handlers)
- ✅ State management (useState, useReducer)
- ✅ Effects and lifecycle (useEffect)
- ✅ Browser-only APIs (localStorage, window)
- ✅ Custom hooks
- ✅ React Context

**Composition Pattern**:
```tsx
// app/dashboard/layout.tsx (Server Component)
import { Sidebar } from './sidebar'  // Client Component
import { fetchUserData } from '@/lib/data'

export default async function DashboardLayout({ children }) {
  const user = await fetchUserData()  // Server-side data fetching

  return (
    <div>
      <Sidebar user={user} />  {/* Pass serializable data to client */}
      {children}
    </div>
  )
}

// app/dashboard/sidebar.tsx (Client Component)
'use client'

import { useState } from 'react'

export function Sidebar({ user }) {
  const [isOpen, setIsOpen] = useState(false)

  return (
    <aside>
      <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
      {isOpen && <nav>...</nav>}
    </aside>
  )
}
```

### 3. Data Fetching Patterns

**Server Components - Direct Fetch**:
```tsx
// app/posts/page.tsx
async function getPosts() {
  const res = await fetch('https://api.example.com/posts', {
    // Next.js extends fetch with caching options
    cache: 'force-cache',     // Default: cache indefinitely
    // cache: 'no-store',     // Never cache (dynamic)
    // next: { revalidate: 3600 }  // Revalidate every hour
  })
  return res.json()
}

export default async function PostsPage() {
  const posts = await getPosts()

  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

**Request Memoization** (automatic):
- Multiple identical `fetch` calls in same render are automatically deduplicated
- Only one request is made

**Using React `cache` for Non-Fetch Data**:
```tsx
import { cache } from 'react'
import db from '@/lib/db'

// Deduplicate database calls across components
export const getUser = cache(async (id: string) => {
  return await db.user.findUnique({ where: { id } })
})
```

**Parallel Data Fetching**:
```tsx
async function getData() {
  // Start all requests in parallel
  const [user, posts, comments] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchComments()
  ])

  return { user, posts, comments }
}
```

**Sequential Data Fetching** (when needed):
```tsx
async function getData() {
  const user = await fetchUser()
  // Wait for user before fetching posts
  const posts = await fetchUserPosts(user.id)
  return { user, posts }
}
```

**Streaming with Suspense**:
```tsx
import { Suspense } from 'react'

export default function Page() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<div>Loading stats...</div>}>
        <Stats />
      </Suspense>
      <Suspense fallback={<div>Loading posts...</div>}>
        <Posts />
      </Suspense>
    </div>
  )
}

async function Stats() {
  const data = await fetchStats()  // Slow query
  return <div>{data}</div>
}

async function Posts() {
  const posts = await fetchPosts()  // Fast query
  return <ul>{posts.map(...)}</ul>
}
```

### 4. Server Actions

**Creating Server Actions**:
```tsx
// app/actions/posts.ts
'use server'

import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string
  const content = formData.get('content') as string

  // Validate
  if (!title || !content) {
    return { error: 'Title and content required' }
  }

  // Database operation
  await db.post.create({
    data: { title, content }
  })

  // Revalidate cache
  revalidatePath('/posts')

  // Redirect to new post
  redirect('/posts')
}

export async function deletePost(id: string) {
  await db.post.delete({ where: { id } })
  revalidatePath('/posts')
  return { success: true }
}
```

**Using Server Actions in Forms**:
```tsx
// app/posts/create/page.tsx
import { createPost } from '@/app/actions/posts'

export default function CreatePost() {
  return (
    <form action={createPost}>
      <input name="title" placeholder="Title" required />
      <textarea name="content" placeholder="Content" required />
      <button type="submit">Create Post</button>
    </form>
  )
}
```

**Using Server Actions with Client Components**:
```tsx
'use client'

import { useActionState } from 'react'
import { createPost } from '@/app/actions/posts'

export default function CreatePostForm() {
  const [state, formAction, isPending] = useActionState(createPost, null)

  return (
    <form action={formAction}>
      <input name="title" />
      <textarea name="content" />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create'}
      </button>
      {state?.error && <p className="error">{state.error}</p>}
    </form>
  )
}
```

**Progressive Enhancement**:
- Forms with Server Actions work without JavaScript
- Users can submit forms even before hydration completes

### 5. Layouts and Pages

**Root Layout** (Required):
```tsx
// app/layout.tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        <header>Global Header</header>
        {children}
        <footer>Global Footer</footer>
      </body>
    </html>
  )
}
```

**Nested Layouts**:
```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <div className="dashboard">
      <aside>Sidebar</aside>
      <main>{children}</main>
    </div>
  )
}
```

**Pages with Props**:
```tsx
// app/posts/[slug]/page.tsx
interface PageProps {
  params: { slug: string }
  searchParams: { [key: string]: string | string[] | undefined }
}

export default async function PostPage({ params, searchParams }: PageProps) {
  const post = await getPost(params.slug)
  const filter = searchParams.category

  return <article>{post.title}</article>
}
```

**Loading UI**:
```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <div>Loading dashboard...</div>
}
```

**Error Handling**:
```tsx
// app/dashboard/error.tsx
'use client'  // Error components must be Client Components

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={reset}>Try again</button>
    </div>
  )
}
```

### 6. Navigation

**Link Component**:
```tsx
import Link from 'next/link'

export default function Nav() {
  return (
    <nav>
      <Link href="/about">About</Link>
      <Link href="/posts/123">Post 123</Link>
      <Link href="/dashboard" prefetch={false}>Dashboard</Link>
    </nav>
  )
}
```

**Programmatic Navigation**:
```tsx
'use client'

import { useRouter } from 'next/navigation'

export default function LoginButton() {
  const router = useRouter()

  const handleLogin = async () => {
    await loginUser()
    router.push('/dashboard')
    // router.refresh()  // Refresh current route
    // router.back()     // Go back
  }

  return <button onClick={handleLogin}>Login</button>
}
```

### 7. Data Revalidation

**Time-based Revalidation**:
```tsx
// Revalidate every hour
fetch('https://api.example.com/data', {
  next: { revalidate: 3600 }
})
```

**On-demand Revalidation**:
```tsx
import { revalidatePath, revalidateTag } from 'next/cache'

// Revalidate specific path
revalidatePath('/posts')
revalidatePath('/posts/[slug]', 'page')

// Revalidate by cache tag
fetch('https://api.example.com/posts', {
  next: { tags: ['posts'] }
})
revalidateTag('posts')
```

### 8. Metadata and SEO

**Static Metadata**:
```tsx
// app/about/page.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'About Us',
  description: 'Learn more about our company',
}

export default function About() {
  return <div>About page</div>
}
```

**Dynamic Metadata**:
```tsx
// app/posts/[slug]/page.tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  const post = await getPost(params.slug)

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      images: [post.image],
    },
  }
}
```

### 9. Cache Components (Next.js 16+)

**Enable in config**:
```ts
// next.config.ts
const nextConfig = {
  experimental: {
    cacheComponents: true,  // Enables Cache Components + PPR
  },
}
```

**Using `use cache` directive**:
```tsx
// app/posts/page.tsx
import { cache } from 'react'

async function getPosts() {
  'use cache'  // Cache this component/function

  const posts = await db.post.findMany()
  return posts
}

export default async function PostsPage() {
  const posts = await getPosts()
  return <div>{posts.map(...)}</div>
}
```

**Cache Lifetime**:
```tsx
import { cacheLife } from 'next/cache'

async function getData() {
  'use cache'
  cacheLife('hours')  // Predefined: 'seconds', 'minutes', 'hours', 'days', 'weeks', 'max'

  return await fetchData()
}
```

**Mixing Static and Dynamic**:
```tsx
import { Suspense } from 'react'

export default function Page() {
  return (
    <div>
      <StaticHeader />  {/* Cached with "use cache" */}
      <Suspense fallback={<Loading />}>
        <DynamicUserData />  {/* Uses cookies(), not cached */}
      </Suspense>
    </div>
  )
}
```

### 10. Optimization Best Practices

**Images**:
```tsx
import Image from 'next/image'

export default function Profile() {
  return (
    <Image
      src="/profile.jpg"
      alt="Profile"
      width={500}
      height={500}
      priority  // Load immediately (above fold)
      // loading="lazy"  // Default for below fold
    />
  )
}
```

**Fonts**:
```tsx
// app/layout.tsx
import { Inter } from 'next/font/google'

const inter = Inter({ subsets: ['latin'] })

export default function RootLayout({ children }) {
  return (
    <html lang="en" className={inter.className}>
      <body>{children}</body>
    </html>
  )
}
```

**Dynamic Imports**:
```tsx
import dynamic from 'next/dynamic'

const DynamicChart = dynamic(() => import('@/components/chart'), {
  loading: () => <p>Loading chart...</p>,
  ssr: false,  // Disable server-side rendering
})
```

**Bundle Analysis**:
```bash
ANALYZE=true pnpm build
```

## Project-Specific Patterns

### TypeScript Configuration

This project uses strict TypeScript with path aliases:
```json
{
  "compilerOptions": {
    "paths": {
      "@/components/*": ["./components/*"],
      "@/lib/*": ["./lib/*"],
      "@/actions/*": ["./app/actions/*"],
      "@/services/*": ["./lib/services/*"]
    }
  }
}
```

Always use absolute imports:
```tsx
// ✅ Good
import { Button } from '@/components/ui/button'
import { postClient } from '@/actions/clients/post-client'

// ❌ Avoid
import { Button } from '../../../components/ui/button'
```

### Service Layer Architecture

**CRITICAL**: Follow the service layer pattern:

```tsx
// app/actions/clients/post-client.ts
'use server'

import { ClientService } from '@/services/models'
import { revalidatePath } from 'next/cache'

export async function postClient(formData: FormData) {
  const clientService = new ClientService()

  // Actions use services, NEVER repositories directly
  const client = await clientService.insert({
    company_name: formData.get('company_name') as string,
    // ... other fields
  })

  revalidatePath('/[team]/clients')
  return { success: true, data: client }
}
```

### Icon Management

**ALWAYS** use centralized icon exports:
```tsx
// ✅ Good - imports from centralized barrel
import { PlusCircle, Edit, Trash } from '@/components/ui/icons'

// ❌ Bad - direct import increases bundle size
import { PlusCircle } from 'lucide-react'
```

### Form Handling

Use Server Actions with React Hook Form:
```tsx
'use client'

import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { postClient } from '@/actions/clients/post-client'

export function ClientForm() {
  const form = useForm({
    resolver: zodResolver(clientSchema),
  })

  const onSubmit = async (data) => {
    const formData = new FormData()
    Object.entries(data).forEach(([key, value]) => {
      formData.append(key, value)
    })

    await postClient(formData)
  }

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      {/* form fields */}
    </form>
  )
}
```

### Internationalization

This project uses next-intl:
```tsx
import { useTranslations } from 'next-intl'

export default function Welcome() {
  const t = useTranslations('common')
  return <h1>{t('welcome')}</h1>
}

// Server Components
import { getTranslations } from 'next-intl/server'

export default async function Page() {
  const t = await getTranslations('common')
  return <h1>{t('welcome')}</h1>
}
```

## Common Patterns Checklist

When implementing features, ensure:

- ✅ Use Server Components by default
- ✅ Add `'use client'` only when needed (interactivity, hooks)
- ✅ Server Actions for mutations (create, update, delete)
- ✅ Proper error boundaries with `error.tsx`
- ✅ Loading states with `loading.tsx` or Suspense
- ✅ Absolute imports with `@/` prefix
- ✅ Follow service layer architecture (actions → services → repositories)
- ✅ Revalidate cache after mutations
- ✅ Use centralized icon imports
- ✅ Type-safe with TypeScript
- ✅ Metadata for SEO
- ✅ Optimize images with next/image
- ✅ Don't modify ShadCN UI components

## Quick Reference Commands

```bash
# Development
pnpm dev

# Build
pnpm build

# Type check
pnpm tsc --noEmit

# Lint
pnpm lint

# Bundle analysis
ANALYZE=true pnpm build

# Generate types (Supabase)
pnpm generate:types
```

## Resources

- **Next.js Docs**: https://nextjs.org/docs
- **App Router**: https://nextjs.org/docs/app
- **Server Actions**: https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations
- **Cache Components**: https://nextjs.org/docs/app/building-your-application/rendering/cache-components
- **Project Standards**: `.junie/standards/`
