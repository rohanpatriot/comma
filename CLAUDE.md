# CLAUDE.md - AI Assistant Guide for Comma

This document provides comprehensive guidance for AI assistants working on the Comma codebase. It covers architecture, conventions, workflows, and key patterns to follow when making changes.

## Table of Contents

- [Project Overview](#project-overview)
- [Technology Stack](#technology-stack)
- [Architecture Overview](#architecture-overview)
- [Directory Structure](#directory-structure)
- [Key Conventions](#key-conventions)
- [Development Workflow](#development-workflow)
- [Database Schema](#database-schema)
- [API Routes](#api-routes)
- [Component Patterns](#component-patterns)
- [Authentication & Authorization](#authentication--authorization)
- [Environment Variables](#environment-variables)
- [Code Quality & Formatting](#code-quality--formatting)
- [AI Assistant Guidelines](#ai-assistant-guidelines)

---

## Project Overview

**Comma** is an open-source minimal blogging platform that allows users to create beautiful personal websites with rich content. It's built with modern web technologies and follows the App Router pattern of Next.js 15.

### Key Features
- Rich text editor (TipTap) with Notion-like experience
- Analytics powered by Tinybird
- Email collection and newsletter functionality
- Custom domain support
- Multiple page themes (default, link-in-bio, freestyle)
- Projects, bookmarks, and work experience showcases
- Protected content with password authentication
- SEO optimization

### License
AGPLv3 - All contributions must comply with this license.

---

## Technology Stack

### Core Framework
- **Next.js 15** (App Router) - React framework with server components
- **React 19** - UI library
- **TypeScript 5** - Type safety

### Styling
- **Tailwind CSS 4** - Utility-first CSS framework
- **Radix UI** - Accessible component primitives
- **Framer Motion** - Animation library
- **class-variance-authority** - Component variant styling

### Backend & Database
- **Prisma 6** - ORM with Neon adapter
- **PostgreSQL** (Neon) - Database with full-text search
- **NextAuth.js** - Authentication (Google, GitHub, Email)
- **Upstash Redis** - Rate limiting and caching

### Content & Editor
- **TipTap** - Rich text editor
- **tiptap-markdown** - Markdown support
- **Katex** - Math rendering
- **react-email** - Email templates

### Analytics & Monitoring
- **Tinybird** - Analytics platform
- **Vercel Analytics** - Performance monitoring
- **LogSnag** - Event logging

### Payments & Storage
- **Lemon Squeezy** - Payment processing
- **Vercel Blob** - File storage
- **Resend** - Email delivery

### Developer Tools
- **ESLint** - Linting
- **Prettier** - Code formatting with plugins (tailwindcss, organize-imports)
- **Prisma Studio** - Database studio for Prisma
- **React Email** - Email development

---

## Architecture Overview

### Multi-Domain Architecture

Comma uses a sophisticated multi-domain architecture handled by middleware:

1. **App Domain** (`app.comma.to`) - Admin dashboard and content management
2. **User Domain** (`.comma.to` subdomains) - User-facing blogs
3. **Custom Domains** - Full custom domain support
4. **Legacy Domains** - Redirects from old domains (nucelo.co/com)

The middleware (`src/middleware.ts`) handles:
- Domain routing and rewrites
- Session-based authentication checks
- Password-protected site access
- Legacy domain redirects
- Subdomain to user mapping

### Route Groups

The app uses Next.js route groups for organization:

```
src/app/
├── (app)/          # Admin dashboard (app.comma.to)
├── (user)/         # User-facing pages (subdomains)
├── (home)/         # Marketing pages (comma.to)
├── (protected)/    # Password protection page
└── (unsubscribe)/  # Newsletter unsubscribe
```

### Data Flow

1. **Client Request** → Middleware (domain routing)
2. **Server Components** → Direct database queries via Prisma
3. **Server Actions** → Mutations and side effects
4. **API Routes** → External integrations and webhooks
5. **Edge Functions** → Performance-critical operations

---

## Directory Structure

```
comma/
├── emails/                      # React Email templates
│   ├── components/              # Shared email components
│   ├── magic-link.tsx
│   ├── newsletter.tsx
│   └── welcome-email.tsx
├── prisma/
│   └── schema.prisma            # Database schema
├── public/
│   ├── _static/                 # Static assets
│   ├── fonts/                   # Custom fonts
│   └── llms.txt                 # LLM context file
├── src/
│   ├── app/                     # Next.js App Router
│   │   ├── (app)/              # Admin dashboard
│   │   │   └── app/
│   │   │       ├── (app)/      # Main app pages
│   │   │       └── (console)/  # Admin console
│   │   ├── (user)/             # User-facing pages
│   │   │   └── user/[domain]/  # Dynamic user pages
│   │   ├── (home)/             # Marketing site
│   │   ├── (protected)/        # Password protection
│   │   ├── (unsubscribe)/      # Unsubscribe flow
│   │   └── api/                # API routes
│   ├── components/             # React components
│   │   ├── ui/                 # Base UI components (shadcn)
│   │   ├── editor/             # TipTap editor components
│   │   ├── analytics/          # Analytics components
│   │   ├── articles/           # Article components
│   │   ├── bookmarks/          # Bookmark components
│   │   ├── projects/           # Project components
│   │   └── ...
│   ├── config/                 # Configuration files
│   │   ├── site.ts             # Site configuration
│   │   ├── app.ts              # App navigation
│   │   ├── marketing.ts        # Marketing site config
│   │   └── subscriptions.ts    # Subscription plans
│   ├── hooks/                  # Custom React hooks
│   ├── lib/                    # Utility libraries
│   │   ├── actions/            # Server actions
│   │   ├── constants/          # Constants and enums
│   │   ├── fetchers/           # Data fetchers
│   │   ├── validations/        # Zod schemas
│   │   ├── auth.ts             # NextAuth config
│   │   ├── db.ts               # Prisma client
│   │   ├── analytics.ts        # Tinybird integration
│   │   ├── ratelimit.ts        # Rate limiting
│   │   └── utils.ts            # Utility functions
│   ├── styles/                 # Global styles
│   ├── types/                  # TypeScript type definitions
│   └── middleware.ts           # Next.js middleware
├── .env.example                # Environment variables template
├── .eslintrc.json              # ESLint configuration
├── next.config.ts              # Next.js configuration
├── package.json                # Dependencies
├── prettier.config.mjs         # Prettier configuration
├── tailwind.config.ts          # Tailwind configuration
└── tsconfig.json               # TypeScript configuration
```

---

## Key Conventions

### File Naming

- **Components**: PascalCase for component files matching component name
- **Utilities**: kebab-case for utility files
- **Config**: kebab-case for configuration files
- **Types**: kebab-case with `.d.ts` extension
- **Routes**: Use Next.js conventions (page.tsx, route.ts, layout.tsx)

### Code Organization

#### Import Order (enforced by Prettier)
1. React and Next.js imports
2. Third-party libraries
3. Internal imports (using path aliases)
4. Type imports
5. Relative imports

#### Path Aliases
```typescript
"@/*"         // src/*
"@/emails/*"  // emails/*
"@/user/*"    // src/app/(user)/user/[domain]/*
```

### Component Patterns

#### Server Components (Default)
```typescript
// No "use client" directive
export default async function ArticlePage({ params }: Props) {
  const article = await db.article.findUnique(...);
  return <ArticleView article={article} />;
}
```

#### Client Components
```typescript
"use client";

import { useState } from "react";

export function InteractiveComponent() {
  const [state, setState] = useState();
  // ...
}
```

#### Server Actions
```typescript
"use server";

import { revalidatePath } from "next/cache";
import { db } from "@/lib/db";

export async function updateArticle(id: string, data: FormData) {
  // Validation
  // Database operation
  revalidatePath(`/articles/${id}`);
  return { success: true };
}
```

### Styling Conventions

#### Tailwind Usage
- Use utility classes directly in JSX
- Use `cn()` helper for conditional classes
- Extract repeated patterns into components, not custom CSS
- Use Tailwind config for theme tokens

#### Component Variants (CVA)
```typescript
import { cva } from "class-variance-authority";

const buttonVariants = cva("base-classes", {
  variants: {
    variant: {
      default: "...",
      outline: "...",
    },
    size: {
      sm: "...",
      lg: "...",
    },
  },
  defaultVariants: {
    variant: "default",
    size: "sm",
  },
});
```

### Database Conventions

#### Prisma Models
- Use camelCase for field names
- Use `@map()` for snake_case database columns
- Use `@relation()` for relationships with explicit onDelete
- Use enums for fixed value sets
- Use `@unique([field1, field2])` for composite unique constraints

#### Queries
- Use `db` from `@/lib/db` (singleton Prisma client)
- Use `.findUnique()` for single records by unique field
- Use `.findFirst()` for single records with complex queries
- Use `select` to limit returned fields
- Use `include` for relations
- Consider N+1 query performance

---

## Development Workflow

### Setup

```bash
# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
# Fill in required environment variables

# Generate Prisma client
npx prisma generate

# Run database migrations (if needed)
npx prisma migrate dev

# Start development server
npm run dev

# Optional: Start email development server
npm run email

# Optional: Open Prisma Studio
npx prisma studio
```

### Development Commands

```bash
npm run dev          # Start Next.js dev server with Turbo
npm run build        # Build production bundle
npm run start        # Start production server
npm run format       # Format code with Prettier
npm run email        # Email development server (port 7777)
npx prisma studio    # Prisma Studio for database inspection
```

### Git Workflow

1. Create feature branch from main
2. Make changes following conventions
3. Format code before committing (`npm run format`)
4. Write clear commit messages
5. Create PR with description of changes
6. Ensure builds pass before merging

### Testing Changes

- Test both app domain and user domain flows
- Verify authentication states (logged in/out)
- Check mobile responsiveness
- Test protected content access
- Verify analytics tracking
- Test email functionality (use email dev server)

---

## Database Schema

### Core Models

#### User
Primary entity representing a user/blogger. Contains profile, settings, and subscription info.

**Key Fields:**
- `id`, `email`, `username` - Identity
- `domain` - Custom domain (nullable)
- `category`, `location`, `title` - Profile
- `newsletter`, `newsletterCta` - Newsletter settings
- `theme` - Page theme (default/linkInBio/freeStyle)
- `sections`, `navLinks`, `links` - JSON configuration
- `lsId`, `lsVariantId`, `lsCurrentPeriodEnd` - Lemon Squeezy subscription
- `showBranding`, `showOnExplore`, `showBottomNav` - Display settings

#### Article
Blog posts with full content management.

**Key Fields:**
- `id`, `slug`, `authorId` - Identity (unique on [authorId, slug])
- `title`, `subTitle`, `content` - Content
- `published`, `publishedAt` - Publication status
- `tags` - Array of tags
- `seoTitle`, `seoDescription`, `ogImage` - SEO
- `isPinned` - Featured flag
- `views` - View count
- `lastNewsletterSentAt` - Newsletter tracking

#### Project
Portfolio projects/work items.

**Key Fields:**
- Similar to Article
- `year` - Project year
- `url` - Project link
- `password` - Optional protection
- `description` - Short description

#### Page
Custom pages with flexible content.

**Key Fields:**
- Similar to Article
- `visibility` - visible/unlisted enum
- `password` - Optional protection

#### Bookmark
Saved links/resources.

**Key Fields:**
- `id`, `title`, `url`
- `collectionId` - Optional grouping
- `clicks` - Click tracking
- `isPinned` - Featured flag

#### Callout
Call-to-action cards (hiring, partnerships, etc.).

**Key Fields:**
- `category` - CalloutCategory enum
- `title`, `description`, `url`
- `relatedPostURL`, `postId` - Optional related content

#### WorkExperience
Employment history entries.

**Key Fields:**
- `from`, `to` - Time period (to is string for "Present")
- `title`, `company`, `location`
- `description`, `url`

### Relationships

```
User
├── articles (1:many)
├── projects (1:many)
├── pages (1:many)
├── bookmarks (1:many)
├── collections (1:many)
├── workExperiences (1:many)
├── callouts (1:many)
└── subscribers (1:many)

Collection
└── bookmarks (1:many)
```

---

## API Routes

### Structure

API routes are organized by resource in `src/app/api/`:

```
api/
├── admin/              # Admin operations
├── analytics/          # Analytics endpoints
├── articles/           # Article CRUD
├── assets/             # Asset management
├── auth/               # NextAuth endpoints
├── bookmarks/          # Bookmark operations
├── explore/            # Public explore API
├── og/                 # Open Graph image generation
├── pages/              # Page operations
├── projects/           # Project operations
├── subscribers/        # Newsletter subscribers
├── user/               # User profile operations
└── webhooks/           # External webhooks (Lemon Squeezy, etc.)
```

### Common Patterns

#### Route Handler Example
```typescript
import { NextRequest, NextResponse } from "next/server";
import { getServerSession } from "next-auth";
import { authOptions } from "@/lib/auth";
import { db } from "@/lib/db";

export async function GET(request: NextRequest) {
  const session = await getServerSession(authOptions);
  if (!session) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  const data = await db.user.findUnique({
    where: { id: session.user.id },
  });

  return NextResponse.json(data);
}
```

#### Rate Limiting
```typescript
import { ratelimit } from "@/lib/ratelimit";

export async function POST(request: NextRequest) {
  const ip = request.ip ?? "127.0.0.1";
  const { success } = await ratelimit.limit(ip);

  if (!success) {
    return NextResponse.json(
      { error: "Too many requests" },
      { status: 429 }
    );
  }

  // Handle request
}
```

---

## Component Patterns

### UI Components (src/components/ui/)

Base components following shadcn/ui patterns:
- Built on Radix UI primitives
- Fully accessible (ARIA compliant)
- Customizable with Tailwind
- Support dark mode via next-themes

**Usage:**
```typescript
import { Button } from "@/components/ui/button";
import { Dialog, DialogContent, DialogTitle } from "@/components/ui/dialog";

<Button variant="outline" size="lg">
  Click me
</Button>
```

### Editor Components (src/components/editor/)

TipTap editor with extensions:
- Rich text formatting
- Image uploads
- Math equations (Katex)
- Markdown export
- Bubble menu
- File handler

### Form Patterns

Using react-hook-form + Zod:

```typescript
"use client";

import { zodResolver } from "@hookform/resolvers/zod";
import { useForm } from "react-hook-form";
import { z } from "zod";

const schema = z.object({
  title: z.string().min(1).max(100),
  content: z.string().optional(),
});

type FormData = z.infer<typeof schema>;

export function MyForm() {
  const form = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  const onSubmit = async (data: FormData) => {
    // Handle submission
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      {/* Form fields */}
    </form>
  );
}
```

### Data Fetching Patterns

#### Server Component (Preferred)
```typescript
import { db } from "@/lib/db";

export default async function Page() {
  const data = await db.article.findMany();
  return <List items={data} />;
}
```

#### Client Component with SWR
```typescript
"use client";

import useSWR from "swr";
import { fetcher } from "@/lib/utils";

export function ClientData() {
  const { data, error, isLoading } = useSWR("/api/data", fetcher);

  if (isLoading) return <Loading />;
  if (error) return <Error />;
  return <Display data={data} />;
}
```

---

## Authentication & Authorization

### NextAuth Configuration

Located in `src/lib/auth.ts`:

**Providers:**
- Google OAuth
- GitHub OAuth
- Email (magic link via Resend)

**Adapter:**
- Prisma adapter with Neon
- Session strategy: JWT
- Pages: Custom login/signup

### Session Management

#### Server Components
```typescript
import { getServerSession } from "next-auth";
import { authOptions } from "@/lib/auth";

export default async function Page() {
  const session = await getServerSession(authOptions);
  if (!session) redirect("/login");
  // ...
}
```

#### Client Components
```typescript
"use client";

import { useSession } from "next-auth/react";

export function Component() {
  const { data: session, status } = useSession();
  if (status === "loading") return <Loading />;
  if (!session) return <LoginPrompt />;
  // ...
}
```

#### API Routes
```typescript
import { getServerSession } from "next-auth";
import { authOptions } from "@/lib/auth";

export async function GET(request: NextRequest) {
  const session = await getServerSession(authOptions);
  if (!session) {
    return new NextResponse("Unauthorized", { status: 401 });
  }
  // ...
}
```

### Protected Routes

Handled by middleware (`src/middleware.ts`):
- App domain requires authentication
- User pages can be password-protected
- Projects/pages can have individual passwords

### Authorization Patterns

```typescript
// Check ownership
const article = await db.article.findUnique({
  where: { id: params.id },
});

if (article.authorId !== session.user.id) {
  return new NextResponse("Forbidden", { status: 403 });
}
```

---

## Environment Variables

### Required Variables

**Database:**
```bash
DATABASE_URL=          # Neon PostgreSQL connection string
DIRECT_URL=            # Direct connection (for migrations)
```

**Authentication:**
```bash
NEXTAUTH_SECRET=       # Generate with: openssl rand -base64 32
NEXTAUTH_URL=          # Base URL of your app
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GITHUB_CLIENT_ID=
GITHUB_CLIENT_SECRET=
```

**Email:**
```bash
RESEND_API_KEY=        # For magic links and newsletters
```

**Storage:**
```bash
BLOB_READ_WRITE_TOKEN= # Vercel Blob for file uploads
```

**Infrastructure:**
```bash
UPSTASH_REDIS_REST_URL=
UPSTASH_REDIS_REST_TOKEN=
```

**Analytics:**
```bash
TINYBIRD_API_KEY=
TINYBIRD_API_URL=
```

**Payments:**
```bash
SQUEEZY_API_KEY=       # Lemon Squeezy
SQUEEZY_STORE_ID=
SQUEEZY_WEBHOOK_SECRET=
NUCELO_PRO_ID=         # Product ID
```

**Domain Configuration:**
```bash
NEXT_PUBLIC_URL=              # Main site URL
NEXT_PUBLIC_APP_URL=          # App dashboard URL
NEXT_PUBLIC_APP_DOMAIN=       # Main domain
NEXT_PUBLIC_USER_DOMAIN=      # User subdomain base
```

**Optional:**
```bash
VERCEL_AUTH_TOKEN=            # For domain management
VERCEL_PROJECT_ID=
VERCEL_TEAM_ID=
LOGSNAG_TOKEN=                # Event logging
TIPTAP_TOKEN=                 # TipTap Pro extensions
AES_KEY=                      # API key encryption
AES_IV=
ADMINS=                       # JSON array of admin user IDs
```

### Environment-Specific Notes

- **Development:** Use `.env.local` (gitignored)
- **Production:** Set via hosting platform (Vercel)
- **Never commit:** Real credentials or secrets

---

## Code Quality & Formatting

### ESLint Configuration

Located in `.eslintrc.json`:

**Disabled Rules:**
- `react-hooks/exhaustive-deps` - Manual dependency management
- `jsx-a11y/alt-text` - Custom image handling
- `@next/next/no-img-element` - Uses custom Image components
- `import/no-anonymous-default-export` - Allow anonymous exports

### Prettier Configuration

Located in `prettier.config.mjs`:

**Settings:**
- Trailing commas: All
- Semicolons: Yes
- Bracket spacing: Yes

**Plugins:**
- `prettier-plugin-tailwindcss` - Sorts Tailwind classes
- `prettier-plugin-organize-imports` - Auto-organizes imports

**Usage:**
```bash
npm run format  # Format all files
```

### TypeScript Configuration

Located in `tsconfig.json`:

**Key Settings:**
- Strict mode: Enabled
- Target: ES5
- Module resolution: Bundler
- JSX: preserve (handled by Next.js)
- Path aliases: `@/*`, `@/emails/*`, `@/user/*`

**Type Safety:**
- No implicit any
- Strict null checks
- No unused locals/parameters (in strict environments)

---

## AI Assistant Guidelines

### When Making Changes

1. **Understand Context First**
   - Read related files before modifying
   - Check existing patterns in similar components
   - Understand data flow (server → client)

2. **Follow Conventions**
   - Use established file naming patterns
   - Match existing code style
   - Use path aliases (@/*) not relative imports
   - Follow component patterns (server vs client)

3. **Maintain Type Safety**
   - Add proper TypeScript types
   - Use Zod for runtime validation
   - Leverage Prisma types for database models
   - Avoid `any` types

4. **Consider Performance**
   - Prefer server components when possible
   - Use client components only when needed (interactivity, hooks)
   - Optimize database queries (select only needed fields)
   - Consider pagination for large datasets

5. **Security Considerations**
   - Always validate user input (Zod schemas)
   - Check authorization (user owns resource)
   - Use rate limiting for public endpoints
   - Sanitize content before rendering
   - Never expose sensitive env vars to client

6. **Testing Considerations**
   - Test both authenticated and unauthenticated states
   - Verify mobile responsiveness
   - Check dark mode compatibility
   - Test edge cases (empty states, errors)

### Common Tasks

#### Adding a New Page

1. Create route in appropriate group
2. Add to navigation config if needed
3. Implement server component
4. Add metadata for SEO
5. Style with Tailwind
6. Test authentication requirements

#### Adding a New API Endpoint

1. Create route handler in `src/app/api/`
2. Add authentication check if needed
3. Implement rate limiting if public
4. Validate input with Zod
5. Handle errors gracefully
6. Return appropriate status codes

#### Adding a New Database Model

1. Update `prisma/schema.prisma`
2. Run `npx prisma migrate dev --name description`
3. Generate client: `npx prisma generate`
4. Create Zod schema in `src/lib/validations/`
5. Add server actions in `src/lib/actions/`
6. Create UI components as needed

#### Adding a New Component

1. Determine if server or client component
2. Place in appropriate `src/components/` subdirectory
3. Use TypeScript for props
4. Style with Tailwind
5. Make accessible (ARIA attributes)
6. Export from index if needed

### Code Review Checklist

- [ ] Follows existing patterns and conventions
- [ ] TypeScript types are properly defined
- [ ] Input validation is present
- [ ] Authorization checks are in place
- [ ] Error handling is implemented
- [ ] Code is formatted (Prettier)
- [ ] No console.logs in production code
- [ ] Comments explain "why" not "what"
- [ ] Responsive design works
- [ ] Dark mode works (if applicable)
- [ ] No security vulnerabilities introduced
- [ ] Performance considerations addressed

### Debugging Tips

1. **Check Middleware First**
   - Domain routing issues often originate here
   - Verify hostname matching logic

2. **Server vs Client Boundaries**
   - Ensure client components have "use client"
   - Don't pass non-serializable props

3. **Database Issues**
   - Check Prisma schema matches database
   - Verify migrations are applied
   - Use Prisma Studio for data inspection

4. **Authentication Issues**
   - Check session in browser dev tools
   - Verify NEXTAUTH_URL matches current domain
   - Check middleware authentication logic

5. **Build Errors**
   - Run `npm run build` locally first
   - Check for missing environment variables
   - Verify all imports are correct

### Best Practices

1. **Server Components by Default**
   - Only use client components when necessary
   - Keep client bundle size small

2. **Progressive Enhancement**
   - Ensure basic functionality works without JS
   - Use loading and error states

3. **Accessibility**
   - Use semantic HTML
   - Add ARIA labels where needed
   - Ensure keyboard navigation works

4. **SEO Optimization**
   - Add proper metadata to all pages
   - Use semantic HTML structure
   - Implement Open Graph tags

5. **Error Handling**
   - Provide user-friendly error messages
   - Log errors appropriately (LogSnag)
   - Handle edge cases gracefully

6. **Documentation**
   - Update this file when adding new patterns
   - Comment complex logic
   - Keep README.md updated

---

## Additional Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [Prisma Documentation](https://www.prisma.io/docs)
- [TipTap Documentation](https://tiptap.dev/docs)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [Radix UI Documentation](https://www.radix-ui.com/primitives/docs/overview/introduction)
- [NextAuth.js Documentation](https://next-auth.js.org/getting-started/introduction)

---

**Last Updated:** 2025-12-23

This document is maintained to help AI assistants understand and work effectively with the Comma codebase. Update it when significant architectural changes are made or new patterns are established.
