# Technical Context

## Runtime & Build Tools

### Bun (Backend)
- Version: 1.x (latest stable)
- Features used: HTTP server, test runner, bundler, workspaces
- Configuration: `bunfig.toml` for workspace settings

### Node.js Compatibility
- Bun provides Node.js compatibility layer
- Some packages may require Node.js-specific polyfills

## Backend Stack

### Hono Framework
- Lightweight, fast web framework
- Native TypeScript support
- Middleware-based architecture
- RPC support for type-safe API calls

### Database (Configurable)
- Default: SQLite for development
- Production: PostgreSQL recommended
- ORM: Drizzle ORM (type-safe, lightweight)

### Authentication
- JWT-based token authentication
- Refresh token rotation
- Secure cookie storage

## Frontend Stack

### Vue 3
- Composition API with `<script setup>`
- Reactive primitives: `ref`, `reactive`, `computed`
- Lifecycle hooks: `onMounted`, `onUnmounted`, etc.

### Pinia
- Composition API-style stores
- DevTools integration
- SSR-ready (if needed later)

### Vue Router
- History mode routing
- Route guards for authentication
- Lazy loading for code splitting

### Styling
- Tailwind CSS (utility-first)
- Component-scoped styles when needed

## Shared Packages

### @app/shared-types
- API request/response types
- Domain entity types
- Validation schemas (Zod)

### @app/shared-utils
- Pure utility functions
- Date formatting, validation helpers
- No side effects, fully testable

## Testing Tools

### Vitest
- Unit and integration testing
- Compatible with Jest API
- Fast, native ESM support

### Playwright
- E2E browser testing
- Cross-browser support
- Visual regression testing

### Cucumber.js
- BDD testing framework
- Gherkin syntax support
- Step definitions in TypeScript

## Development Tools

### ESLint
- TypeScript-aware linting
- Vue-specific rules
- Architecture boundary checks

### TypeScript
- Strict mode enabled
- Project references for monorepo
- Path aliases for clean imports

## Environment Configuration
```
NODE_ENV=development|production|test
API_PORT=3000
DATABASE_URL=sqlite://./dev.db
JWT_SECRET=<from-env>
CORS_ORIGINS=http://localhost:5173
```

## API Design
- RESTful endpoints following JSON:API spec
- Versioned routes: `/api/v1/*`
- Standard error response format
- Request validation with Zod
