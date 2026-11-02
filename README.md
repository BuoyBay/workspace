# Open URLs, reveal files

`Import workspace` provides native open/reveal integration on macOS, Linux and
Windows. Existing macOS signatures and implementation bodies are preserved.

No equivalent application-path lookup is claimed on Linux/Windows: a desktop
entry, protocol ProgID or executable command is not the macOS bundle-path contract.
Both lookup functions retain their existing unsupported exception off macOS.

## Install

From a Buoy consumer project with BPM installed:

```sh
bpm add workspace --git https://github.com/BuoyBay/workspace.git --rev refs/heads/main
```basic

BPM records the resolved commit in its lockfile. Keep that lockfile with the
consumer project for reproducible installs.

## Example

```basic
Import workspace

Sub Main()
    If Not WorkspaceSupported() Then Exit Sub
    ValidateExternalURL("https://example.com")
    Dim handler As Optional Of String = DefaultApplication("https://example.com")
    If handler.HasValue Then
        Print handler.Value
    End If
End Sub
```

## Platform and scope

Uses macOS native APIs, Linux GIO and the Windows Shell. Opening a URL/file is an explicit side effect; support probes do not launch another application.

## Build from source

With the native `build-packages` driver from [devtools](https://github.com/BuoyBay/devtools)
on your `PATH`, run from this checkout:

```sh
build-packages --root "$PWD" --compile-consumer workspace
```

The driver checks source targets and compiles an import consumer. See the
[API guide](docs/API.md) for behavior, limits and any host prerequisites.

## Local Buoy setup

On your own machine, install Buoy and run:

```sh
buoy ai init --both
```

## Contributing

[Contributing](CONTRIBUTING.md) · [License](LICENSE)
