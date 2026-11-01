# Workspace · 0.1.0

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

## Validation


Maintainers run the private validation suite before release. See the repository README for public build instructions.

## Primary API references

- [NSWorkspace](https://developer.apple.com/documentation/appkit/nsworkspace)
- [GIO launch](https://docs.gtk.org/gio/type_func.AppInfo.launch_default_for_uri.html),
  [GFile URI](https://docs.gtk.org/gio/method.File.get_uri.html),
  [FileManager1](https://wiki.freedesktop.org/www/Specifications/file-manager-interface/)
- [D-Bus address](https://docs.gtk.org/gio/func.dbus_address_get_for_bus_sync.html),
  [private connection](https://docs.gtk.org/gio/ctor.DBusConnection.new_for_address_sync.html),
  [synchronous call](https://docs.gtk.org/gio/method.DBusConnection.call_sync.html),
  [close](https://docs.gtk.org/gio/method.DBusConnection.close_sync.html),
  [variant tuple ownership](https://docs.gtk.org/glib/ctor.Variant.new_tuple.html)
- [ShellExecuteExW](https://learn.microsoft.com/en-us/windows/win32/api/shellapi/nf-shellapi-shellexecuteexw),
  [SHELLEXECUTEINFOW layout](https://learn.microsoft.com/en-us/windows/win32/api/shellapi/ns-shellapi-shellexecuteinfow),
  [SHParseDisplayName](https://learn.microsoft.com/en-us/windows/win32/api/shlobj_core/nf-shlobj_core-shparsedisplayname),
  [SHOpenFolderAndSelectItems](https://learn.microsoft.com/en-us/windows/win32/api/shlobj_core/nf-shlobj_core-shopenfolderandselectitems),
  [COM initialization](https://learn.microsoft.com/en-us/windows/win32/api/combaseapi/nf-combaseapi-coinitializeex),
  [GetFullPathNameW](https://learn.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-getfullpathnamew),
  [GetFileAttributesW](https://learn.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-getfileattributesw)

- Windows PIDL ownership/grouping: [ILClone](https://learn.microsoft.com/en-us/windows/win32/api/shlobj_core/nf-shlobj_core-ilclone),
  [ILRemoveLastID](https://learn.microsoft.com/en-us/windows/win32/api/shlobj_core/nf-shlobj_core-ilremovelastid),
  [ILFindLastID](https://learn.microsoft.com/en-us/windows/win32/api/shlobj_core/nf-shlobj_core-ilfindlastid),
  [ILIsEqual](https://learn.microsoft.com/en-us/windows/win32/api/shlobj_core/nf-shlobj_core-ilisequal),
  [task allocator cleanup](https://learn.microsoft.com/en-us/windows/win32/api/shlobj_core/nf-shlobj_core-ilfree).


### Caller policy and validation limits

Custom-scheme opt-in accepts syntactically valid schemes such as `shell:`,
`search-ms:` and `ms-msdt:`. Callers must allowlist the exact scheme they intend
to handle before opting in. Validation is not a destination trust, DNS or
hostname-existence check. Invisible Unicode characters are permitted; applications
must apply any display/spoofing policy their workflow requires.

Buoy 0.72 can discard an embedded NUL and truncate the remaining suffix before
validation, or drop the NUL while constructing a String (BUI-003). The validator
can only inspect the resulting String and cannot recover the original input.
Windows file opening requests the explicit `open` verb; types that offer only
another default action can fail. Default application and bundle lookup remain
macOS-only read-only operations; they do not launch an application.
