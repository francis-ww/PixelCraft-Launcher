# Privacy & Security Policy

Last Updated: October 2026

**PixelCraft Launcher** is an independent, non-commercial, privately developed Windows desktop application for Minecraft: Java Edition. This document outlines how authentication, user credentials, and operational data are handled by the application.

---

## 1. Core Principles

- **No Developer Servers**: PixelCraft Launcher does not operate any centralized servers, user accounts, databases, telemetry collectors, or analytics backends.
- **No Password Handling**: The launcher does not request, collect, or store Microsoft or Xbox account passwords. Authentication is conducted externally via the official Microsoft login portal in the user's system web browser.
- **Local-Only Storage**: All configuration files, game instances, mods, and encrypted session tokens remain strictly on the user's local file system.
- **No Third-Party Analytics**: The launcher contains no advertising identifiers, third-party analytics trackers, or user behavioral profiling tools.

---

## 2. Microsoft & Xbox Authentication

PixelCraft Launcher integrates with Microsoft identity services in accordance with official Microsoft and Mojang protocol specifications:

1. **OAuth 2.0 PKCE Flow**:
   - The launcher initiates authentication by opening the official Microsoft login portal (`https://login.microsoftonline.com/`) in the user's **system default web browser**.
   - Proof Key for Code Exchange (**PKCE**) using the SHA-256 (`S256`) method is enforced to protect against authorization code interception.
   - The authentication callback is received by a temporary, single-use HTTP loopback server listening on the loopback interface (`127.0.0.1`).
   - A cryptographically random `state` parameter is generated per request and validated before the authorization code is accepted.
   - Host headers are validated to ensure only loopback requests (`localhost` / `127.0.0.1`) are accepted.
   - Once the callback is processed or timed out, the loopback server is closed and network resources are released.

2. **OS-Protected Credential Encryption (Windows DPAPI)**:
   - Acquired session credentials (`refresh_token` and temporary `access_token`) are encrypted at rest using **Windows Data Protection API (DPAPI)** via Electron's `safeStorage` subsystem.
   - Credential protection is managed by the operating system, binding encryption keys to the user's local Windows login account.
   - Plaintext fallback storage is not permitted: if DPAPI encryption is unavailable, the launcher refuses to persist authentication tokens.
   - Metadata stored in `accounts.json` contains only public profile details (username, player UUID, skin preview URL, and active account selection). No tokens or secrets exist in plaintext configuration files.

3. **Game Launch Credential Passing**:
   - When launching the Minecraft client, authentication arguments (`--accessToken`) and verified UUIDs are passed directly to the game process via command-line arguments in the Electron Main process.
   - The sandboxed Renderer process (the user interface) has no access to sensitive raw tokens.

4. **Account Removal & Credential Wiping**:
   - When a user logs out or removes an account within the launcher, its encrypted credentials in the DPAPI store are cleared.

> [!NOTE]
> **Microsoft Services Approval Status**: The OAuth 2.0 PKCE browser authorization flow and code exchange are fully implemented and verified. However, official **Minecraft Services Application ID approval is currently pending**. Consequently, downstream authentication with Minecraft Java Edition services is temporarily restricted until official application approval is completed by Microsoft / Mojang.

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

PixelCraft Launcher is an independent, non-commercial, privately developed Windows application by Francis Wang ([@francis-ww](https://github.com/francis-ww)).

If you have questions regarding this policy or the application's implementation, please open an issue in the official GitHub repository.

---

## 6. Disclaimer

PixelCraft Launcher is an independent, unofficial launcher and is **NOT** affiliated with, endorsed by, or associated with Mojang AB, Microsoft Corporation, or any of their affiliates.
