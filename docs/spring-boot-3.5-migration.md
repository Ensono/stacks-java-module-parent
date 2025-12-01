# Spring Boot 3.5.x Migration - Parent POM Requirements

## Overview

The `stacks-modules-parent:3.0.98` brings in Spring Boot 3.5.7, which introduces several breaking changes and compatibility issues that need to be addressed either in the parent POM or in downstream projects.

## Issues Identified

### 1. Spring Cloud Version Incompatibility

**Problem:**  
The current Spring Cloud version (`2022.0.4`) is incompatible with Spring Boot 3.5.7.

```text
Spring Boot [3.5.7] is not compatible with this Spring Cloud release train.
Change Spring Boot version to one of the following versions [3.0.x, 3.1.x].
```

**Required Fix in Parent POM:**  
Update `spring.cloud.dependencies.version` to a version compatible with Spring Boot 3.5.x:

| Spring Boot Version | Compatible Spring Cloud Version |
|---------------------|---------------------------------|
| 3.0.x, 3.1.x        | 2022.0.x (Kilburn)              |
| 3.2.x               | 2023.0.x (Leyton)               |
| 3.3.x, 3.4.x        | 2024.0.x                        |
| 3.5.x               | 2025.0.x (when available)       |

**Workaround (current):**  
Projects can disable the compatibility verifier in `application-test.yml`:

```yaml
spring:
  cloud:
    compatibility-verifier:
      enabled: false
```

**Action Required:** Update parent POM to use Spring Cloud 2024.0.x or later (once 2025.0.x is available for Spring Boot 3.5.x support).

---

### 2. Spring Security Filter Chain Conflict

**Problem:**  
Spring Boot 3.5.x has stricter validation for Spring Security filter chains. Multiple `SecurityFilterChain` beans matching "any request" now throw an error:

```text
UnreachableFilterChainException: A filter chain that matches any request
[...ApplicationConfig...] has already been configured, which means that this
filter chain [...ApplicationNoSecurity...] will never get invoked.
```

**Required Fix:**  
Security configurations with overlapping matchers must be mutually exclusive. If using profile-based security configurations:

```java
// Production security config - exclude from test profile
@Configuration
@EnableWebSecurity
@Profile("!test")  // Add this annotation
public class ApplicationConfig {
    // ...
}

// Test security config - only active in test profile
@Configuration
@EnableWebSecurity
@Profile("test")
public class ApplicationNoSecurity {
    // ...
}
```

**Action Required:** This is a downstream project fix, but parent POM documentation should note this breaking change.

---

### 3. Bean Resolution Changes for Inheritance Hierarchies

**Problem:**  
Spring Boot 3.5.x has stricter bean resolution when multiple beans of the same type exist through inheritance:

```text
NoUniqueBeanDefinitionException: expected single matching bean but found 2:
menuService, menuServiceV2
```

**Required Fix:**  
When a service class is extended, the base class should be marked as `@Primary`:

```java
@Service
@Primary  // Add this annotation
public class MenuService {
    // ...
}

@Service
public class MenuServiceV2 extends MenuService {
    // ...
}
```

**Action Required:** This is a downstream project fix, but parent POM documentation should note this breaking change.

---

### 4. Maven Resource Filtering

**Problem:**  
Property placeholders like `@aws.profile.name@` in `application.yml` are not being replaced because Maven resource filtering is not enabled by default.

```text
Profile '@aws.profile.name@' must start and end with a letter or digit
```

**Required Fix in Projects:**  
Enable resource filtering in `pom.xml`:

```xml
<build>
  <resources>
    <resource>
      <directory>src/main/resources</directory>
      <filtering>true</filtering>
    </resource>
  </resources>
  <testResources>
    <testResource>
      <directory>src/test/resources</directory>
      <filtering>true</filtering>
    </testResource>
  </testResources>
</build>
```

**Action Required:** Consider adding this configuration to the parent POM so all child projects inherit it.

---

## Summary of Parent POM Changes Required

### Critical (Must Fix) ✅ COMPLETED

1. **Update Spring Cloud version** to be compatible with Spring Boot 3.5.x
   - ~~Current: `2022.0.4`~~
   - ✅ Updated to: `2025.0.0` (compatible with Spring Boot 3.5.x)
   - Added `spring.cloud.dependencies.version` property
   - Added Spring Cloud BOM to `dependencyManagement`

### Recommended (Should Add) ✅ COMPLETED

1. **Add default resource filtering configuration** so child projects don't need to configure it manually
   - ✅ Added `<resources>` and `<testResources>` configuration with `<filtering>true</filtering>`

2. **Update documentation** to note the following breaking changes for downstream projects:
   - Security filter chain mutual exclusivity requirements
   - Bean resolution changes for inheritance hierarchies
   - Profile annotation requirements for conditional configurations

### Temporary Workarounds Applied in stacks-java

After updating to the new parent POM version, some workarounds may no longer be needed:

| Issue                          | Workaround                      | File                                      |
|--------------------------------|---------------------------------|-------------------------------------------|
| Spring Cloud incompatibility   | Disabled compatibility verifier | `src/test/resources/application-test.yml` |
| Security filter chain conflict | Added `@Profile("!test")`       | `ApplicationConfig.java`                  |
| Bean resolution conflict       | Added `@Primary`                | `MenuService.java`                        |
| Resource filtering             | Added filtering config          | `pom.xml`                                 |

## Testing Verification

After parent POM updates, verify with:

```bash
cd java
./mvnw clean test
./mvnw clean verify -P owasp-dependency-check
```

## References

- [Spring Cloud Release Train Compatibility](https://spring.io/projects/spring-cloud#overview)
- [Spring Boot 3.5 Release Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.5-Release-Notes)
- [Spring Security Filter Chain Configuration](https://docs.spring.io/spring-security/reference/servlet/configuration/java.html)
