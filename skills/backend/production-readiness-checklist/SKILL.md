---
name: production-readiness-checklist
description: Comprehensive production readiness verification, code quality gates, deployment checks, and production standards compliance for platform-go
---

# Production Readiness Checklist

This skill provides comprehensive checklists to ensure all code meets production-grade standards before deployment.

## When to Use

Apply this skill when:
- Preparing code for production deployment
- Conducting final code review before merge
- Verifying system readiness for release
- Implementing quality gates in CI/CD
- Auditing existing production code
- Planning major feature releases
- Setting up new environments
- Establishing deployment procedures

## Pre-Commit Checklist (Code Level)

### Code Quality (10 items)

- [ ] Code follows golang-production-standards skill
- [ ] All functions have clear documentation comments
- [ ] No hardcoded values (use constants or config)
- [ ] No print statements (use structured logging)
- [ ] No commented-out code (delete or explain)
- [ ] Variable names are meaningful (no single letters except i, j)
- [ ] Function names describe exact behavior
- [ ] No TODO/FIXME comments without issue reference
- [ ] Imports are organized (stdlib, third-party, internal)
- [ ] File does not exceed 200 lines (except special cases)

### Error Handling (8 items)

- [ ] All errors are wrapped with context (fmt.Errorf %w)
- [ ] No ignored errors (no blank _ assignment)
- [ ] Custom error types defined for domain errors
- [ ] Error messages are user-friendly
- [ ] No error information leaks (no secrets in messages)
- [ ] Panic only in main, never in libraries
- [ ] Error recovery implemented where needed
- [ ] Goroutine errors are properly handled

### Testing Coverage (7 items)

- [ ] Unit tests exist for all public functions
- [ ] Test coverage >= 70%
- [ ] Edge cases and error scenarios tested
- [ ] Tests use table-driven pattern where applicable
- [ ] Mocking used appropriately (not over-mocked)
- [ ] Concurrent code tested with race detector
- [ ] Tests pass locally with `go test -race -cover ./...`

### Security (8 items)

- [ ] No hardcoded secrets or credentials
- [ ] Passwords hashed with bcrypt (cost >= 12)
- [ ] All inputs validated at API boundary
- [ ] SQL injection prevented (parameterized queries)
- [ ] Path traversal prevented (validated file paths)
- [ ] No sensitive data logged (passwords, tokens, PII)
- [ ] TLS used for all external communications
- [ ] Authentication/authorization implemented

## Pre-Review Checklist (Integration Level)

### API Design (6 items)

- [ ] RESTful endpoints follow naming conventions
- [ ] Request/response DTOs used (not domain models)
- [ ] Error responses standardized
- [ ] Pagination implemented for large result sets
- [ ] API versioning clear (v1, v2, etc.)
- [ ] Documentation present (Swagger/OpenAPI)

### Database (7 items)

- [ ] Migrations are versioned and tested
- [ ] Indexes created for frequently queried columns
- [ ] Foreign keys properly defined
- [ ] No N+1 queries (use preloading)
- [ ] Transactions used for multi-step operations
- [ ] Connection pool configured correctly
- [ ] Performance tested (<100ms typical queries)

### Concurrency (5 items)

- [ ] Goroutine leaks prevented
- [ ] Race detector passes (`go test -race`)
- [ ] Context used correctly (not leaked)
- [ ] Timeouts set for all blocking operations
- [ ] Resource cleanup guaranteed (defer cleanup)

### Kubernetes (6 items)

- [ ] K8s client nil-checked for test environments
- [ ] Resources labeled properly
- [ ] Retry logic for transient failures
- [ ] Graceful shutdown implemented
- [ ] Resource requests/limits defined
- [ ] Probes configured (startup, readiness, liveness)

## Pre-Deployment Checklist (System Level)

### Configuration Management (8 items)

- [ ] All config from environment variables
- [ ] No secrets in version control
- [ ] Config validation on startup
- [ ] Defaults sensible but explicit
- [ ] Config documented in README
- [ ] Multiple environment configs tested
- [ ] Feature flags implemented where needed
- [ ] Config hot-reload tested if supported

### Logging and Monitoring (8 items)

- [ ] Structured JSON logging configured
- [ ] Log levels appropriate (Debug, Info, Warn, Error)
- [ ] Request IDs tracked through request lifecycle
- [ ] Metrics exposed at /metrics endpoint
- [ ] Health checks implemented (/health endpoint)
- [ ] Readiness/liveness probes work correctly
- [ ] Error rates monitored
- [ ] Performance metrics baseline established

### Performance (6 items)

- [ ] API response time < 200ms (p95)
- [ ] Database queries < 100ms typical
- [ ] K8s API calls < 500ms
- [ ] Memory usage < 512MB per pod
- [ ] Startup time < 30 seconds
- [ ] Load tested with expected traffic

### Security (10 items)

- [ ] Authentication implemented
- [ ] Authorization (RBAC) enforced
- [ ] Rate limiting enabled
- [ ] CORS configured correctly
- [ ] Security headers present
- [ ] Input validation enforced
- [ ] SQL injection prevention verified
- [ ] Secrets management configured
- [ ] TLS certificates valid
- [ ] Vulnerability scan passed (gosec)

### Operations (8 items)

- [ ] Runbooks written for common issues
- [ ] Alert thresholds set and tested
- [ ] Rollback procedures documented
- [ ] Backup/restore tested
- [ ] Disaster recovery plan exists
- [ ] On-call documentation complete
- [ ] Incident response procedures defined
- [ ] Service dependencies documented

## Pre-Release Checklist (Quality Gate)

### Code Review Completion (5 items)

- [ ] Minimum 2 reviewers approved
- [ ] All review comments addressed
- [ ] No blocking comments remain
- [ ] Security review completed
- [ ] Architecture review completed

### Testing Completion (8 items)

- [ ] Unit tests pass (100%)
- [ ] Integration tests pass (100%)
- [ ] Smoke tests pass
- [ ] Load tests pass
- [ ] Security tests pass
- [ ] No test skips without reason
- [ ] Coverage report reviewed
- [ ] Race detector clean

### CI/CD Pipeline (8 items)

- [ ] All GitHub Actions workflows pass
- [ ] Build completes in < 5 minutes
- [ ] Docker image builds successfully
- [ ] Linting passes (golangci-lint)
- [ ] Format check passes (gofmt)
- [ ] Vet passes (go vet)
- [ ] Dependency check passes
- [ ] License check passes (if applicable)

### Documentation (7 items)

- [ ] README.md updated
- [ ] API documentation updated
- [ ] Migration guide written (if applicable)
- [ ] Changelog entry added
- [ ] Code comments added for complex logic
- [ ] Architecture decision recorded
- [ ] Performance benchmarks updated

### Deployment Readiness (8 items)

- [ ] Deployment plan documented
- [ ] Rollback plan documented
- [ ] Communication plan ready
- [ ] Stakeholders notified
- [ ] Maintenance window scheduled (if needed)
- [ ] Monitoring configured
- [ ] Logging configured
- [ ] Alerting configured

## Production Deployment Checklist

### Pre-Deployment (10 items)

- [ ] Backup taken
- [ ] Deployment plan reviewed with team
- [ ] Rollback procedure tested
- [ ] Database migrations tested in staging
- [ ] Feature flags disabled by default
- [ ] Circuit breakers configured
- [ ] Rate limits tested
- [ ] Load balancing configured
- [ ] DNS propagation planned
- [ ] Communication channels open

### Deployment Execution (8 items)

- [ ] Deployment performed during planned window
- [ ] Deployment leader assigned
- [ ] Changes deployed incrementally
- [ ] Health checks passing after each step
- [ ] Logs monitored during deployment
- [ ] Metrics monitored during deployment
- [ ] Incidents tracked if any occur
- [ ] All steps documented in runbook

### Post-Deployment (10 items)

- [ ] All services healthy
- [ ] No error rate spike
- [ ] Performance metrics normal
- [ ] User-facing features working
- [ ] Database queries responsive
- [ ] API latency acceptable
- [ ] Memory/CPU usage normal
- [ ] All probes returning healthy
- [ ] Alerts not triggering
- [ ] Team standby for 1 hour

### Post-Release (8 items)

- [ ] Feature monitored for 24 hours
- [ ] Performance metrics stable
- [ ] Error rates normal
- [ ] User feedback positive
- [ ] No critical issues found
- [ ] Documentation updated with lessons learned
- [ ] Monitoring alerts tuned if needed
- [ ] Success communicated to stakeholders

## Production Code Compliance

### Skills Compliance Verification

Ensure code follows all applicable skills:

```
Mandatory Skills for All Code:
- golang-production-standards (required)
- error-handling-guide (required)
- security-best-practices (if handling user data)

Feature-Specific Skills:
- api-design-patterns (for API endpoints)
- database-best-practices (for database operations)
- kubernetes-integration (for K8s operations)
- testing-best-practices (for test code)
- package-organization (for new packages)
- file-structure-guidelines (for file organization)

Operations Skills:
- monitoring-observability (for logging/metrics)
- cicd-pipeline-optimization (for CI/CD)
```

### Automated Checks

```bash
# Code quality checks
go vet ./...
golangci-lint run
gofmt -l .

# Security checks
gosec ./...
trufflehog filesystem ./

# Testing
go test -race -cover -timeout 30m ./...

# Build
go build ./cmd/api
go build ./cmd/scheduler

# Docker
docker build -t platform-go:latest .

# Compliance
grep -r "TODO\|FIXME" --include="*.go" internal/ cmd/ || true
grep -r "print\|println" --include="*.go" internal/ cmd/ || true
```

## Common Failure Scenarios

### API Latency High (> 200ms p95)

Checklist:
- [ ] Database queries analyzed (use slow query log)
- [ ] N+1 queries identified and fixed
- [ ] Indexes verified on queried columns
- [ ] Connection pool size verified
- [ ] Caching strategy reviewed
- [ ] Load test results analyzed
- [ ] Network latency checked
- [ ] Third-party API latency checked

### Memory Usage High (> 512MB)

Checklist:
- [ ] Goroutine leaks detected with pprof
- [ ] Memory profiling run
- [ ] Large object allocations identified
- [ ] Cache eviction policies checked
- [ ] Database connection pool reviewed
- [ ] Resource cleanup verified
- [ ] GC tuning optimized
- [ ] Heap snapshot analyzed

### Error Rate Spike (> 1%)

Checklist:
- [ ] Error logs analyzed for pattern
- [ ] Dependencies health checked
- [ ] Database connectivity verified
- [ ] Rate limits triggered?
- [ ] Circuit breaker states checked
- [ ] Resource exhaustion checked
- [ ] Configuration changes reviewed
- [ ] Network connectivity tested

### Build Failure

Checklist:
- [ ] Compilation errors cleared
- [ ] Linting errors resolved
- [ ] Test failures investigated
- [ ] Docker build logs analyzed
- [ ] Dependency versions compatible
- [ ] Go version compatible
- [ ] CGO dependencies installed
- [ ] Build cache cleaned

## Metrics to Monitor Post-Deployment

### Availability Metrics

```
- Uptime percentage (target: 99.9%)
- Health check pass rate (target: 100%)
- Pod crash rate (target: 0%)
- Service availability (target: 99.9%)
```

### Performance Metrics

```
- API response time p50 (target: <50ms)
- API response time p95 (target: <200ms)
- API response time p99 (target: <500ms)
- Database query time (target: <100ms)
- K8s API call time (target: <500ms)
```

### Error Metrics

```
- Error rate (target: <0.1%)
- 5xx error rate (target: <0.01%)
- Timeout rate (target: <0.01%)
- Panic rate (target: 0%)
```

### Resource Metrics

```
- CPU usage (target: <70%)
- Memory usage (target: <70%)
- Disk usage (target: <80%)
- Network I/O (monitor trends)
```

## Production Standards Verification

All code must satisfy:

```
Code Quality:
- Golangci-lint: all checks pass
- Go fmt: all files formatted
- Go vet: no issues
- Coverage: >= 70%

Security:
- gosec: no high/critical issues
- trufflehog: no secrets found
- Dependencies: no known vulnerabilities

Performance:
- API: <200ms p95
- Database: <100ms
- Memory: <512MB per pod
- Startup: <30s

Testing:
- All tests pass
- Race detector clean
- Integration tests pass
- Load tests pass
```

## Sign-Off Process

Before deployment, require sign-off from:

- [ ] **Code Owner**: Reviewed code changes
- [ ] **Security Lead**: Security review passed
- [ ] **QA Lead**: Testing complete
- [ ] **DevOps Lead**: Deployment plan reviewed
- [ ] **Product Manager**: Feature readiness confirmed

## Emergency Rollback

If deployment issues occur:

1. **Immediate Actions** (< 5 minutes)
   - [ ] Alert team immediately
   - [ ] Stop deployment if in progress
   - [ ] Assess impact scope
   - [ ] Decide rollback or fix forward

2. **Rollback Execution** (< 30 minutes)
   - [ ] Execute rollback procedure
   - [ ] Verify previous version healthy
   - [ ] Monitor metrics return to normal
   - [ ] Document incident

3. **Post-Incident** (< 24 hours)
   - [ ] Root cause analysis
   - [ ] Prevention steps documented
   - [ ] Team retro/learning session
   - [ ] Updates to deployment procedure

---

**Note**: This checklist is comprehensive. Not all items apply to every release.
Customize based on your risk profile and service criticality.
