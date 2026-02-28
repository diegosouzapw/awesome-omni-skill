---
name: keycloak
description: Keycloak identity and access management. Use for SSO.
---

# Keycloak

Keycloak is an open-source Identity and Access Management solution aimed at modern applications and services. It makes it easy to secure applications and services with little to no code.

## When to Use

- **Self-Hosted IAM**: You want Auth0 features but deployed on your own infrastructure (GDPR/Compliance).
- **Enterprise Integration**: Connecting to legacy LDAP/Active Directory user federations.
- **Single Sign-On (SSO)**: One login for your internal wiki, chat, and cloud apps.

## Quick Start (Docker)

```bash
docker run -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:latest start-dev
```

## Core Concepts

### Realm

A space where you manage objects (users, apps, roles). You usually create a dedicated realm for your app (e.g., `my-app-realm`) and leave `master` for admin tasks.

### Clients

Applications (Web, Mobile, Service) that can request login.

### Identity Brokering

Keycloak can act as a broker: User clicks "Login with GitHub" -> Keycloak talks to GitHub -> Keycloak issues its own token to your app.

## Best Practices (2025)

**Do**:

- **Use the Operator**: On Kubernetes, use the Keycloak Operator for upgrades and scaling.
- **Production Mode**: `start-dev` is for local only. Use an external DB (Postgres) and proper HTTPS for production.
- **Theme It**: Don't use the default login page. Extend the theme to match your brand.

**Don't**:

- **Don't Modify Core**: Use the SPI (Service Provider Interface) to write plugins if you need custom logic.
- **Don't expose Admin Console**: Block `/admin` and `/master` access from the public internet.

## References

- [Keycloak Documentation](https://www.keycloak.org/documentation)
