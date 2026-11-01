---
title: "Workspace integration API"
summary: "Public symbols and contracts for the workspace package."
category: "Reference"
order: 20
keywords: ["workspace", "urls", "finder", "applications", "launch services"]
symbols: ["WorkspaceSupported", "ValidateExternalURL", "OpenExternalURL", "OpenFile", "RevealFiles", "DefaultApplication", "FindApplication"]
related: ["index.md", "getting-started.md", "typed-api.md"]
---

# API reference

`Import workspace` provides native open/reveal integration on macOS, Linux and
Windows. Existing macOS signatures and implementation bodies are preserved.

| API | Contract |
| --- | --- |
| `WorkspaceSupported()` | Frozen compatibility flag: **macOS only**. |
| `OpenAndRevealSupported()` | Backend implemented for this target: macOS, Linux or Windows. No native calls. |
| `OpenAndRevealAvailable()` | Library-load probe only on Linux/Windows; true on macOS. Does not connect, prompt, check handlers or prove a desktop service is running. |
| `ValidateExternalURL(value, allowCustomScheme = False)` | Validation only. No launch, file access, DNS or connection. |
| `OpenExternalURL(value, allowCustomScheme = False)` | Validate, then ask the registered handler to open the URL. True means native request acceptance. |
| `OpenFile(file)` | Open an existing file/folder through its native handler. File paths never enter URL validation. |
| `RevealFiles(files)` | Validate the entire nonempty batch before sending reveal requests. |
| `DefaultApplication(value, allowCustomScheme = False)` | **macOS only**: optional application bundle path. |
| `FindApplication(bundleIdentifier)` | **macOS only**: optional installed application bundle path. |

No equivalent application-path lookup is claimed on Linux/Windows: a desktop
entry, protocol ProgID or executable command is not the macOS bundle-path contract.
Both lookup functions retain their existing unsupported exception off macOS.

## URL and file policy

HTTP, HTTPS, mailto and tel are allowed by default. Other schemes require explicit
opt-in; file, javascript, data and about remain forbidden. Relative URLs, embedded
authority credentials, controls and hostless HTTP(S) are rejected. Leading and
trailing whitespace is trimmed only after raw control checks. The same validator
is used by validation and opening on each target.

Linux and Windows share a pure parser and additionally reject malformed escapes,
escaped ASCII controls, raw whitespace/backslashes, invalid ports/IP literals,
ambiguous authority escapes and one-letter schemes (drive-path ambiguity).
Bracketed IPv6 accepts ordinary and IPv4-mapped addresses; zone identifiers and
IPvFuture are outside this parser's accepted subset. The existing macOS NSURL
parser remains unchanged and can differ on these additional syntax cases.
Unicode destinations are allowed; the receiving handler determines their meaning.

Empty/Nil files, embedded NULs still present in strings, nonexistent files and
inaccessible paths are rejected. Windows uses wide native path resolution and
attribute inspection, preserving non-BMP filenames. Relative paths depend on the
process working directory; avoid changing it concurrently. Existence checks do
not eliminate races or bypass permissions. **BUI-003 limitation:** the current
Buoy runtime can discard NUL before a String reaches this API; the validator
cannot recover discarded input. The receipt explicitly records that test gap.

Only open items the user chose: native file handlers can execute application
bundles, executables or automation documents. The package constructs no shell
commands or command lines, and passes no launch parameters. It does not promise
that a remote page loaded, a handler processed the request, or selection remained
visible after another application changed it.

## Native behavior, threads and cleanup

- **macOS:** unchanged NSWorkspace/NSURL behavior; use the main thread. Open can
  return false. Reveal is the existing void native request.
- **Linux:** GIO `g_app_info_launch_default_for_uri` opens URLs and file URIs;
  `GFile` constructs escaped file URIs. Reveal sends `ShowItems(as URIs, s
  StartupId)` to `org.freedesktop.FileManager1`, with an empty startup id and a
  five-second method timeout. This can activate the installed file manager.
  Calls may block: use a worker. Bus discovery, connection and cleanup have no
  end-to-end deadline. A private D-Bus connection is closed and unreferenced per
  reveal; no shared exit-on-close connection is modified. GObjects, GVariants,
  builders, allocated strings and GErrors are released. GIO's synchronous launch
  has an application-lifetime caveat for D-Bus-activated handlers: do not terminate
  the caller immediately after acceptance.
- **Windows:** `ShellExecuteExW` uses the `open` verb, `SEE_MASK_NOASYNC` and
  `SEE_MASK_FLAG_NO_UI`. This suppresses ordinary error dialogs, not OS security
  prompts. No process handle is requested. Use an STA-compatible worker thread;
  each operation balances successful `CoInitializeEx` (including S_FALSE) with
  `CoUninitialize`. An existing incompatible MTA produces an HRESULT error;
  the package does not change its apartment. Reveal parses all absolute PIDLs
  first, then groups binary-identical parent PIDLs for `SHOpenFolderAndSelectItems`
  with the corresponding borrowed child IDs. Equivalent paths with distinct
  native PIDL representations can still cause separate requests. Roots without
  a removable child use the zero-child form. Original and cloned PIDLs are
  freed with `CoTaskMemFree` (the Shell task allocator on supported Windows). Partial native acceptance cannot be rolled back.

Invalid input raises `InvalidArgumentException`. Linux/Windows native failures
raise `WorkspaceException` with read-only `Operation`, `Domain` and `NativeCode`;
messages omit supplied URLs/paths. Domains are GLib quark strings, `Win32`,
`HRESULT` or `buoy-workspace` for package-detected invariant failures. Missing
optional libraries use Buoy's `LibraryUnavailableException`. Windows missing or
inaccessible files retain their native Win32 codes; Linux existence rejection
uses `InvalidArgumentException` because FileSystemItem cannot distinguish causes.

Native buffers use documented 64-bit little-endian ABI layouts (Linux/Windows
x64 and ARM64). The Windows fixture compiler checks `SHELLEXECUTEINFOW` size and
field offsets against platform headers. Private byte-buffer helpers avoid
exporting ABI structures through Buoy's non-public-import visibility limitation;
the narrow repro is retained for parent triage. No public pointer API is exposed.


[Full typed API](typed-api.md) · [Getting started](getting-started.md)


## Custom-scheme and input limitations

`allowCustomScheme = True` is an explicit request to accept other syntactically
valid schemes; callers must allowlist the exact scheme they intend to handle.
This can include operating-system actions such as `shell:` or `search-ms:`.
Validation does not establish destination trust, DNS validity or hostname
existence, and invisible Unicode characters remain permitted. Buoy 0.72 may drop
NUL or truncate a suffix before this API receives the String (BUI-003); input
already lost at that boundary cannot be checked here.
