---
name: rust-cloud-native
description: Use when building cloud-native apps, microservices, or Kubernetes-deployed Rust services. Covers Docker, container builds, Dockerfile, multi-stage build, health endpoint, readiness and liveness probes, prometheus metrics, distributed tracing with OpenTelemetry and jaeger, graceful shutdown, 12-factor config, gRPC with tonic, and container optimization.
---

# Cloud-Native Development

## Domain Constraints

| Domain Rule | Design Constraint | Rust Implication |
|-------------|-------------------|------------------|
| 12-Factor | Config from env | Environment-based config |
| Observability | Metrics + traces | tracing + opentelemetry |
| Health checks | Liveness/readiness | Dedicated endpoints |
| Graceful shutdown | Clean termination | Signal handling |
| Horizontal scale | Stateless design | No local state |
| Container-friendly | Small binaries | Release optimization |

## Critical Rules

- **No local persistent state** — pods can be killed/rescheduled anytime. Use external state (Redis, DB).
- **Handle SIGTERM, drain connections** — zero-downtime deployments require graceful shutdown.
- **Every request must be traceable** — distributed systems need tracing spans + opentelemetry export.

## Key Crates

| Purpose | Crate |
|---------|-------|
| gRPC | tonic |
| Kubernetes | kube, kube-runtime |
| Docker | bollard |
| Tracing | tracing, opentelemetry |
| Metrics | prometheus, metrics |
| Config | config, figment |

## Graceful Shutdown Pattern

```rust
use tokio::signal;

async fn run_server() -> anyhow::Result<()> {
    let app = Router::new()
        .route("/health", get(health))
        .route("/ready", get(ready));

    let addr = SocketAddr::from(([0, 0, 0, 0], 8080));

    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .with_graceful_shutdown(shutdown_signal())
        .await?;

    Ok(())
}

async fn shutdown_signal() {
    signal::ctrl_c().await.expect("failed to listen for ctrl+c");
    tracing::info!("shutdown signal received");
}
```

## Health Check Pattern

```rust
async fn health() -> StatusCode {
    StatusCode::OK
}

async fn ready(State(db): State<Arc<DbPool>>) -> StatusCode {
    match db.ping().await {
        Ok(_) => StatusCode::OK,
        Err(_) => StatusCode::SERVICE_UNAVAILABLE,
    }
}
```

## Common Mistakes

| Mistake | Domain Violation | Fix |
|---------|-----------------|-----|
| Local file state | Not stateless | External storage |
| No SIGTERM handling | Hard kills | Graceful shutdown |
| No tracing | Can't debug | tracing spans |
| Static config | Not 12-factor | Env vars |
