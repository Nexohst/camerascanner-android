# CameraScanner SDK — Android

**Open Beta** · v1.4.9 · Started May 25, 2026

---

## What It Does

Detects virtual and spoofed cameras on Android devices — the kind used to feed
pre-recorded video or virtual camera output into apps that expect a real camera feed.

Works entirely on-device. No internet connection required. No data leaves the device.

---

## How It Works

Six independent detection methods run in a single call:

| Method | What It Checks |
|--------|----------------|
| PackageManager scan | Known virtual camera applications installed on the device |
| Clone / Parallel Space | Whether the app is running inside a secondary user profile or attacker-controlled sandbox |
| Camera2 API analysis | Hardware characteristics that virtual cameras cannot accurately replicate |
| Kernel filesystem scan | Virtual camera drivers at the system level |
| Overlay + UsageStats | Screen recording activity and overlay permissions |
| Memory + hook detection | Injected frameworks in process memory and active debugger attachment |

Single call, structured result:

```kotlin
val result = isFakeCamera(context)

result.isFake        // Boolean
result.confidence    // "HIGH" | "MEDIUM" | "CLEAN" | "UNKNOWN"
result.detected      // List of findings
result.safeDevices   // Confirmed physical cameras
result.errors        // Anything that could not be checked, with explanation
result.scannedAt     // UTC timestamp, ISO-8601
result.sdkVersion    // "1.4.9"
```

No silent failures. If a method cannot run — permission missing, path inaccessible,
service unavailable — it says so in `errors`. If a system check is actively blocked,
the result is escalated to `HIGH` rather than returning a false `CLEAN`.

---

## Tested On

| Device | API | Platform | Result | Scan Time |
|--------|-----|----------|--------|-----------|
| Pixel 7 emulator | 33 | x86_64 | CLEAN — Safe Devices: 2 | 76ms |
| Pixel 9 emulator | 36.1 | x86_64 | CLEAN — Safe Devices: 2 | 898ms |
| Xiaomi HyperOS physical device | 36.0 (Android 16) | ARM / MediaTek | CLEAN — Safe Devices: 2 | 394ms |

---

## Current State

Phase 1 (hardening and bugfixes) is complete. Detection engine is stable and validated
on physical hardware including a Xiaomi device running Android 16 (HyperOS).

**What's stable:**
- All 6 detection methods
- Confidence tier system (HIGH / MEDIUM / CLEAN / UNKNOWN)
- Fail-Open: any detection blindness forces HIGH — no false CLEANs on hooked devices
- False positive whitelist (15 legitimate apps)
- OEM Dual App vs Illegal Sandbox classification
- Permission chain (CAMERA + PACKAGE_USAGE_STATS)
- ProGuard / R8 rules — public API preserved, internals obfuscated

**What's coming:**
- API 26–30 backward compatibility test (physical devices)
- Module separation — `:library` (SDK) + `:app` (test harness)
- AAR packaging for distribution
- Versioned blacklist update mechanism
- Passive liveness detection (on-device only)
- Active liveness (challenge-response)

---

## Why

Cloud-based identity verification is expensive, creates data liability, and requires
constant network access. This SDK is an attempt to build something that runs at full
capacity on the device itself — aimed at platforms that cannot justify per-verification
pricing or do not want biometric data leaving the device.

The end goal is a local-first alternative to services like AWS Rekognition or Azure Face.

---

## Source

This repository is **closed source**. The code is not publicly available.

What is here: documentation, devlog, and release notes.

instagram: ahmetaligm

---

## Status

```
Version   : 1.4.9 (open beta)
Platform  : Android (Kotlin), API 26+
Started   : May 25, 2026
Updated   : May 29, 2026
Time Spent: ~20 hours total
Author    : Nexohst
```
