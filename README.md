# PBRP build for Redmi Note 8 Pro (`begonia`)

GitHub Actions recipe that compiles **PBRP** (PitchBlack Recovery Project)
`recovery.img` for the Xiaomi Redmi Note 8 Pro, codename `begonia`.

Everything is derived from the device tree and PBRP sources published by
**SaiKrishna1504**.

## Branches

| Branch | Manifest | Device tree | Status |
|---|---|---|---|
| `main` | `manifest_pb@android-12.1` / `vendor_pb@pb-12.1` | `twrp-12.1` | built and verified |
| `android-14.0` | `manifest_pb@android-14.0` / `vendor_pb@pb-14.0` | `pb-14.0` | port in progress |

## What this builds

| | |
|---|---|
| PBRP manifest | `PitchBlackRecoveryProject/manifest_pb` @ `android-14.0` |
| Recovery source | `PitchBlackRecoveryProject/android_bootable_recovery` @ `android-14.0` |
| PBRP vendor config | `PitchBlackRecoveryProject/vendor_pb` @ `pb-14.0` |
| Device tree | `pacman-Sy/device_xiaomi_begonia-pbrp` @ `pb-14.0` (fork of `SaiKrishna1504/device_xiaomi_begonia-pbrp`) |
| Product / lunch | `pb_begonia` / `pb_begonia-eng` |
| Make target | `recoveryimage` |
| Output | `out/target/product/begonia/recovery.img` |

## Device tree branch selection

| Branch | Product file | Project |
|---|---|---|
| `pb-14.0` | `pb_begonia.mk` | **PBRP — used by this branch** |
| `twrp-12.1` | `pb_begonia.mk` | PBRP — used by `main` |
| `dynamic` | `omni_begonia.mk` | TWRP / OmniROM |
| `fbev2` | `omni_begonia.mk` | TWRP / OmniROM |

## Usage

1. **Actions → PBRP build (begonia) → Run workflow**
2. Options:
   - `makefile` — default `pb_begonia-eng`
   - `build_target` — default `recoveryimage`
   - `jobs` — default `4` (the runner has 4 vCPUs)
   - `ccache` — default `true`
   - `publish_release` — attach the image to a GitHub release
3. When the job goes green, download `pbrp-begonia-recovery` from the run's
   **Artifacts** section. `recovery.img` is the file to flash to the
   `recovery` partition.

## Upstream breakage worked around

`manifest_pb/pbrp-default.xml` on **both** `android-12.1` and `android-14.0`
declares:

```xml
<project path="vendor/utils" name="vendor_utils"
         remote="PitchBlackRecoveryProject" revision="pb" />
```

The `pb` branch of `PitchBlackRecoveryProject/Utils` **no longer exists** —
`master` now holds an unrelated project, and `git ls-remote` shows only
`master` and `refs/meta/config`. A plain `repo sync` therefore aborts with
revision-not-found.

`local_manifests/begonia.xml` removes that project. It only existed to
symlink `vendor/utils/pb_build.sh` → `vendor/pb/pb_build.sh`, a developer
helper script, and is not needed to compile the image. The same local
manifest also registers the `begonia` device tree so `repo sync` fetches it
along with everything else.

## Notes on the device tree

The tree is self-contained — it carries its own kernel, DTB, DTBO and
keymaster/gatekeeper blobs under `prebuilt/` and `recovery/root/`, so no
kernel build, proprietary vendor blobs, or hardware trees are needed.

`BoardConfig.mk` sets `ALLOW_MISSING_DEPENDENCIES`, `BUILD_BROKEN_DUP_RULES`
and `BUILD_BROKEN_ELF_PREBUILT_PRODUCT_COPY_FILES` to `true` to tolerate the
reduced `remove-minimal.xml` tree, and sets `PLATFORM_SECURITY_PATCH` to
`2099-12-31` to bypass AVB rollback checks.

## android-14.0 notes

- Upstream ships **no** begonia device tree for `android-14.0` (only
  `android-9.0`), so `pb-14.0` is a device-tree port, not a version bump.
- `remove-minimal.xml` on `android-14.0` keeps ~513 projects where
  `android-12.1` kept ~252, so the checkout is substantially larger. The
  workflow's disk-cleanup and post-sync trim steps exist for that reason.
- AOSP 14 builds against `prebuilts/jdk/jdk17` (present in the manifest);
  the workflow installs JDK 17 only so the system default matches.
- `vendor_utils` is still declared dead on `android-14.0`, so the
  `<remove-project>` workaround is still required.

## Repository layout

```
.github/workflows/build.yml   the CI recipe
local_manifests/begonia.xml   device tree + upstream fix
```