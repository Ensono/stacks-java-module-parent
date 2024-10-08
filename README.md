# Stacks parent module

## Module Overview

This parent module provides shared functionality to the Stacks java modules. It's based on Spring
and contains the dependency management, shared properties, plugins, profiles, reporting and repositories.
It's intended to be the parent module for Ensono Stacks but can also get used in your project.

## Module Structure

In the following diagram, you can see all the relevant files of this module. Be aware, pulling from
the repository will have some extra files that are not relevant to the logic but required to build and
deploy.

### Project structure

```
java
|_pom.xml
```

## How to use

Use it as a dependency.

#### Maven

In the `parent` section of your application's `pom.xml` add:

```xml
<dependency>
    <groupId>com.amido.stacks.modules</groupId>
    <artifactId>stacks-modules-parent</artifactId>
    <version>1.0.1</version>
    <type>pom</type>
</dependency>
```

NOTE: You should check to see the latest version and use that.

Then you can do a `./mvnw clean compile` to fetch it; after that, you can use it like any other dependency.

```bash
./mvnw clean compile
```

### Building the module locally from this repository

To build the module locally:

1.  Clone this repo
3.  run `./mvnw clean install` to install the module locally.
4.  Add it as any other [dependency](#use-it-as-a-dependency)
