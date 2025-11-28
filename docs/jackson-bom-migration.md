# Jackson BOM Migration Guide for stacks-modules-parent

## Overview

Starting with Jackson 2.20, the versioning scheme for `jackson-annotations` diverged from other Jackson modules. This document explains how to properly configure Jackson dependencies in `stacks-modules-parent` to avoid dependency resolution failures.

## The Problem

Jackson 2.20 introduced a versioning change:

| Module                | Version Format | Example                         |
|-----------------------|----------------|---------------------------------|
| `jackson-core`        | `2.20.x`       | `2.20.0`, `2.20.1`              |
| `jackson-databind`    | `2.20.x`       | `2.20.0`, `2.20.1`              |
| `jackson-datatype-*`  | `2.20.x`       | `2.20.0`, `2.20.1`              |
| `jackson-annotations` | `2.20`         | `2.20` only (no patch versions) |

If the parent POM defines a single `jackson.version` property (e.g., `2.20` or `2.20.1`) and applies it to all Jackson modules, builds will fail:

```text
Could not find artifact com.fasterxml.jackson.core:jackson-annotations:jar:2.20.1
```

or

```text
Could not find artifact com.fasterxml.jackson.core:jackson-core:jar:2.20
```

## The Solution

### Option 1: Import Jackson BOM (Recommended)

The Jackson BOM (`jackson-bom`) correctly manages all Jackson module versions, including the special case for `jackson-annotations`.

```xml
<properties>
  <jackson-bom.version>2.20.1</jackson-bom.version>
</properties>

<dependencyManagement>
  <dependencies>
    <!-- Jackson BOM must be imported FIRST to take precedence -->
    <dependency>
      <groupId>com.fasterxml.jackson</groupId>
      <artifactId>jackson-bom</artifactId>
      <version>${jackson-bom.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>

    <!-- Other BOMs follow -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-dependencies</artifactId>
      <version>${spring-boot.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

With this approach, you don't need to specify versions on individual Jackson dependencies:

```xml
<dependencies>
  <dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <!-- Version managed by BOM -->
  </dependency>
  <dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <!-- Version managed by BOM -->
  </dependency>
  <dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <!-- Version managed by BOM (correctly uses 2.20, not 2.20.1) -->
  </dependency>
</dependencies>
```

### Option 2: Separate Version Properties

If you cannot use the BOM, define separate version properties:

```xml
<properties>
  <!-- Core Jackson version for most modules -->
  <jackson.version>2.20.1</jackson.version>
  <!-- Annotations uses different versioning from 2.20 onwards -->
  <jackson.version.annotations>2.20</jackson.version.annotations>
</properties>

<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-core</artifactId>
      <version>${jackson.version}</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>${jackson.version}</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-annotations</artifactId>
      <version>${jackson.version.annotations}</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.datatype</groupId>
      <artifactId>jackson-datatype-jsr310</artifactId>
      <version>${jackson.version}</version>
    </dependency>
    <!-- Add other Jackson modules as needed -->
  </dependencies>
</dependencyManagement>
```

## Child Project Overrides

If a child project needs to override Jackson versions from a parent POM that doesn't use the BOM correctly, add these to the child's `pom.xml`:

```xml
<properties>
  <!-- Override Jackson version from parent POM -->
  <jackson-bom.version>2.20.1</jackson-bom.version>
  <jackson.version>2.20.1</jackson.version>
</properties>

<dependencyManagement>
  <dependencies>
    <!-- Import Jackson BOM first -->
    <dependency>
      <groupId>com.fasterxml.jackson</groupId>
      <artifactId>jackson-bom</artifactId>
      <version>${jackson-bom.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
    <!-- Explicit override for annotations (different versioning scheme) -->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-annotations</artifactId>
      <version>2.20</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

## Verification

After making changes, verify the resolved versions:

```bash
# Check effective POM for Jackson versions
./mvnw help:effective-pom | grep -A2 "jackson"

# Verify dependency resolution
./mvnw dependency:tree | grep jackson

# Test full dependency resolution
./mvnw dependency:go-offline
```

Expected output should show:

- `jackson-core:2.20.1`
- `jackson-databind:2.20.1`
- `jackson-annotations:2.20` (note: no patch version)
- `jackson-datatype-jsr310:2.20.1`

## References

- [Jackson BOM on Maven Central](https://repo.maven.apache.org/maven2/com/fasterxml/jackson/jackson-bom/)
- [Jackson 2.20 Release Notes](https://github.com/FasterXML/jackson/wiki/Jackson-Release-2.20)
- [Maven BOM Import Order](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#importing-dependencies)

## Changelog

| Date       | Change                                                    |
|------------|-----------------------------------------------------------|
| 2025-11-28 | Initial documentation for Jackson 2.20 versioning changes |
