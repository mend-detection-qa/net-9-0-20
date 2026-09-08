# version-pm-dotnet-sdk-net8-vs-net9 (Probe B — SDK 9.0.20)

## Feature exercised

This probe exercises the bundled NuGet client change introduced with
.NET SDK 9.0.20 (NuGet 6.11). When the Mend Unified Agent shells out
to `dotnet restore` to generate `project.assets.json`, the SDK
version determines:

- Which TFM asset group is selected in `project.assets.json`
  (net9.0 vs net8.0 paths inside each nupkg differ for
  Microsoft.Extensions.* packages).
- Whether `System.Diagnostics.DiagnosticSource` is included as a
  transitive dep (net8 includes the polyfill; net9 BCL absorbs it).
- Whether a `restoreAuditProperties` block appears in
  `project.assets.json` (SDK 9 populates this; SDK 8 does not).

This probe is **Probe B** in the
`version-pm-dotnet-sdk-net8-vs-net9` pair. Its sibling Probe A uses
`<TargetFramework>net8.0</TargetFramework>` and
`pm_version_under_test = "8.0.404"`.

## Pattern catalog entry

Defined in:
`plugins/dotnet-nuget/skills/nuget-core/references/version-variations.md`
Pattern: `version-pm-dotnet-sdk-net8-vs-net9`

## Project layout

```
version-pm-dotnet-sdk-net8-vs-net9-20260908-223801/
├── src/
│   └── DotnetSdkVersionProbe/
│       └── DotnetSdkVersionProbe.csproj
├── .whitesource
├── README.md
└── expected-tree.json
```

## Expected dependency tree

### Direct dependencies

| Package | Version | Source |
|---------|---------|--------|
| Microsoft.Extensions.Http | 8.0.0 | registry (nuget.org) |
| Serilog.Extensions.Hosting | 8.0.0 | registry (nuget.org) |

### Transitive dependencies (net9.0 asset selection)

Microsoft.Extensions.Http 8.0.0 (net9.0 assets):
- Microsoft.Extensions.Logging.Abstractions 8.0.0
- Microsoft.Extensions.Options 8.0.0
- Microsoft.Extensions.DependencyInjection.Abstractions 8.0.0

Serilog.Extensions.Hosting 8.0.0 (net9.0 assets):
- Serilog 3.1.1
- Microsoft.Extensions.Hosting.Abstractions 8.0.0
- Microsoft.Extensions.Logging 8.0.0
- Microsoft.Extensions.Logging.Abstractions 8.0.0 (shared)
- Microsoft.Extensions.DependencyInjection 8.0.0
- Microsoft.Extensions.DependencyInjection.Abstractions 8.0.0 (shared)
- Microsoft.Extensions.Options 8.0.0 (shared)
- Microsoft.Extensions.Configuration.Abstractions 8.0.0

### Key difference from Probe A (net8.0)

`System.Diagnostics.DiagnosticSource` is **absent** in this probe's
expected tree. Under net9.0, this package is part of the .NET BCL and
is not emitted as a dependency node in `project.assets.json`. Probe A
(net8.0) includes it.

`restoreAuditProperties` appears in `project.assets.json` generated
by SDK 9.0.20 but should NOT appear in the expected tree (it is
metadata, not a dep).

## Mend config

Bucket A — default-emit `.whitesource` with `dotnet` pinned to
`"9.0.20"`.

The `dotnet` install-tool key covers both the .NET SDK and the NuGet
client bundled with it. Pinning to `"9.0.20"` ensures
`install-tool` provisions exactly the SDK version that generates the
`project.assets.json` this probe's expected tree was built against.

If the pin is omitted or set to an 8.x version, `dotnet restore` will
produce a `project.assets.json` with `net8.0` targets and a different
transitive set (including `System.Diagnostics.DiagnosticSource`),
causing the downstream comparison to fail with an unexpected
transitive-set difference.

`configMode` is `"AUTO"` — no `whitesource.config` ships with this
probe.

## Paired probe reference

Probe A is generated alongside this probe and lives at:
`$QA_DETECTION_OUTPUT_DIR/nuget/version-pm-dotnet-sdk-net8-vs-net9-*/`
with `pm_version_under_test = "8.0.404"` and
`<TargetFramework>net8.0</TargetFramework>`.

## Release context

Source release: .NET 9.0.20
Category: bundled_pm_change
Mend relevance: UA shells out to `dotnet restore` for dependency
resolution; the bundled NuGet version in SDK 9.0.20 (NuGet 6.11)
changes the `project.assets.json` schema and package-selection
behavior for net9.0 targets.
