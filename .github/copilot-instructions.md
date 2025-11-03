# Copilot Instructions for Stacks Java Module Parent

> **Security Note**: All contributors must follow the security and compliance guidelines defined in [copilot-security-instructions.md](./copilot-security-instructions.md).

## Project Overview

This is a **Maven parent POM** for Ensono Stacks Java modules, not a standalone application. It defines shared dependencies, plugins, and build configuration for downstream Spring Boot projects.

## Key Architecture Concepts

### Maven Parent POM Pattern

- **Purpose**: Centralized dependency management and build configuration
- **Packaging**: `<packaging>pom</packaging>` - no source code, only configuration
- **Consumption**: Other projects inherit from this via `<parent>` section
- **Versioning**: Uses `0.0.0-PR` as placeholder - replaced by CI/CD pipeline

### Dependency Strategy

- **Core Dependencies**: Automatically inherited by all child modules (Lombok, Logback, JUnit, Mockito)
- **Managed Dependencies**: Available via `<dependencyManagement>` but must be explicitly declared by child modules
- **Spring Boot BOM**: Imported via `spring-boot-dependencies` for version consistency

## Essential Build Commands

```bash
# Install parent POM locally for development
./mvnw clean install

# Format code using Google Java Style
./mvnw fmt:format

# Run security vulnerability scan
./mvnw -P owasp-dependency-check clean verify

# Run mutation testing with PIT
./mvnw pitest:mutationCoverage

# Run all quality checks (checkstyle, spotbugs, tests)
./mvnw clean verify
```

## Code Quality Standards

### Automatic Code Formatting

- **Tool**: Google Java Format via `fmt-maven-plugin`
- **Style**: Google Java Style Guide
- **Auto-format**: Runs on every build
- **Pattern**: `.*\.java` files only

### Testing Strategy

- **Unit Tests**: JUnit 5 + Mockito + AssertJ + Hamcrest
- **Integration Tests**: Maven Failsafe plugin
- **Mutation Testing**: PIT with STRONGER mutators
- **Coverage**: JaCoCo with minimal 0% threshold (configurable per project)

### Static Analysis Stack

- **Checkstyle**: Google checks (`google_checks.xml`)
- **SpotBugs**: Bug pattern detection
- **OWASP**: Dependency vulnerability scanning

## CI/CD Integration

### Azure DevOps Pipeline

- **Template Repo**: `Ensono/stacks-pipeline-templates`
- **Container Images**: Custom Java build containers (`ensono/eir-java`)
- **Versioning**: `major.minor.buildnumber` pattern
- **Profiles**: `test` (default), `local`, `owasp-dependency-check`

### Publishing

- **Target**: Maven Central via Sonatype Central Publishing
- **Artifacts**: JAR, sources, javadocs (all signed with GPG)
- **Auto-publish**: Enabled in pipeline

## Development Patterns

### Annotation Processing

Configured for Lombok + MapStruct integration:

```xml
<annotationProcessorPaths>
  <path>lombok</path>
  <path>lombok-mapstruct-binding</path>
  <path>mapstruct-processor</path>
</annotationProcessorPaths>
```

### Security Updates

- **Dependabot**: Weekly Monday updates, max 50 PRs
- **Overrides**: Explicit versions for security-sensitive dependencies (Gson, Netty, etc.)
- **Exclusions**: Netty codec-http excluded from reactor-netty-core

## When Modifying This Parent POM

1. **Version Updates**: Update dependency versions in `<properties>` section
2. **New Dependencies**: Add to `<dependencyManagement>` for optional use, or `<dependencies>` for automatic inheritance
3. **Plugin Configuration**: Modify in `<pluginManagement>` for optional use, or `<plugins>` for automatic inheritance
4. **Testing Changes**: Verify with `./mvnw clean install` before committing
5. **Security**: Always run OWASP check after dependency updates

## Common Issues

- **Lombok + MapStruct**: Requires specific annotation processor ordering (already configured)
- **Netty Conflicts**: Reactor Netty exclusions prevent version conflicts
- **Test Execution**: Surefire runs in `test` phase only, Failsafe handles integration tests
- **Code Formatting**: Must pass fmt check - run `./mvnw fmt:format` if build fails
