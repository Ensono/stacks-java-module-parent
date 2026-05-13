# PIT History Disabled by Default for PIT 1.23.1+

## Overview

`stacks-modules-parent` now leaves PIT mutation history disabled by default.

This change is intentional for compatibility with PIT `1.23.1+`. Recent PIT versions fail when history is enabled without an installed and activated history plugin.

## Root Cause

Previous parent POM defaults configured PIT history directly:

```xml
<historyInputFile>target/pitHistory.txt</historyInputFile>
<historyOutputFile>target/pitHistory.txt</historyOutputFile>
```

With PIT `1.23.1`, that default now triggers the following error when no history plugin is present:

```text
History has been enabled but no history plugin has been installed/activated.
```

Because this configuration lived in the shared parent POM, downstream modules inherited a broken default even when they did not explicitly opt into mutation history.

## Parent POM Change

The default PIT configuration now keeps:

```xml
<configuration>
  <threads>15</threads>
  <timestampedReports>false</timestampedReports>
  <mutators>
    <value>STRONGER</value>
  </mutators>
  <outputFormats>
    <outputFormat>XML</outputFormat>
    <outputFormat>HTML</outputFormat>
  </outputFormats>
</configuration>
```

The parent still uses:

- PIT `1.23.1`
- `pitest-junit5-plugin` integration

The parent no longer enables any history plugin or history feature by default.

## Consumer Impact

If a child project wants PIT mutation history, it must opt in explicitly with compatible history-plugin configuration.

If a child project does not need mutation history, no action is required.

## Why This Is Best Practice

Safe shared-parent default:

- does not require vendor-specific history tooling
- does not enable optional PIT features implicitly
- avoids breaking downstream consumers on PIT `1.23.1+`
- lets child projects choose history only when they intentionally configure it
