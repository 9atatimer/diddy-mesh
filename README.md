# Diddy-Mesh

A lightweight service mesh specifically designed for Cloudflare Workers and Pages environments.

## Overview

Diddy-Mesh provides service mesh capabilities optimized for Cloudflare's edge computing platform, enabling secure, observable, and reliable service-to-service communication without the complexity and overhead of traditional service mesh solutions.

## Key Features

- **🌍 Edge-Native**: Built specifically for Cloudflare's global edge network
- **⚡ Lightweight**: Minimal impact on Worker execution time and bundle size
- **🔄 Service Discovery**: Automatic service registration and discovery using Cloudflare KV
- **🎯 Intelligent Routing**: Geographic and performance-based load balancing
- **📊 Observability**: Built-in monitoring, tracing, and metrics collection
- **🔒 Security**: Optional mTLS, authentication, and authorization
- **🛡️ Resilience**: Circuit breaking, retries, and graceful degradation
- **⚙️ Zero Infrastructure**: No control plane deployment required

## Quick Start

### Installation

```bash
npm install diddy-mesh
```

### Basic Usage

```typescript
import { DiddyMesh } from 'diddy-mesh';

export default {
  async fetch(request: Request, env: Env, ctx: ExecutionContext): Promise<Response> {
    const mesh = new DiddyMesh({
      serviceName: 'user-service',
      version: '1.0.0',
      registry: env.KV_NAMESPACE,
      observability: true
    });

    return mesh.handle(request, {
      '/api/users': () => handleUsers(request),
      '/api/health': () => new Response('OK')
    });
  }
};
```

### Service Configuration

```javascript
{
  "services": {
    "user-service": {
      "endpoints": ["https://users.example.workers.dev"],
      "healthCheck": "/health",
      "timeout": 5000,
      "retries": 3
    },
    "order-service": {
      "endpoints": ["https://orders.example.workers.dev"],
      "loadBalancer": "geographic"
    }
  },
  "routing": {
    "rules": [
      {
        "path": "/api/users/*",
        "service": "user-service"
      },
      {
        "path": "/api/orders/*",
        "service": "order-service"
      }
    ]
  }
}
```

## Architecture

Diddy-Mesh implements a distributed architecture using Cloudflare's edge infrastructure:

- **Service Registry**: Stores service metadata in Cloudflare KV Store
- **Edge Router**: Embedded in each Worker for intelligent request routing
- **Observability Layer**: Distributed monitoring and tracing across Workers
- **Configuration Manager**: Dynamic configuration using Cloudflare KV

For detailed architecture information, see [ARCHITECTURE.md](./ARCHITECTURE.md).

## Documentation

- **[Architecture Guide](./ARCHITECTURE.md)** - Technical architecture and design patterns
- **[Development Guide](./GITHUB.md)** - Repository structure and development workflows
- **[Development Prompts](./PROMPT.md)** - AI assistant guidelines and development prompts
- **[Claude Context](./CLAUDE.md)** - Specific context for Claude AI assistant
- **[Project Context](./CONTEXT.md)** - Project background and problem statement

## Use Cases

- **Microservices on Workers**: Connect and manage multiple Cloudflare Workers
- **API Gateways**: Intelligent routing and load balancing for API endpoints
- **Edge Computing**: Low-latency service communication at the edge
- **Jamstack Applications**: Service mesh for Cloudflare Pages with API routes
- **Multi-tenant SaaS**: Secure service isolation and routing

## Performance

Diddy-Mesh is optimized for Cloudflare Workers constraints:

- **Bundle Size**: < 50KB gzipped impact
- **Cold Start**: < 10ms initialization overhead
- **Request Latency**: < 1ms routing decision time
- **Memory Usage**: < 1MB runtime footprint

## Comparison with Traditional Service Meshes

| Feature | Traditional Service Mesh | Diddy-Mesh |
|---------|-------------------------|------------|
| Deployment | Kubernetes sidecar | Embedded library |
| Control Plane | Dedicated infrastructure | Cloudflare KV Store |
| Service Discovery | etcd/Consul | Cloudflare KV/Durable Objects |
| Load Balancing | Pod-level | Geographic + performance |
| Observability | Prometheus/Jaeger | Cloudflare Analytics |
| Security | mTLS certificates | Cloudflare security features |
| Resource Usage | High (sidecars) | Minimal (library) |

## Contributing

We welcome contributions! Please see our [development guide](./GITHUB.md) for details on:

- Setting up the development environment
- Code style and conventions
- Testing requirements
- Pull request process

## License

MIT License - see [LICENSE](./LICENSE) file for details.

## Support

- **Issues**: Report bugs and request features on [GitHub Issues](https://github.com/9atatimer/diddy-mesh/issues)
- **Discussions**: Join the conversation on [GitHub Discussions](https://github.com/9atatimer/diddy-mesh/discussions)
- **Documentation**: Comprehensive guides available in the [docs](./docs) directory

## Roadmap

- [ ] Core service mesh functionality
- [ ] TypeScript SDK and CLI tools
- [ ] Advanced traffic management features
- [ ] Integration with Cloudflare Analytics
- [ ] Performance optimization and benchmarking
- [ ] Community examples and templates

---

Made with ❤️ for the Cloudflare Workers community
