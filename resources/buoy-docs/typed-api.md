---
title: "workspace typed API"
summary: "Source-derived public signatures, properties, parameters, defaults, and return values."
category: "Reference"
order: 21
types: ["WorkspaceException"]
dependencies: []
symbols: ["WorkspaceException"]
related: ["api.md", "getting-started.md", "index.md"]
---

# workspace typed API

Generated from the package source declarations. Platform guards are shown where present; inherited members are documented on the base type. See the [API contracts](api.md) for behavior and limitations.

## Workspace functions

### WorkspaceSupported

`Public Function WorkspaceSupported() As Boolean`

No parameters.

#### Returns

| Type |
| --- |
| `Boolean` |

### ValidateExternalURL

`Public Sub ValidateExternalURL(value As String, allowCustomScheme As Boolean = False)`

Validation only. Never opens another app.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `value` | `String` | `ByVal` | Required |
| `allowCustomScheme` | `Boolean` | `ByVal` | `False` |

Returns: no value.

### OpenExternalURL

`Public Function OpenExternalURL(value As String, allowCustomScheme As Boolean = False) As Boolean`

Submits externally; may block/prompt. True means native acceptance, not content loaded. macOS: main thread. Linux: worker. Windows: STA worker; keep the caller alive.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `value` | `String` | `ByVal` | Required |
| `allowCustomScheme` | `Boolean` | `ByVal` | `False` |

#### Returns

| Type |
| --- |
| `Boolean` |

### OpenFile

`Public Function OpenFile(file As FileSystemItem) As Boolean`

Submits externally; may block/prompt. Opens only an existing file or folder. macOS: main thread. Linux: worker. Windows: STA worker. Native acceptance only.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `file` | `FileSystemItem` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Boolean` |

### RevealFiles

`Public Sub RevealFiles(files As Array Of FileSystemItem)`

Submits externally; may block. Validates all inputs before any reveal request. macOS: main thread. Linux: worker; D-Bus call timeout 5s (bus acquisition unbounded). Windows: STA worker; groups binary-identical parent PIDLs into selection requests.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `files` | `Array Of FileSystemItem` | `ByVal` | Required |

Returns: no value.

### DefaultApplication

`Public Function DefaultApplication(value As String, allowCustomScheme As Boolean = False) As Optional Of String`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `value` | `String` | `ByVal` | Required |
| `allowCustomScheme` | `Boolean` | `ByVal` | `False` |

#### Returns

| Type |
| --- |
| `Optional Of String` |

### FindApplication

`Public Function FindApplication(bundleIdentifier As String) As Optional Of String`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `bundleIdentifier` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Optional Of String` |

### OpenAndRevealSupported

`Public Function OpenAndRevealSupported() As Boolean`

Read-only compile-target predicate. Does not load libraries, connect or launch.

No parameters.

#### Returns

| Type |
| --- |
| `Boolean` |

### OpenAndRevealAvailable

`Public Function OpenAndRevealAvailable() As Boolean`

Read-only library probe; does not connect to D-Bus or check for a handler.

No parameters.

#### Returns

| Type |
| --- |
| `Boolean` |

## WorkspaceException

`Public Class WorkspaceException Inherits RuntimeException`

Native Linux/Windows operation failure. Input errors use InvalidArgumentException.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `Domain` | `String` | Yes | No | — | — |
| `NativeCode` | `Integer` | Yes | No | — | — |
| `Operation` | `String` | Yes | No | — | — |

### Constructor

`Public Sub Constructor(operation As String, domain As String, code As Integer)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `operation` | `String` | `ByVal` | Required |
| `domain` | `String` | `ByVal` | Required |
| `code` | `Integer` | `ByVal` | Required |

Returns: no value.
