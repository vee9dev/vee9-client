# vee9-client

# V9 Client

A custom launcher for **Minecraft: Java Edition**, built from scratch with Electron and TypeScript.

V9 signs you in with your Microsoft account, installs the official game files straight from Mojang, and launches vanilla or Fabric profiles with a curated, optional set of performance and quality-of-life mods from Modrinth.

![V9 Client home screen](docs/screenshot.png)

## Features

- **Microsoft sign-in.** Standard Microsoft → Xbox Live → Minecraft authentication, including a game ownership check. Sessions refresh silently.
- **Official game files only.** Versions, libraries, assets and the Java runtime are downloaded from Mojang's servers and verified by SHA-1. Nothing from Mojang is bundled or redistributed.
- **Automatic Java.** The matching Mojang Java runtime is installed for each Minecraft version. A custom Java path can be set instead.
- **Profiles.** Each profile has its own Minecraft version, loader (vanilla or Fabric), memory setting and game folder.
- **Mods from Modrinth.** Mods can be switched on per profile. Required dependencies are resolved automatically, and mods you add to the folder yourself are never touched. The curated list covers performance and comfort mods such as Sodium, Lithium, FerriteCore, Zoomify, AppleSkin and Mod Menu.
- **Live console.** Game output is streamed into the launcher and written to log files.
- **Repair.** Re-verifies every file of a profile and re-downloads anything that is missing or damaged.

V9 contains no cheats and nothing that bypasses server rules.

## Authentication and privacy

V9 is a public client application with no client secret.

| Step | Endpoint |
| --- | --- |
| Microsoft OAuth 2.0, authorization code with PKCE, scopes `XboxLive.signin offline_access` | `login.microsoftonline.com/consumers` |
| Xbox Live user token | `user.auth.xboxlive.com` |
| XSTS token for `rp://api.minecraftservices.com/` | `xsts.auth.xboxlive.com` |
| Minecraft access token and profile | `api.minecraftservices.com` |

- The sign-in page is Microsoft's own, opened in a separate window. V9 never sees your password.
- Refresh and access tokens are stored **only on your device**, encrypted with the operating system's secure storage (DPAPI on Windows, Keychain on macOS, libsecret on Linux).
- The Minecraft access token is passed only to the locally launched game.
- No tokens or personal data are sent to any server other than the Microsoft, Xbox and Mojang endpoints above. V9 has no backend of its own.

The relevant code is in [`src/main/auth/microsoft.ts`](src/main/auth/microsoft.ts) and [`src/main/auth/accounts.ts`](src/main/auth/accounts.ts).

## Network access

V9 connects only to these hosts:

- `piston-meta.mojang.com`, `launchermeta.mojang.com`, `libraries.minecraft.net`, `resources.download.minecraft.net`, `piston-data.mojang.com` (game files and Java runtime)
- `login.microsoftonline.com`, `user.auth.xboxlive.com`, `xsts.auth.xboxlive.com`, `api.minecraftservices.com` (sign-in)
- `meta.fabricmc.net`, `maven.fabricmc.net` (Fabric loader)
- `api.modrinth.com`, `cdn.modrinth.com` (mods, only for Fabric profiles)
- `mc-heads.net` (player head shown next to your account name)

## Getting started

Requires Node.js 20 or newer.

```bash
npm install
npm start        # build and run
npm test         # unit tests for the launcher core
npm run dist     # build an installer for your platform (output in release/)
```

### Using your own Azure app

To build V9 yourself with Microsoft sign-in, register your own app in the [Azure portal](https://portal.azure.com) under **App registrations**:

1. Supported account types: **Personal Microsoft accounts only**.
2. Redirect URI: platform **Public client/native (mobile & desktop)**, URI `https://login.microsoftonline.com/common/oauth2/nativeclient`.
3. Enter the **Application (client) ID** in V9 under *Optionen → Microsoft-Login*, or set the environment variable `CORTEX_MSA_CLIENT_ID`.

New Azure apps must be approved by Mojang before they can use the Minecraft services API. Until then the API answers with HTTP 403. For local testing, V9 also offers offline accounts, which work in singleplayer and on servers running in offline mode.

## Project structure

```
src/
  main/
    main.ts              window, app:// protocol, IPC
    http.ts              downloads with SHA-1 check, retries, concurrency limit
    auth/
      microsoft.ts       OAuth (PKCE) and the Xbox / XSTS / Minecraft chain
      accounts.ts        account storage with encrypted tokens
      offline.ts         offline UUIDs, identical to vanilla
    game/
      versions.ts        Mojang version manifest, Fabric meta, profile merging
      rules.ts           OS and feature rules from version files
      libraries.ts       classpath, Maven paths, natives
      assets.ts          asset index and objects
      java.ts            Mojang Java runtimes
      mods.ts            Modrinth catalog, dependency resolution, mod sync
      launch.ts          JVM and game arguments
      runner.ts          install flow and game process
  preload/preload.ts     narrow, typed bridge (contextIsolation and sandbox on)
  renderer/              user interface (HTML, CSS, TypeScript, no framework)
  shared/types.ts        types shared between processes
test/                    unit tests
```

## Disclaimer

V9 Client is an independent project. It is not affiliated with, endorsed by or associated with Mojang Studios or Microsoft. Minecraft is a trademark of Mojang Studios. A legitimate copy of Minecraft: Java Edition is required to play online.

## Contact

Built by Vee9. Questions and bug reports are welcome in the issues of this repository.
