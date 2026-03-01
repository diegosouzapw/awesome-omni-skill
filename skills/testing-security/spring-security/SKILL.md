---
name: spring-security
description: Spring Security 6 patterns for authentication, authorization, and OAuth2
keywords:
  - spring-security
  - security
  - authentication
  - authorization
  - oauth2
  - jwt
  - filter
  - csrf
  - method security
filePatterns:
  - "*.java"
  - "application.yml"
  - "application.properties"
frameworks:
  - spring-security
  - spring-boot
tokenCount: 3000
version: 1.0.0
---

# Spring Security Patterns

> Spring Security 6 patterns for securing APIs.
> Favor explicit security rules and least privilege.

## Selective Reading Rule

Read only files relevant to the request. Use the content map to focus.

## Content Map

| File | Description | When to Read |
|------|-------------|--------------|
| `http-security.md` | SecurityFilterChain, HttpSecurity | Basic config |
| `jwt.md` | JWT auth, filters | Token-based APIs |
| `oauth2.md` | OAuth2 login/resource server | OAuth flows |
| `method-security.md` | @PreAuthorize, @Secured | Method-level rules |

## Core Patterns

### 1. SecurityFilterChain

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    return http
        .csrf(csrf -> csrf.disable())
        .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/actuator/**").permitAll()
            .requestMatchers(HttpMethod.POST, "/api/v1/auth/**").permitAll()
            .anyRequest().authenticated()
        )
        .build();
}
```

### 2. JWT Authentication Filter

```java
public class JwtAuthFilter extends OncePerRequestFilter {

    private final JwtService jwtService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws IOException, ServletException {
        String token = resolveToken(request);
        if (token != null && jwtService.isValid(token)) {
            Authentication auth = jwtService.getAuthentication(token);
            SecurityContextHolder.getContext().setAuthentication(auth);
        }
        filterChain.doFilter(request, response);
    }
}
```

### 3. Method Security

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(String id) {
    userRepository.deleteById(id);
}
```

## Decision Checklist

- Stateless API? -> JWT + stateless session
- Admin-only endpoints? -> @PreAuthorize
- Public endpoints whitelisted?
- Secrets stored outside code?

## Anti-Patterns

| Anti-Pattern | Why Bad | Better Approach |
|--------------|---------|-----------------|
| PermitAll on broad paths | Security holes | Explicit allow list |
| Storing secrets in code | Leaks | Env variables or vault |
| Mixing auth and business logic | Hard to maintain | Dedicated filters/services |

## Related Skills

| Need | Skill |
|------|-------|
| Core Spring patterns | `@[skills/spring-boot-patterns]` |
| Testing | `@[skills/spring-testing]` |
