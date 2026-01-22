# Technology Stack

> **Project Type**: Personal/Side Project
> **Primary Priorities**: Developer Experience, Performance & Scale
> **Last Updated**: 2025-01-22

## Architecture Overview

This is a modern full-stack application leveraging **edge computing** for maximum performance and global distribution. The architecture follows a **server-first rendering** approach with React 19, deployed entirely on Cloudflare's edge network.

```
┌─────────────────────────────────────────────────────────────────┐
│                     Client Browser                               │
│                  React 19 + TanStack Router                      │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Cloudflare Edge Network                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Cloudflare Workers (Hono + TanStack Start SSR)         │    │
│  │  ├─ D1 Database Middleware (Drizzle Client)             │    │
│  │  ├─ Clerk Authentication Middleware                     │    │
│  │  └─ Custom API Routes                                   │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              │                                   │
│  ┌───────────────────────────┼─────────────────────────────┐    │
│  │                           ▼                             │    │
│  │  ┌───────────────────────────────────────────────────┐  │    │
│  │  │  Cloudflare D1 (SQLite Database)                  │  │    │
│  │  │  - User data and application state                │  │    │
│  │  │  - Drizzle ORM with D1 adapter                    │  │    │
│  │  └───────────────────────────────────────────────────┘  │    │
│  │                                                           │    │
│  │  ┌───────────────────────────────────────────────────┐  │    │
│  │  │  Clerk (Authentication Service)                   │  │    │
│  │  │  - Sign-in/sign-up, session management            │  │    │
│  │  │  - JWT token validation                           │  │    │
│  │  └───────────────────────────────────────────────────┘  │    │
│  └───────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

---

## Frontend Stack

### Core Framework
| Technology | Version | Purpose |
|------------|---------|---------|
| **React** | 19.2.0 | UI framework with Server Components support |
| **TanStack Start** | 1.132.0 | Full-stack React framework with file-based routing |
| **TanStack Router** | 1.132.0 | File-based routing with SSR and type-safe navigation |

**Why TanStack Start?**
- Built-in React 19 Server Components support
- File-based routing with automatic code-splitting
- Type-safe navigation and data loading
- Excellent developer experience with first-class TypeScript support
- Designed for edge deployment from the ground up

### UI Components & Styling
| Technology | Version | Purpose |
|------------|---------|---------|
| **Tailwind CSS** | 4.0.6 | Utility-first CSS framework |
| **Radix UI** | Various | Headless, accessible component primitives |
| **Lucide React** | 0.562.0 | Icon library |
| **Sonner** | 2.0.7 | Toast notifications |
| **next-themes** | 0.4.6 | Dark mode support |
| **Recharts** | 2.15.4 | Data visualization charts |

**Why Radix UI over alternatives?**
- Unstyled components give full styling control with Tailwind
- Excellent accessibility (WCAG 2.1+ compliant) built-in
- Compound component pattern for flexibility
- No runtime overhead (zero dependencies)

### Form & Data Management
| Technology | Version | Purpose |
|------------|---------|---------|
| **React Hook Form** | 7.68.0 | Form state management and validation |
| **Zod** | 4.2.1 | Schema validation with TypeScript inference |
| **@hookform/resolvers** | 5.2.2 | Zod integration for React Hook Form |

**Why this stack?**
- React Hook Form minimizes re-renders (performance)
- Zod provides type-safe validation with runtime checks
- Excellent TypeScript integration throughout

### Developer Tools
| Technology | Version | Purpose |
|------------|---------|---------|
| **@tanstack/react-devtools** | 0.7.0 | React and Router devtools |
| **Biome** | 0.3.3 | Fast linter and formatter (Rust-based) |

---

## Backend Stack

### Runtime & Framework
| Technology | Version | Purpose |
|------------|---------|---------|
| **Cloudflare Workers** | - | Edge JavaScript runtime (V8 isolates) |
| **Hono** | 4.10.4 | Ultra-fast web framework for edge |
| **TanStack Start SSR** | 1.132.0 | Server-side rendering integration |

**Why Hono?**
- Built specifically for edge runtimes (Workers, Deno, Bun)
- Minimal overhead (fastest web framework for edge)
- Middleware-based architecture
- Excellent TypeScript support
- Wraps TanStack Start seamlessly

### Authentication & Authorization
| Technology | Version | Purpose |
|------------|---------|---------|
| **Clerk** | 5.53.8 / 2.20.0 | Complete authentication solution |
| **@hono/clerk-auth** | 3.0.3 | Hono middleware for Clerk |

**Why Clerk?**
- Eliminates building auth from scratch (saves weeks)
- Built-in OAuth providers (Google, GitHub, etc.)
- Session management, JWT validation, MFA
- Excellent React SDK for TanStack Start
- RBAC (Role-Based Access Control) with granular permissions

**Role Structure:**
- `user` - Default role for authenticated users
- `admin` - Administrative access
- `superadmin` - Full system access

### Database & ORM
| Technology | Version | Purpose |
|------------|---------|---------|
| **Cloudflare D1** | - | SQLite-based edge database |
| **Drizzle ORM** | 0.44.7 | Type-safe ORM with zero overhead |
| **Drizzle Kit** | 0.31.7 | Migration and schema management tooling |

**Why D1 + Drizzle?**
- **D1**: Only SQL database built for the edge (low latency globally)
- **Drizzle**: Type-safe queries, minimal bundle size, excellent DX
- SQLite: Simple, reliable, perfect for read-heavy workloads
- No connection pooling issues (serverless-friendly)

**Access Patterns:**
- Server functions: `env.DB` from `cloudflare:workers`
- Hono routes: `c.get("Database")` from middleware
- Migrations: Automatic via `drizzle-kit generate`

---

## Infrastructure & Deployment

### Hosting Platform
| Technology | Purpose |
|------------|---------|
| **Cloudflare Workers** | Edge compute runtime (400+ cities globally) |
| **Wrangler** | Deployment CLI and configuration management |

**Deployment Environments:**
- `production` - Live production environment
- `staging` - Pre-production testing
- `staging-bo` - Backup/overflow staging (if needed)

### Environment Variables
```jsonc
// wrangler.jsonc
{
  "vars": {
    // Client-side (prefixed with VITE_)
    "VITE_CLERK_PUBLISHABLE_KEY": "...",

    // Server-side
    "CLERK_PUBLISHABLE_KEY": "...",
    "CLERK_SECRET_KEY": "...",
    "CLERK_JWT_ISSUER_DOMAIN": "..."
  }
}
```

### Database Binding
```jsonc
{
  "d1_databases": [
    {
      "binding": "DB",
      "database_id": "...",
      "database_name": "app-db"
    }
  ]
}
```

### Observability
- **Cloudflare Workers Analytics**: Built-in request metrics, error tracking
- **Wrangler Tail**: Real-time log streaming (`bun run tail:prod`)

---

## Development Tools

### Package Management
| Tool | Purpose |
|------|---------|
| **Bun** | Package manager and runtime (faster than npm) |

### Code Quality
| Tool | Purpose |
|------|---------|
| **TypeScript** | Type safety with strict mode |
| **Biome** | Linting and formatting (replaces ESLint/Prettier) |
| **Husky** | Git hooks for pre-commit checks |

### Testing
| Tool | Purpose |
|------|---------|
| **Vitest** | Fast unit testing with native ESM support |
| **Testing Library** | React component testing utilities |
| **jsdom** | DOM implementation for Node.js tests |

### Build & Development
| Command | Purpose |
|---------|---------|
| `bun run dev` | Start development server (port 4000) |
| `bun run build` | Build for production |
| `bun run preview` | Preview production build locally |
| `bun run check` | Run all quality checks (type, lint, build) |

---

## Decision Rationale

### Why This Stack for a Personal Project?

#### 1. **Developer Experience** (Top Priority)
- **TanStack Start**: File-based routing means less boilerplate, automatic code-splitting
- **TypeScript everywhere**: End-to-end type safety from database to UI
- **Hot reload**: Fast iteration with Vite dev server
- **Drizzle ORM**: Schema-first development with auto-generated migrations

#### 2. **Performance & Scale** (Top Priority)
- **Edge deployment**: Sub-50ms response times globally via Cloudflare
- **React 19 Server Components**: Reduced client bundle size
- **D1 Database**: Single-digit millisecond queries at the edge
- **Hono**: Minimal framework overhead

#### 3. **Cost Efficiency**
- **Cloudflare Workers**: Free tier handles 100k requests/day
- **D1 Database**: Generous free tier (5GB storage, 5M reads/day)
- **Clerk**: Free tier for up to 5,000 monthly active users
- **Total cost**: $0/month for most side projects

#### 4. **Future-Proof**
- **Modern stack**: React 19, TypeScript 5.7, latest Tailwind
- **Edge-native**: Ready for the shift away from centralized servers
- **Portable**: Can migrate to Cloudflare Pages, R2, or other providers easily

---

## Trade-offs & Alternatives

### Considered & Rejected

#### Frontend Frameworks
| Alternative | Why Rejected |
|-------------|--------------|
| **Next.js** | Requires Node.js runtime, not optimized for edge deployment |
| **Remix** | Excellent choice, but TanStack Start has better TypeScript DX |
| **SvelteKit** | Great performance, but smaller ecosystem than React |

#### Backend Runtime
| Alternative | Why Rejected |
|-------------|--------------|
| **Node.js (Express/Fastify)** | Cold starts, not edge-native, higher latency |
| **Deno** | Great platform, but Cloudflare has better global distribution |
| **Byon** | Excellent, but fewer deployment options than Cloudflare |

#### Database
| Alternative | Why Rejected |
|-------------|--------------|
| **PostgreSQL (Neon/Supabase)** | Connection overhead from edge, higher latency |
| **MongoDB (Atlas)** | NoSQL overkill for most side projects |
| **SQLite (local files)** | Not designed for concurrent edge access |
| **Turso** | Great alternative to D1, but chose native Cloudflare integration |

#### Authentication
| Alternative | Why Rejected |
|-------------|--------------|
| **Auth.js (NextAuth)** | More setup required, fewer built-in features |
| **Supabase Auth** | Tied to Supabase ecosystem |
| **Custom auth** | Security risks, maintenance burden |

### Known Limitations

1. **D1 Database**:
   - No write replication (single region for writes)
   - 10MB query result size limit
   - Not suitable for write-heavy workloads

2. **Cloudflare Workers**:
   - 10ms CPU time limit on free tier, 30s on paid
   - No native WebSockets (use Cloudflare Durable Objects instead)
   - Limited Node.js compatibility (use `nodejs_compat` flag)

3. **Clerk**:
   - Vendor lock-in (can't easily switch providers)
   - Pricing scales with MAUs (can get expensive at scale)

---

## Migration Paths

If you outgrow this stack, here are migration options:

### **To Traditional Hosting**
- Move server logic to Node.js/Bun on Vercel or Railway
- Replace D1 with PostgreSQL/MySQL
- Keep TanStack Router (works on any Node.js host)

### **To Different Edge Provider**
- Cloudflare Workers → Deno Deploy or Fastly Compute@Edge
- D1 → Turso or Neon Postgres with connection pooling
- Hono framework works on all edge platforms

### **To Different Frontend**
- TanStack Start → Remix or Next.js (similar file-based routing)
- Keep React components and Tailwind styles

---

## Learning Resources

### Official Documentation
- [TanStack Start](https://tanstack.com/start/latest)
- [Cloudflare Workers](https://developers.cloudflare.com/workers/)
- [Hono](https://hono.dev/)
- [Drizzle ORM](https://orm.drizzle.team/)
- [Clerk](https://clerk.com/docs)

### Key Concepts to Master
1. **Server Components vs Client Components** (React 19)
2. **Edge Runtime Constraints** (no Node.js APIs)
3. **File-based Routing** (TanStack Router)
4. **D1 Database Patterns** (schema design, migrations)
5. **Clerk Middleware** (session management, RBAC)

---

## Summary

This stack prioritizes **developer experience** and **edge performance** for a **personal project** that can scale to production. The combination of TanStack Start, Cloudflare Workers, and Clerk provides:

- ✅ Fast iteration with excellent DX
- ✅ Global edge deployment with sub-50ms latency
- ✅ Type safety from database to UI
- ✅ Zero upfront costs with generous free tiers
- ✅ Modern React 19 with Server Components
- ✅ Production-ready authentication and authorization

**Estimated startup time**: < 5 minutes from clone to running locally
**Estimated deployment time**: < 2 minutes to production
**Monthly cost**: $0 (free tiers) → $20+ (paid tiers as you scale)
