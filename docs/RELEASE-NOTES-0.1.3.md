# Treacky.Tols 0.1.3

**Live 2× FPS overlay. One executable.** The uncap engine now multiplies the
emulator's measured frame rate instead of printing a fixed number, and the
release ships as a single self-contained `Treacky.Tols.exe`.

## Download

> **WARNING:** This release includes emulator-process injection and live memory/code patching. Use only on a controlled system and emulator installation you own or administer. It modifies renderer hooks and FPS data; it can change power, timer, priority, registry, and emulator runtime state. Test in a non-production environment and review [SECURITY.md](../SECURITY.md) before use.

`Treacky.Tols-0.1.3-win-x64.exe` — no ZIP, no installer, no .NET Desktop Runtime
install. The uncap payload, branding, and profile avatar are embedded in the
executable and unpacked on first use.

## What changed

### The overlay follows the real rate

The old cave wrote a fixed 1000. The new cave multiplies the frame counter that
BlueStacks already maintains, so the overlay shows **2× the live measured FPS**:
100 real frames become 200, 143 becomes 286, and the number moves with the
emulator instead of sitting still. `Multiplier=` in `UncapEngine.ini` controls
the factor (1–10, default 2).

### One Boost, one Restore

**Boost priority** is now the single arming action: power plan, CPU clock,
payload injection, and the high-FPS config in one click. **Restore normal**
unhooks the engine and returns the previous power plan. The separate Uncap FPS
and Eject uncap buttons are gone. Re-arming restores the store first, so
repeated clicks no longer compound 200 → 400 → 800.

### Injection that survives the protected driver

The payload loads through a thread-scoped `WH_GETMESSAGE` hook and an in-module
`xor eax, eax; ret` gadget. It needs no `OpenProcess`, remote allocation, or
remote thread, so the stock BlueStacks driver does not block it. Priority
changes still go through the payload's own `OptimaizerSetPriority` export when
the outside handle is denied.

### Single-file packaging

`UncapEngine.dll`, `UncapEngine.ini`, `branding.json`, and `assets/avatar.png`
are embedded in the assembly. The payload is extracted to
`%APPDATA%\Treacky.Tols\payload\<version>` before injection; the loose files are
no longer required next to the executable.

## Verification performed

- Release build: 0 warnings, 0 errors.
- Service harness: 23 read-only checks pass; 26 checks pass with `--system`.
- Single-file publish: `Treacky.Tols.exe` starts, extracts the payload, and
  serves the embedded avatar.
- Live BlueStacks verification of the 2× overlay: pending. The hook path is
  exercised by the payload export and gadget checks; the visible rate change
  still needs a run on the target build.
- Authenticode: `NotSigned` (expected; no CA certificate was supplied).

## Signature status

This build was produced without a CA-issued code-signing certificate, so it is
**unsigned**. `Get-AuthenticodeSignature` reports `NotSigned`. Verify the
SHA-256 published with the release and build from source if you need stronger
provenance. The release pipeline supports real signing and labels a build signed
only after `signtool` and `Get-AuthenticodeSignature` both succeed.

## Before running

Verify the downloaded executable on the same machine that will run it:

```powershell
Get-FileHash .\Treacky.Tols-0.1.3-win-x64.exe -Algorithm SHA256
Get-AuthenticodeSignature .\Treacky.Tols-0.1.3-win-x64.exe | Format-List Status,SignerCertificate
```

Compare the SHA-256 value to the `SHA-256` field in this release. For this build, the expected signature result is `NotSigned`; do not treat it as a CA-signed executable. Never launch a renamed or mismatched copy.

## Known limitations

- **Live overlay verification is pending.** The injection and patch paths are
  covered by the harness, but the visible 2× rate has not been confirmed on a
  running BlueStacks build in this release.
- BlueStacks 5.22 denies external priority changes. The app reports the block
  and uses the in-target export instead of bypassing protection.
- Emulator-specific features target BlueStacks 5 and MSI App Player 5 on x64.
- The native renderer module (EGL swap hook, frame pacing, shader grade) is not
  included.
- The build is unsigned.

## Upgrading

0.1.3 supersedes 0.1.1. Settings, presets, and backups under
`%APPDATA%\Treacky.Tols` and `%ProgramData%\Treacky.Tols` are compatible. The
payload now lives under `%APPDATA%\Treacky.Tols\payload\0.1.3`; older loose
`UncapEngine.dll` files next to a previous executable are ignored.

## Links

- [Documentation](../docs/)
- [Reverse engineering](../docs/01-REVERSE-ENGINEERING.md)
- [Feature map](../docs/02-FEATURE-MAP.md)
- [Security policy](../SECURITY.md)
- [Changelog](../CHANGELOG.md)

Developer: **FRHANLVLY** · FLY.PROJECT-X · `@FRHANLVLY`
