# prism-offline-patcher

Auto-patcher that tracks [PrismLauncher](https://github.com/PrismLauncher/PrismLauncher) releases and applies minimal offline patches to remove the premium account requirement.

## How it works

A GitHub Actions cron job runs daily, checks for new upstream Prism Launcher releases, and if a new version is found:

1. Clones Prism Launcher at that tag
2. Applies the 3 `.patch` files via `git apply`
3. Creates a GitHub Release with patched source archives + combined `.patch` file

## The patches

Only **3 files** changed:

| Patch | File | Change |
|-------|------|--------|
| `0001` | `launcher/minecraft/auth/MinecraftAccount.h` | `ownsMinecraft()` hardcoded to `return true` |
| `0002` | `launcher/minecraft/auth/AccountList.cpp` | `anyAccountIsValid()` hardcoded to `return true` |
| `0003` | `launcher/LaunchController.cpp` | `decideLaunchMode()` bypasses auth gating while preserving explicit offline/demo launches |

## Manual usage

```bash
# Clone Prism at any tag
git clone --depth 1 --branch 10.0.5 https://github.com/PrismLauncher/PrismLauncher.git prism-src
cd prism-src

# Apply patches
git apply ../patches/0001-ownsMinecraft-always-true.patch
git apply ../patches/0002-anyAccountIsValid-always-true.patch
git apply ../patches/0003-decideLaunchMode-skip-auth.patch

# Build per Prism's build docs
cmake --preset linux && cmake --build --preset linux
```

## Manual workflow options

Trigger the workflow manually from the **Actions** tab.

- `force_version`: build a specific upstream tag (for example `10.0.5`)
- `force_rebuild`: rebuild even if a release for that tag already exists

Use `force_rebuild` when only the patches changed and you want to rebuild the current release tag.
