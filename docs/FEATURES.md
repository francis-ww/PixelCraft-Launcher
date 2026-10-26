# PixelCraft Launcher — Technical Features & Architecture

This document provides a comprehensive technical overview of the subsystems, completed capabilities, and engineering architecture of PixelCraft Launcher.

---

## 1. System Architecture

The application is structured into strict architectural tiers designed around stability, security, and process isolation:

```
┌─────────────────────────────────────────────────────────────┐
│                       Renderer Process                      │
│        (React 19, Tailwind CSS, 16-bit Design Tokens)       │
│  Independent, non-commercial, privately developed launcher  │
└──────────────────────────────┬──────────────────────────────┘
                               │ IPC (contextBridge, Strict Whitelist)
┌──────────────────────────────▼──────────────────────────────┐
│                    Preload / IPC Gateways                   │
│   (Type-safe contracts, sanitized data models, no secrets)  │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│                 Application Services Layer                  │
│  InstanceService  ·  VersionService  ·  DownloadService     │
│  LoaderService    ·  JavaService     ·  AccountService      │
│  ModrinthService  ·  LaunchService   ·  SettingsService     │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│                 Adapters & Storage Engines                  │
│   MicrosoftAuthService  ·  OAuthManager  ·  CredentialStore │
│   Fabric/Forge/NeoForge/Quilt Adapters  ·  Modrinth API     │
│   Windows DPAPI  ·  Isolated File System Instances          │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Completed Capabilities

### 2.1 Refined 16-Bit Pixel Art Design System (M1)
- Custom-crafted pixel art UI tokens including layered bevel borders, tactile button press states, retro badge tags, and custom SVG pixel icons.
- Complete 4-page navigation suite:
  - **Play Page**: Spotlight for the currently selected game instance, active player badge, quick instance status, and real-time log terminal.
  - **Download Center**: Categorized Modrinth browser and Vanilla / Loader installation wizard.
  - **Instances Page**: Multi-instance card grid with search, sorting, tag filtering, configuration overrides, and resource management.
  - **Settings Page**: Appearance, download sources, window management, Java configuration, and multi-account session controls.
- Frameless native window with custom draggable title bar and Windows DPI scaling support.

### 2.2 Vanilla Installation & Java Engine (M2)
- Manifest parsing from official Mojang endpoints (`piston-meta.mojang.com`) with automated fallback to BMCLAPI.
- Bounded-concurrency download engine with monotonic progress updates, `.part` atomic verification, and resume-on-retry.
- SHA-1 integrity validation across all client JARs, runtime libraries, and game asset objects.
- System Java scanner covering Windows Registry, `PATH`, and standard installation paths with automated Java 8 to Java 21+ compatibility verification.
- Offline profile generation using deterministic RFC 4122 MD5 V3 UUIDs.

### 2.3 Comprehensive Mod Loader Ecosystem (M3)
- **Fabric**: Automatic meta lookup, inherited version manifest generation, and KnotClient runtime assembly.
- **Forge**: Implementation of modern Forge workflows:
  - Downloads official Mojang obfuscation mappings.
  - Executes Forge installer processors (mapping merge, binary patch application via `jarsplitter`, and runtime client packaging).
  - Handles classpath construction and native library extraction.
- **NeoForge**: Specialized NeoForge manifest resolution, modern processor support, and runtime execution.
- **Quilt**: Integration with Quilt Meta and automatic Quilt Loader library deployment.

### 2.4 Multi-Instance Isolation & Safety (M4)
- **Directory Isolation**: Shared core files (`assets/`, `libraries/`, `versions/`) remain clean, while instance-specific files (`mods/`, `saves/`, `config/`, `resourcepacks/`, `shaderpacks/`) live in independent instance directories.
- **Lifecycle Management**:
  - Safe cloning with three distinct modes: Full Clone, Keep Configs Only, or Clean Instance.
  - Protected deletions routed through the Windows Recycle Bin.
  - Active game protection: Prevents modifying or deleting instances while the game process is running.
- **One-Click Integrity Repair**: Checks installed libraries and hashes against version manifests, re-fetching any corrupted or missing files.

### 2.5 Modrinth Resource Center & Dependency Resolver (M5)
- Search integration with facet filtering by Minecraft version, loader type, and resource type (Mods, Resource Packs, Shaders).
- Multi-tier memory caching with exponential backoff for HTTP 429 rate-limiting.
- **Recursive Dependency Resolver**:
  - Traverses required dependencies with circular reference blocking (visited set and depth limiter).
  - Identifies already-installed dependencies to prevent redundant downloads.
  - Evaluates incompatible dependency warnings before installation.
- **Transaction-Safe Installer**: Downloads to temporary staging files, verifies SHA-512 hashes, and rolls back cleanly if any error occurs.
- **Resource Management**: In-place enabling/disabling via `.jar` $\leftrightarrow$ `.jar.disabled` renaming, update checking, and safe uninstallation.

### 2.6 Microsoft OAuth 2.0 PKCE & Account Management (M6.1 + M6.2)
- **RFC 7636 PKCE Public Client Flow**:
  - 32-byte base64url `code_verifier` with SHA-256 `code_challenge`.
  - Cryptographically secure 32-hex `state` validation against CSRF.
  - Temporary HTTP loopback listener bound to `127.0.0.1` (`http://localhost:<port>/`) for authorization code reception.
  - Automatic browser launch using the system default browser.
  - Host validation (`localhost` / `127.0.0.1`) and 120-second timeout safeguard.
- **Xbox Live & Minecraft Authentication Pipeline**:
  - Exchange Microsoft OAuth access token for Xbox Live User Token.
  - Authorize with XSTS for the `rp://api.minecraftservices.com/` relying party.
  - Obtain Minecraft Services access token.
  - Check Minecraft Java Edition ownership (`product_minecraft` / `game_minecraft`).
  - Retrieve player profile (UUID, official username, active skin preview URL).
  - Error classification for child accounts (2148916238), missing Xbox gamertags (2148916233), region restrictions (2148916235), and rate limits (429).
- **Windows DPAPI Credential Storage**:
  - Session refresh tokens are encrypted at rest using OS-protected credential encryption via Windows DPAPI (Electron `safeStorage`).
  - Plaintext fallback is rejected; if DPAPI is unavailable, credentials are not saved.
  - Sanitized public profiles in `accounts.json` contain zero secrets or tokens.
  - Account deletion automatically purges encrypted credentials from the DPAPI store.
- **Launch Integration**:
  - Detects active account type before launch.
  - Validates token lifetime and transparently executes silent refresh if near expiry.
  - Injects `--username <Name> --uuid <UUID> --accessToken <Token> --userType mojang` for Microsoft accounts.
  - Injects `--username <Name> --uuid <UUID> --userType legacy` for offline accounts.

> [!NOTE]
> **Microsoft Services Approval Status**: The OAuth 2.0 PKCE browser authorization loopback mechanism is implemented and validated. Downstream access to official Minecraft Services APIs is pending official Microsoft / Mojang Application ID approval. Online login is therefore temporarily restricted until official application approval is completed.

---

## 3. Upcoming Milestones

- **M6.3 — Modpack (.mrpack) Support**: Full parsing, index extraction, overrides merging, and instance creation from Modrinth modpack archives.
- **M7 — Diagnostics, Backup & Release Packaging**: Automated instance backup snapshots, diagnostic export, and production installer creation.
