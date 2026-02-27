---
name: spring-boot-migrator
description: Automated migration of Spring Boot 2.x to 3.x with JDK 8 to 21 upgrade using OpenRewrite. Use when asked to migrate, upgrade, or modernize Spring Boot applications, upgrade from JDK 8/11/17 to JDK 21, migrate from javax to jakarta namespace, or when facing Spring Boot 2 EOL migration tasks. Handles Maven projects with standard or custom parent POMs.
---

# Spring Boot Migrator

Automates Spring Boot 2.x → 3.x migration with JDK 8 → 21 upgrade using OpenRewrite.

## What This Skill Does

- ✅ Automated Spring Boot 2 → 3 upgrade (up to 3.5.x)
- ✅ JDK 8 → 21 migration
- ✅ javax.* → jakarta.* namespace migration
- ✅ Hibernate 5 → 6 adaptation
- ✅ Configuration property updates
- ✅ Dependency version resolution
- ✅ Full validation and testing workflow

## Prerequisites Check

Before starting, verify:
```bash
java -version    # Need JDK 17+ to run OpenRewrite
mvn -version     # Need Maven 3.8.1+
```

If missing, guide user to install or offer to create installation script.

## Workflow

### 1. Quick Assessment

Ask user for project path, then analyze:

```bash
cd <project-path>
# Check if Maven project
test -f pom.xml || { echo "Not a Maven project"; exit 1; }

# Extract key info
grep -oP '(?<=<spring-boot.version>)[^<]+' pom.xml
grep -oP '(?<=<java.version>)[^<]+' pom.xml
grep -c "<parent>" pom.xml
```

Confirm with user:
- Current Spring Boot version
- Current Java version
- Parent POM situation (spring-boot-starter-parent vs custom)
- Kotlin detected? (warn about limited support)

### 2. Pre-Migration Backup

```bash
# Create backup branch
git checkout -b backup-before-sb3-$(date +%Y%m%d)
git add -A && git commit -m "Backup before Spring Boot 3 migration" --allow-empty

# Record baseline
mkdir .migration-validation
mvn dependency:tree > .migration-validation/deps-before.txt
mvn clean test > .migration-validation/test-before.txt 2>&1
```

### 3. Configure OpenRewrite

Add to `pom.xml` in `<build><plugins>`:

```xml
<plugin>
    <groupId>org.openrewrite.maven</groupId>
    <artifactId>rewrite-maven-plugin</artifactId>
    <version>5.47.0</version>
    <configuration>
        <activeRecipes>
            <recipe>org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_3</recipe>
            <recipe>org.openrewrite.java.migrate.UpgradeToJava21</recipe>
            <recipe>org.openrewrite.java.migrate.jakarta.JavaxToJakarta</recipe>
        </activeRecipes>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>org.openrewrite.recipe</groupId>
            <artifactId>rewrite-spring</artifactId>
            <version>5.22.0</version>
        </dependency>
        <dependency>
            <groupId>org.openrewrite.recipe</groupId>
            <artifactId>rewrite-migrate-java</artifactId>
            <version>2.28.0</version>
        </dependency>
    </dependencies>
</plugin>
```

**For custom parent POM projects**: See [references/custom-parent-strategy.md](references/custom-parent-strategy.md)

### 4. Preview Changes

```bash
# Discover applicable recipes
mvn rewrite:discover

# Dry run (preview only)
mvn rewrite:dryRun | tee .migration-validation/dryrun.txt
```

Show user the affected files. Confirm before proceeding.

### 5. Apply Migration

```bash
# Apply changes
mvn rewrite:run | tee .migration-validation/rewrite.txt

# Show what changed
git diff --stat
git diff > .migration-validation/changes.diff
```

### 6. Fix Compilation Issues

```bash
mvn clean compile 2>&1 | tee .migration-validation/compile-after.txt
```

**Common issues and fixes**: See [references/common-fixes.md](references/common-fixes.md)

Auto-detect and suggest fixes for:
- `package javax` not found → Check third-party deps for Jakarta versions
- `PostgreSQL10Dialect` missing → Use generic `PostgreSQLDialect`
- `server.max-http-header-size` → Rename to `server.max-http-request-header-size`

Max 3 retry attempts. If still failing after fixes, provide detailed troubleshooting guide.

### 7. Validate Migration

```bash
# Run tests
mvn test | tee .migration-validation/test-after.txt

# Compare dependencies
mvn dependency:tree > .migration-validation/deps-after.txt
diff .migration-validation/deps-before.txt .migration-validation/deps-after.txt > .migration-validation/deps-diff.txt
```

Generate migration report in `.migration-validation/REPORT.md` with:
- Before/after comparison
- Test results
- Dependency changes
- Manual follow-up items

## Custom Parent POM Handling

For projects with custom parent POMs managing Spring Boot and internal library versions:

**Option 1**: Update parent POM first (if maintainable)
**Option 2**: Switch to BOM import pattern (recommended)

See [references/custom-parent-strategy.md](references/custom-parent-strategy.md) for detailed implementation.

## Manual Follow-Up Checklist

After successful migration, remind user to:

- [ ] Review `.migration-validation/REPORT.md`
- [ ] Check `application.properties/yml` for deprecated properties
- [ ] Update CI/CD to use JDK 21
- [ ] Test critical business flows manually
- [ ] Review third-party library versions for Jakarta compatibility
- [ ] Consider enabling Virtual Threads (JDK 21): `spring.threads.virtual.enabled=true`
- [ ] Update Docker base images to JDK 21

## Troubleshooting

**OpenRewrite fails to run**:
```bash
# Clear Maven cache
rm -rf ~/.m2/repository/org/openrewrite
mvn rewrite:discover
```

**Recipe not found**:
- Check plugin dependency versions are latest
- Verify recipe names with `mvn rewrite:discover`

**Compilation fails after 3 attempts**:
- Save all logs to `.migration-validation/`
- Point user to [references/common-fixes.md](references/common-fixes.md)
- Offer to search Stack Overflow for specific error patterns

## Scripts

Use `scripts/migrate.sh` for fully automated execution when user wants hands-off migration:

```bash
# From skill directory
./scripts/migrate.sh <project-path>
```

The script runs all 10 steps automatically with interactive confirmations at key points.

## References

- **Custom Parent POM Strategy**: [references/custom-parent-strategy.md](references/custom-parent-strategy.md)
- **Common Fixes**: [references/common-fixes.md](references/common-fixes.md)
- **Detailed Research**: [references/research.md](references/research.md) - Official tools, migration notes, breaking changes

## Success Criteria

Migration is complete when:
- ✅ `mvn clean compile` succeeds
- ✅ `mvn test` passes (or user acknowledges expected failures)
- ✅ Migration report generated
- ✅ User informed of manual follow-up items
