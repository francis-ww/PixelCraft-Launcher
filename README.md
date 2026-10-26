# PixelCraft Launcher

[![Platform: Windows](https://img.shields.io/badge/Platform-Windows%2010%2F11%20x64-blue.svg)](#system-requirements)
[![Status: Development](https://img.shields.io/badge/Status-Under%20Active%20Development-orange.svg)](#development-status)
[![Architecture: Electron + React](https://img.shields.io/badge/Stack-Electron%20%7C%20React%20%7C%20TypeScript-purple.svg)](#technology-stack)

> **Notice**: PixelCraft Launcher is an independent, non-commercial, privately developed Windows launcher.  
> **NOT AN OFFICIAL MINECRAFT PRODUCT. NOT APPROVED BY OR ASSOCIATED WITH MOJANG OR MICROSOFT.**

---

## Overview

**PixelCraft Launcher** is an independent, non-commercial, privately developed desktop launcher for **Minecraft: Java Edition** on Windows (Windows 10/11 x64). Built with a distinctive **Refined 16-bit Pixel Art** aesthetic combined with modern desktop usability, the launcher provides a robust, isolated, and reliable environment for managing vanilla game installations, popular mod loaders (Fabric, Forge, NeoForge, Quilt), Modrinth community resources, and multi-instance configurations.

The project prioritizes:
$$\text{Data Safety} > \text{Correctness} > \text{Stability} > \text{Usability} > \text{Compatibility} > \text{Performance}$$

---

## Key Features

### 1. Robust Game Engine & Java Runtime Management
- **Vanilla Minecraft Support**: Direct installation and verification from official Mojang endpoints and trusted mirrors.
- **Integrity Verification**: Strictly verifies SHA-1 checksums for all game jars, libraries, and asset objects.
- **Bounded Concurrency & Fault Tolerance**: Resilient download engine with chunked concurrency, atomic `.part` replacement, and network timeout recovery.
- **Java Auto-Detection**: Scans system registries and `PATH` for installed Java Virtual Machines (Java 8 through Java 21+ LTS) with automatic version-compatibility enforcement based on target Minecraft releases.

### 2. Multi-Mod Loader Integration
- **Fabric**: Seamless metadata querying, inherited version generation, and runtime dependency resolution.
- **Forge**: Full support for modern Forge workflows and runtime processor execution (mapping downloads, patch application, and class extraction).
- **NeoForge**: Specialized NeoForge manifest resolution and artifact assembly.
- **Quilt**: Native Quilt Meta parsing and Knot runtime library dependency management.

### 3. Isolated Multi-Instance Management
- **True Directory Isolation**: Dedicated instances directory where each profile maintains its own isolated `mods/`, `saves/`, `config/`, `resourcepacks/`, and `options.txt`.
- **Instance Operations**: Safe creation, cloning, configuration overrides (custom Java, memory bounds, resolution), and deletion via the Windows Recycle Bin.
- **One-Click Integrity Repair**: Automatic verification and re-downloading of corrupted libraries or assets.

### 4. Modrinth Resource Center
- **Integrated Catalog**: Direct browsing and searching of Modrinth mods, resource packs, and shaders.
- **Recursive Dependency Resolution**: Automatically resolves required dependencies while blocking circular references and flagging incompatibilities.
- **Transaction-Safe Installation**: SHA-512 integrity verification with automated rollback on failure.
- **Resource Management**: Toggle mods on/off without deleting files (`.jar` $\leftrightarrow$ `.jar.disabled`).

### 5. Microsoft OAuth 2.0 & Session Management
- **Standard-Compliant Authentication**: Built on the **OAuth 2.0 Authorization Code Flow with PKCE** (RFC 7636, S256 challenge), specifically designed for public desktop clients.
- **Secure Browser Authorization**: Opens the user's system default web browser for authentication directly on Microsoft's official servers. The launcher does not request or collect user Microsoft account passwords.
- **Local Loopback Callback**: Runs a temporary, single-use HTTP loopback server strictly bound to `127.0.0.1` (`http://localhost:<port>/`) to capture the authorization code with mandatory `state` verification to prevent CSRF.
- **Xbox Live & Minecraft Services Token Chain**: Sequentially performs Xbox Live token acquisition, XSTS token authorization (`rp://api.minecraftservices.com/`), and Minecraft Services token exchange.
- **Java Edition Entitlement Verification**: Verifies ownership (`product_minecraft` / `game_minecraft`) prior to session registration.
- **OS-Protected Credential Encryption**: Sensitive session credentials (such as refresh tokens) are encrypted on disk using OS-protected credential encryption via **Windows DPAPI** (`safeStorage`). Plaintext credential fallbacks are strictly prohibited.
- **Silent Session Refresh**: Automatically refreshes expired or near-expiry session tokens before game launches without prompting the user.
- **Offline Mode Support**: Complete offline player profile management with deterministic UUIDs for offline gameplay and local development.
- **Service Approval Status**: The OAuth 2.0 browser authorization and PKCE code exchange have been successfully implemented and tested. Because official **Minecraft Services Application ID approval is currently pending**, downstream authentication with Minecraft Java Edition services is temporarily restricted until official approval is granted.

---

## Technology Stack

| Layer | Technologies |
|---|---|
| **Runtime & Framework** | Electron, Node.js (LTS), Vite, electron-vite |
| **Language & Typings** | TypeScript (Strict Mode enabled across Main, Preload, and Renderer) |
| **User Interface** | React 19, Tailwind CSS, Zustand |
| **Minecraft Libraries** | `@xmcl/core`, `@xmcl/installer`, `@xmcl/user`, `@xmcl/modrinth` |
| **Security & Cryptography** | Windows DPAPI (`node:crypto`, Electron `safeStorage`), PKCE (SHA-256) |
| **Testing & Quality** | Vitest, ESLint, TypeScript Compiler (`tsc`) |

---

## Development & Verification Status

PixelCraft Launcher is being developed systematically through planned milestone quality gates:

- [x] **M0 — Project Initialization & Infrastructure**: Electron + React + TypeScript scaffolding, strict IPC contracts, security boundaries (`contextIsolation: true`, `nodeIntegration: false`, strict CSP).
- [x] **M1 — Refined 16-bit Pixel Art Design System**: Handcrafted design tokens, pixel-perfect buttons, cards, modals, and navigation.
- [x] **M2 — Vanilla Engine & Java Management**: Manifest downloading, bounded concurrency hash verification, Java compatibility scanning, and offline launch.
- [x] **M3 — Mod Loader Integration**: Fabric, Forge, NeoForge, and Quilt installation, processor execution, and launch pipelines.
- [x] **M4 — Instance Architecture**: Multi-instance isolation, configuration overrides, and safe lifecycle management.
- [x] **M5 — Modrinth Resource Center**: Search, dependency graph resolution, SHA-512 verification, resource activation/deactivation, and updates.
- [x] **M6.1 & M6.2 — Microsoft Authentication & Session Management**:
  - OAuth 2.0 PKCE Authorization Code loopback implementation.
  - Xbox Live, XSTS, and Minecraft Services authentication pipeline.
  - Windows DPAPI secure credential storage.
  - Multi-account management and launch argument injection.
- [ ] **M6.3 — Modpack (.mrpack) Support**: *(Planned)*
- [ ] **M7 — Diagnostics, Backup & Release Packaging**: *(Planned)*

> [!IMPORTANT]
> **Microsoft Authentication & Services Approval Notice**  
> The Microsoft OAuth 2.0 PKCE authorization flow is functional and verified in local development: the launcher opens the system browser, receives loopback authorization codes, and handles OAuth token exchange.  
> However, **Minecraft Services Application ID approval is currently pending.** Access to official Minecraft Services (such as Xbox Live / Minecraft entitlement and profile APIs) is subject to official registration approval. Therefore, full online Microsoft login remains restricted until approval is granted.

---

## Security & Privacy Highlights

PixelCraft Launcher is engineered around desktop security and privacy:

1. **No Password Handling**: All authentication occurs directly on official Microsoft login domains in the user's default browser.
2. **OS-Protected Credential Encryption**: Session refresh tokens are stored encrypted using Windows DPAPI. No credentials or tokens are ever sent to any third-party or developer-operated servers.
3. **No Telemetry or Tracking**: The launcher contains no remote telemetry, analytics, tracking beacons, or cloud logging.
4. **Renderer Isolation**: The React user interface runs in a sandboxed context and has zero access to raw tokens or sensitive operating system APIs.

For detailed privacy information, please consult [docs/PRIVACY.md](docs/PRIVACY.md).  
For the complete technical breakdown of features, please refer to [docs/FEATURES.md](docs/FEATURES.md).

---

## System Requirements

- **Operating System**: Windows 10 / Windows 11 (64-bit)
- **Runtime Environment**: Java Runtime Environment (JRE) / Java Development Kit (JDK) 17 or 21 (for modern Minecraft versions)
- **Hardware**: Compatible x64 processor, 4 GB+ RAM recommended (allocated per instance)

---

## Disclaimers & Trademarks

- **Minecraft** is a registered trademark of Mojang AB.
- **Microsoft** and **Xbox** are registered trademarks of Microsoft Corporation.
- PixelCraft Launcher is an independent, non-commercial, privately developed project and is **NOT** an official Minecraft product, nor is it endorsed by, affiliated with, or associated with Mojang Studios, Microsoft Corporation, or any of their affiliates.
- Modrinth is an independent platform operated by Rinth, Inc.
