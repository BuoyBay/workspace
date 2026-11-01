---
title: "Workspace integration getting started"
summary: "Install workspace and use its native API with explicit ownership."
category: "Guides"
order: 10
keywords: ["workspace", "urls", "finder", "applications", "launch services"]
related: ["index.md", "api.md", "typed-api.md"]
---

# Getting started

Install the sealed package in `packages/` and import `workspace`. This macOS-only lookup example validates a URL and finds its handler without launching another application. Perform opening or revealing only from an explicit user action; neither validation nor lookup performs that action for you.

```buoy
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

For portable open/reveal code, gate on `OpenAndRevealSupported()`. Validation is
side-effect-free on all three targets. `OpenAndRevealAvailable()` only probes
libraries; service/handler failures are reported by the operation. Linux calls
belong on a worker; Windows calls require an STA-compatible worker; macOS keeps
its main-thread requirement. See [API contracts](api.md) before launching.

## Handle native constraints

Default URL schemes are HTTP, HTTPS, mailto, and tel. Other schemes require `allowCustomScheme = True`, but file, javascript, data, and about remain rejected. Relative URLs, URL credentials, control characters, and hostless HTTP(S) URLs are rejected. `OpenFile` requires an existing file/folder and can launch registered executable application bundles or automation documents; pass only items the user chose. True from an opening method means the native backend accepted the request, not that processing or a remote page load completed.

[Package overview](index.md) · [API reference](api.md)
