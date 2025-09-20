# Diddy-Mesh Development Prompts

## AI Assistant Guidelines

These prompts provide context and guidelines for AI assistants working on the Diddy-Mesh project.

## Project Understanding Prompts

### Core Concept Prompt
```
You are working on Diddy-Mesh, a lightweight service mesh specifically designed for Cloudflare Workers and Pages environments. This is NOT a traditional Kubernetes-based service mesh. Key constraints:

- Must work within Cloudflare Workers execution limits (CPU time, memory, bundle size)
- Leverages Cloudflare's edge network and APIs (KV, Durable Objects, etc.)
- Focuses on edge-to-edge communication rather than container-to-container
- Prioritizes simplicity and performance over feature completeness
- Integrates with Cloudflare's existing infrastructure (DNS, CDN, security)

Always consider these constraints when suggesting solutions or implementations.
```

### Architecture Decision Prompt
```
When making architectural decisions for Diddy-Mesh:

1. Prefer Cloudflare-native solutions over external dependencies
2. Minimize bundle size and execution time impact
3. Design for edge-first, distributed operation
4. Consider Cloudflare Workers' stateless nature
5. Leverage Cloudflare's global network for service discovery
6. Ensure compatibility with Workers and Pages deployment models

Example: Instead of using a traditional database for service registry, use Cloudflare KV Store or Durable Objects.
```

## Development Task Prompts

### Feature Implementation Prompt
```
When implementing new features for Diddy-Mesh:

1. Start with the minimal viable implementation
2. Consider TypeScript-first development
3. Write comprehensive tests (unit, integration, e2e)
4. Document API changes thoroughly
5. Provide working examples
6. Consider performance impact on Workers execution
7. Ensure compatibility across different Cloudflare environments

Feature template:
- Core functionality in src/core/
- Types in packages/types/
- Tests in tests/
- Examples in examples/
- Documentation in docs/
```

### Bug Fix Prompt
```
When fixing bugs in Diddy-Mesh:

1. Reproduce the bug in a minimal test case
2. Identify the root cause in the context of Cloudflare Workers constraints
3. Implement the fix with minimal code changes
4. Add regression tests
5. Consider edge cases specific to distributed edge environments
6. Verify fix doesn't introduce performance regressions
7. Update documentation if behavior changes

Always test fixes in actual Cloudflare Workers environment when possible.
```

### Performance Optimization Prompt
```
When optimizing Diddy-Mesh performance:

1. Profile in realistic Cloudflare Workers environment
2. Focus on bundle size reduction first
3. Optimize for cold start performance
4. Minimize CPU time for routing decisions
5. Use efficient serialization/deserialization
6. Leverage Cloudflare Workers caching where appropriate
7. Consider memory allocation patterns

Measure before and after:
- Bundle size impact
- Cold start time
- Request latency overhead
- Memory usage
```

## Code Quality Prompts

### TypeScript Best Practices
```
For TypeScript code in Diddy-Mesh:

1. Use strict TypeScript configuration
2. Prefer interfaces over types for public APIs
3. Use generics for flexible, reusable components
4. Implement proper error types
5. Document complex types with JSDoc
6. Use discriminated unions for state management
7. Prefer immutable data patterns

Example:
interface ServiceConfig {
  readonly name: string;
  readonly endpoints: readonly string[];
  readonly timeout?: number;
}
```

### Testing Approach Prompt
```
For testing Diddy-Mesh components:

1. Unit tests: Test individual functions in isolation
2. Integration tests: Test component interactions
3. E2E tests: Test complete workflows in Workers environment
4. Mock Cloudflare APIs appropriately in tests
5. Test error conditions and edge cases
6. Use property-based testing for complex logic
7. Benchmark performance-critical paths

Testing tools:
- Jest for unit/integration tests
- Playwright for e2e tests
- Miniflare for Workers environment simulation
```

## Documentation Prompts

### API Documentation Prompt
```
When documenting Diddy-Mesh APIs:

1. Use clear, concise descriptions
2. Provide working code examples
3. Document all parameters and return types
4. Include common usage patterns
5. Explain Cloudflare-specific considerations
6. Link to related concepts
7. Include error handling examples

Format:
```typescript
/**
 * Registers a service with the mesh registry
 * @param config - Service configuration
 * @param options - Registration options
 * @returns Promise resolving to service handle
 * @throws {RegistrationError} When service name conflicts
 * @example
 * ```typescript
 * const service = await mesh.register({
 *   name: 'user-service',
 *   endpoints: ['https://users.example.workers.dev']
 * });
 * ```
 */
async register(config: ServiceConfig, options?: RegisterOptions): Promise<ServiceHandle>
```

### Tutorial Writing Prompt
```
When writing tutorials for Diddy-Mesh:

1. Start with clear learning objectives
2. Provide complete, working examples
3. Explain Cloudflare Workers setup steps
4. Include troubleshooting sections
5. Show actual deployment to Cloudflare
6. Cover common pitfalls and solutions
7. Link to related documentation

Structure:
- Prerequisites
- Step-by-step instructions
- Working code samples
- Deployment verification
- Next steps
```

## Error Handling Prompts

### Error Design Prompt
```
For error handling in Diddy-Mesh:

1. Create specific error types for different failure modes
2. Include helpful context in error messages
3. Design for graceful degradation
4. Consider network failures and timeouts
5. Provide actionable error messages
6. Log errors appropriately for debugging
7. Handle Cloudflare Workers execution limits

Error categories:
- Configuration errors (developer fixable)
- Network errors (transient, retryable)
- Service errors (circuit breaker applicable)
- System errors (requires escalation)
```

## Security Considerations Prompt

```
When implementing security features in Diddy-Mesh:

1. Secure by default configuration
2. Use Cloudflare's security primitives when available
3. Implement proper authentication between services
4. Validate all inputs thoroughly
5. Avoid storing sensitive data in KV unnecessarily
6. Use HTTPS for all service communication
7. Implement rate limiting and DDoS protection

Security checklist:
- Input validation
- Output encoding
- Authentication mechanisms
- Authorization policies
- Audit logging
- Secure defaults
```

## Deployment and Operations Prompts

### Deployment Strategy Prompt
```
For Diddy-Mesh deployment considerations:

1. Design for zero-downtime deployments
2. Support gradual rollouts and canary deployments
3. Provide rollback mechanisms
4. Consider multi-region deployments
5. Handle configuration updates gracefully
6. Monitor deployment health
7. Integrate with Cloudflare's deployment tools

Deployment patterns:
- Blue-green deployment
- Canary releases
- Feature flags
- Circuit breakers
- Health checks
```

### Monitoring and Observability Prompt
```
For observability in Diddy-Mesh:

1. Instrument all critical code paths
2. Use structured logging with consistent formats
3. Collect relevant metrics for service mesh operations
4. Implement distributed tracing
5. Provide health check endpoints
6. Monitor Cloudflare Workers-specific metrics
7. Create actionable alerts

Observability stack:
- Metrics: Request rates, latencies, error rates
- Logs: Structured JSON with correlation IDs
- Traces: Request flow through mesh
- Health: Service and infrastructure status
```