# Claude AI Assistant Context for Diddy-Mesh

## Project-Specific Instructions for Claude

This document provides specific context and instructions for Claude when working on the Diddy-Mesh project.

## Understanding the Domain

### Cloudflare Workers Ecosystem
Claude should understand that Cloudflare Workers differ significantly from traditional server environments:

- **Execution Model**: V8 isolates, not containers or VMs
- **Cold Starts**: Frequent cold starts, optimize for initialization speed
- **Runtime Limits**: 30-second CPU time limit, 128MB memory limit
- **APIs**: Limited to Web APIs and Cloudflare-specific APIs
- **State**: Stateless by design, state stored in KV/Durable Objects
- **Network**: Global edge network, not traditional data center networking

### Service Mesh Adaptations
Traditional service mesh concepts adapted for Cloudflare Workers:

- **Sidecar Proxy** → Embedded library in Worker
- **Control Plane** → Configuration in Cloudflare KV Store
- **Data Plane** → Direct Worker-to-Worker communication
- **Service Discovery** → KV/Durable Objects-based registry
- **Load Balancing** → Geographic and performance-based routing
- **Circuit Breaking** → Fail-fast with Worker execution limits

## Code Generation Guidelines

### TypeScript Patterns
When generating TypeScript code for Diddy-Mesh:

```typescript
// Prefer readonly interfaces for configuration
interface ServiceConfig {
  readonly name: string;
  readonly endpoints: readonly string[];
  readonly timeout?: number;
}

// Use branded types for type safety
type ServiceName = string & { __brand: 'ServiceName' };
type Endpoint = string & { __brand: 'Endpoint' };

// Implement proper error hierarchies
class DiddyMeshError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly context?: Record<string, unknown>
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

// Use async/await patterns consistently
async function registerService(config: ServiceConfig): Promise<void> {
  try {
    // Implementation
  } catch (error) {
    throw new DiddyMeshError(
      'Failed to register service',
      'REGISTRATION_FAILED',
      { serviceName: config.name, originalError: error }
    );
  }
}
```

### Cloudflare Workers Integration
When writing Workers-specific code:

```typescript
// Worker entry point pattern
export default {
  async fetch(request: Request, env: Env, ctx: ExecutionContext): Promise<Response> {
    const mesh = new DiddyMesh({
      serviceName: env.SERVICE_NAME,
      registry: env.KV_NAMESPACE,
      // ... other config
    });
    
    return mesh.handle(request, env, ctx);
  }
};

// Environment variables typing
interface Env {
  KV_NAMESPACE: KVNamespace;
  SERVICE_NAME: string;
  REGISTRY_TOKEN: string;
  // ... other environment variables
}

// Leverage Cloudflare APIs appropriately
async function getServiceConfig(kv: KVNamespace, serviceName: string): Promise<ServiceConfig | null> {
  const configJson = await kv.get(`service:${serviceName}`);
  return configJson ? JSON.parse(configJson) : null;
}
```

### Performance Optimization Patterns
Focus on patterns that optimize for Workers constraints:

```typescript
// Lazy initialization to reduce cold start time
class ServiceRegistry {
  private _cache?: Map<string, ServiceConfig>;
  
  private get cache(): Map<string, ServiceConfig> {
    if (!this._cache) {
      this._cache = new Map();
    }
    return this._cache;
  }
}

// Efficient serialization for KV storage
function serializeConfig(config: ServiceConfig): string {
  // Use compact JSON format, avoid unnecessary whitespace
  return JSON.stringify(config);
}

// Bundle size optimization with tree shaking
export { ServiceRegistry } from './registry';
export type { ServiceConfig, RegistryOptions } from './types';
// Avoid export * to enable better tree shaking
```

## Architecture Decision Making

### When Claude Should Suggest Cloudflare-Native Solutions

1. **Service Discovery**: Always prefer KV Store or Durable Objects over external services
2. **Configuration Management**: Use KV Store for global config, env vars for per-service config
3. **Observability**: Integrate with Cloudflare Analytics, avoid external APM unless necessary
4. **Security**: Use Cloudflare Access, API shields, and Workers built-in security features
5. **Caching**: Leverage Cloudflare's CDN and Workers cache API
6. **Database**: Prefer Cloudflare D1 or KV over external databases

### Performance Trade-offs Claude Should Consider

1. **Bundle Size vs. Features**: Always favor smaller bundles
2. **Cold Start vs. Runtime Performance**: Optimize for cold starts first
3. **Memory Usage vs. Caching**: Be careful with in-memory caches
4. **Network Calls vs. Local Processing**: Minimize external API calls
5. **Complexity vs. Reliability**: Prefer simple, reliable solutions

## Common Anti-patterns to Avoid

### Cloudflare Workers Anti-patterns
Claude should avoid suggesting:

```typescript
// ❌ Don't use Node.js APIs
import fs from 'fs';
import path from 'path';

// ❌ Don't use heavy dependencies
import lodash from 'lodash';
import moment from 'moment';

// ❌ Don't create persistent connections
class DatabasePool {
  private connections: Connection[] = [];
}

// ❌ Don't use synchronous blocking operations
function blockingOperation() {
  for (let i = 0; i < 1000000; i++) {
    // Synchronous heavy computation
  }
}
```

### Preferred Cloudflare Workers Patterns

```typescript
// ✅ Use Web APIs
const url = new URL(request.url);
const response = await fetch(endpoint);

// ✅ Use lightweight utilities or built-ins
const config = JSON.parse(configString);
const timestamp = Date.now();

// ✅ Use per-request resources
async function handleRequest(request: Request, env: Env) {
  const client = new ServiceClient(env.API_TOKEN);
  // Use client for this request only
}

// ✅ Use async operations
async function processRequest(request: Request): Promise<Response> {
  const data = await fetchExternalData();
  return new Response(JSON.stringify(data));
}
```

## Testing Guidance for Claude

### Test Structure
When generating tests, follow this structure:

```typescript
describe('ServiceRegistry', () => {
  let mockKV: KVNamespace;
  let registry: ServiceRegistry;

  beforeEach(() => {
    mockKV = createMockKV();
    registry = new ServiceRegistry(mockKV);
  });

  describe('register', () => {
    it('should register a service successfully', async () => {
      const config: ServiceConfig = {
        name: 'test-service',
        endpoints: ['https://test.workers.dev']
      };

      await registry.register(config);

      const stored = await mockKV.get('service:test-service');
      expect(JSON.parse(stored!)).toEqual(config);
    });

    it('should throw error for duplicate service names', async () => {
      const config: ServiceConfig = {
        name: 'test-service',
        endpoints: ['https://test.workers.dev']
      };

      await registry.register(config);

      await expect(registry.register(config)).rejects.toThrow(
        'Service test-service already registered'
      );
    });
  });
});
```

### Mock Patterns
Use these patterns for mocking Cloudflare APIs:

```typescript
// Mock KV Namespace
function createMockKV(): KVNamespace {
  const store = new Map<string, string>();
  
  return {
    async get(key: string): Promise<string | null> {
      return store.get(key) ?? null;
    },
    async put(key: string, value: string): Promise<void> {
      store.set(key, value);
    },
    async delete(key: string): Promise<void> {
      store.delete(key);
    },
    async list(): Promise<{ keys: { name: string }[] }> {
      return { keys: Array.from(store.keys()).map(name => ({ name })) };
    }
  } as KVNamespace;
}

// Mock fetch for external API calls
global.fetch = jest.fn();
const mockFetch = fetch as jest.MockedFunction<typeof fetch>;
```

## Documentation Standards

### Code Documentation
When documenting code, follow these patterns:

```typescript
/**
 * Manages service registration and discovery in the Diddy-Mesh.
 * 
 * This class provides a simplified interface for service mesh operations
 * specifically designed for Cloudflare Workers environments.
 * 
 * @example
 * ```typescript
 * const registry = new ServiceRegistry(env.KV_NAMESPACE);
 * 
 * await registry.register({
 *   name: 'user-service',
 *   endpoints: ['https://users.example.workers.dev']
 * });
 * 
 * const service = await registry.discover('user-service');
 * ```
 */
class ServiceRegistry {
  /**
   * Creates a new service registry instance.
   * 
   * @param kv - Cloudflare KV namespace for storing service data
   * @param options - Optional configuration for the registry
   */
  constructor(
    private readonly kv: KVNamespace,
    private readonly options: RegistryOptions = {}
  ) {}
}
```

### API Reference Format
Use this format for API documentation:

```markdown
### `registry.register(config)`

Registers a new service with the mesh.

**Parameters:**
- `config` (ServiceConfig): Service configuration object
  - `name` (string): Unique service identifier
  - `endpoints` (string[]): Service endpoint URLs
  - `timeout?` (number): Request timeout in milliseconds

**Returns:** Promise<void>

**Throws:**
- `RegistrationError`: When service name already exists
- `ValidationError`: When configuration is invalid

**Example:**
```typescript
await registry.register({
  name: 'user-service',
  endpoints: ['https://users.example.workers.dev'],
  timeout: 5000
});
```
```

## Error Handling Philosophy

### Error Types Claude Should Generate
Create specific error classes for different failure modes:

```typescript
export class ServiceNotFoundError extends DiddyMeshError {
  constructor(serviceName: string) {
    super(
      `Service '${serviceName}' not found in registry`,
      'SERVICE_NOT_FOUND',
      { serviceName }
    );
  }
}

export class NetworkTimeoutError extends DiddyMeshError {
  constructor(endpoint: string, timeout: number) {
    super(
      `Request to '${endpoint}' timed out after ${timeout}ms`,
      'NETWORK_TIMEOUT',
      { endpoint, timeout }
    );
  }
}

export class ConfigurationError extends DiddyMeshError {
  constructor(field: string, reason: string) {
    super(
      `Invalid configuration for '${field}': ${reason}`,
      'CONFIGURATION_ERROR',
      { field, reason }
    );
  }
}
```

### Error Recovery Patterns
Implement graceful degradation:

```typescript
async function discoverService(name: string): Promise<ServiceConfig> {
  try {
    return await this.registry.get(name);
  } catch (error) {
    if (error instanceof NetworkTimeoutError) {
      // Fall back to cached value
      return this.getCachedService(name);
    }
    throw error;
  }
}
```

This context should help Claude generate appropriate, high-quality code and documentation for the Diddy-Mesh project while respecting the unique constraints and opportunities of the Cloudflare Workers platform.