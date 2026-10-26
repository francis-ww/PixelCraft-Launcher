# Privacy & Security Policy

Last Updated: October 2026

**PixelCraft Launcher** is committed to protecting the privacy and security of its users. This document outlines how authentication, user credentials, and operational data are handled by the application.

---

## 1. Core Principles

- **No Developer Servers**: PixelCraft Launcher does not operate any centralized servers, user databases, telemetry collectors, or analytics backends.
- **Zero Credential Interception**: The launcher **never** requests, intercepts, logs, or stores user Microsoft or Xbox passwords.
- **Local-Only Storage**: All configuration files, game instances, mods, and encrypted session tokens remain strictly on the user's local file system.
- **No Third-Party Analytics**: The launcher contains no advertising identifiers, third-party analytics trackers, or user behavioral profiling tools.

---

## 2. Microsoft & Xbox Authentication

PixelCraft Launcher integrates with Microsoft identity services strictly in accordance with official Microsoft and Mojang protocols:

1. **OAuth 2.0 PKCE Flow**:
   - The launcher initiates authentication by opening the official Microsoft login portal (`https://login.microsoftonline.com/`) in the user's **system default web browser**.
   - Proof Key for Code Exchange (**PKCE**) using the SHA-256 (`S256`) method is enforced to protect against authorization code interception attacks.
   - The authentication callback is received by a temporary, single-use HTTP loopback server listening exclusively on the loopback interface (`127.0.0.1`).
   - A cryptographically random `state` parameter is generated per request and validated before the authorization code is consumed.
   - As soon as the callback is received, the loopback server and all associated network sockets are immediately closed and destroyed.

2. **Session Token Encryption (Windows DPAPI)**:
   - Acquired session credentials (`refresh_token` and temporary `access_token`) are encrypted at rest using the **Windows Data Protection API (DPAPI)** via Electron's `safeStorage` subsystem.
   - Decryption keys are tied to the local Windows user account and hardware encryption provider.
   - Plaintext fallback storage is strictly forbidden. If DPAPI is unavailable, the launcher refuses to persist credentials.
   - Metadata stored in `accounts.json` contains only non-sensitive public profile details (username, player UUID, skin preview URL, and active profile selection). No tokens or secrets exist in plaintext configuration files.

3. **Game Launch Credential Passing**:
   - When launching the Minecraft client, official session tokens (`--accessToken`) and verified UUIDs are passed directly to the game process via standard command-line arguments in the Electron Main process.
   - The sandboxed Renderer process (the user interface) has no access to sensitive tokens.

4. **Account Removal & Credential Wiping**:
   - When a user logs out or removes an account within the launcher, all encrypted token records in the DPAPI store are permanently purged immediately.

---

## 3. Network Communication Endpoints

PixelCraft Launcher communicates solely with essential, user-authorized endpoints required for Minecraft functionality:

| Destination | Purpose | Type |
|---|---|---|
| `login.microsoftonline.com` | Microsoft OAuth 2.0 Authorization & Token Exchange | Official Microsoft |
| `user.auth.xboxlive.com` | Xbox Live User Token Authentication | Official Microsoft |
| `xsts.auth.xboxlive.com` | Xbox Secure Token Service (XSTS) Authorization | Official Microsoft |
| `api.minecraftservices.com` | Minecraft Services Authentication, Entitlement & Profile Query | Official Mojang / Microsoft |
| `piston-meta.mojang.com` / `launchermeta.mojang.com` | Minecraft Version Manifests and Release Metadata | Official Mojang |
| `resources.download.minecraft.net` / `libraries.minecraft.net` | Minecraft Game Assets, Objects, and Libraries | Official Mojang |
| `api.modrinth.com` | Modrinth Project Search, Version Lookup & Dependency Resolution | Public Modrinth API |
| `cdn.modrinth.com` | Modrinth Mod, Resource Pack, and Shader Downloads | Public Modrinth CDN |
| `meta.fabricmc.net` / `maven.fabricmc.net` | Fabric Loader Metadata and Libraries | Official Fabric |
| `maven.minecraftforge.net` | Forge Loader Metadata and Processors | Official Forge |
| `maven.neoforged.net` | NeoForge Loader Metadata and Artifacts | Official NeoForge |
| `meta.quiltmc.org` / `maven.quiltmc.org` | Quilt Loader Metadata and Libraries | Official Quilt |
| `bmclapi2.bangbang93.com` | Optional Community Download Mirror (Configurable by User) | Optional Mirror |

The launcher does not communicate with any other third-party servers.

---

## 4. Local File System & Isolation

- **Game Instances**: Each Minecraft instance is isolated in its own subfolder containing its distinct `saves/`, `mods/`, `config/`, and `resourcepacks/`. No game instance modifies another instance's data.
- **Deletions**: Instance deletions are routed through the native Windows Recycle Bin wherever possible to prevent accidental data loss.

---

## 5. Inquiries & Contact

PixelCraft Launcher is developed as an open-source personal project by Francis Wang ([@francis-ww](https://github.com/francis-ww)).

If you have any security or privacy questions, please open an issue in the official GitHub repository.

---

## 6. Disclaimer

PixelCraft Launcher is an independent, unofficial launcher and is **NOT** affiliated with, endorsed by, or associated with Mojang AB, Microsoft Corporation, or any of their affiliates.
