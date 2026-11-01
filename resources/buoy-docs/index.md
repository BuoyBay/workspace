---
title: "Workspace integration"
summary: "Validate external URLs and use native application and Finder services."
category: "Overview"
order: 0
keywords: ["workspace", "urls", "finder", "applications", "launch services"]
related: ["getting-started.md", "api.md", "typed-api.md"]
---

# Workspace integration

The `workspace` package validates external URLs without side effects and opens
or reveals user-chosen items using NSWorkspace on macOS, GIO/FileManager1 on Linux,
and the native Shell APIs on Windows. Application-path lookup remains macOS-only.

`WorkspaceSupported()` retains its macOS-only meaning. Portable callers use
`OpenAndRevealSupported()` and the library-only `OpenAndRevealAvailable()` probe.
Read the [API contracts](api.md) for URL policy, errors, thread requirements,
resource cleanup and native acceptance semantics. macOS bodies are unchanged.

## Read next

- [Getting started](getting-started.md)
- [API reference](api.md)
