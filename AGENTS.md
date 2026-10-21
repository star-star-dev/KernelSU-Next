# AGENTS.md

## What this repository is

A fork of KernelSU-Next used to run the KernelSU-Next manager on legacy
kernels. The active branch is `legacy-susfs-v230`, which pairs the upstream
KernelSU-Next kernel code with SUSFS glue adapted to work on kernels as old as
Linux 4.14.

This repository holds the *kernel-side* module and the UAPI header. The kernel
tree that consumes it is `star-star-dev/M62-backport`, which includes this repo
as the `KernelSU/` submodule pinned to `legacy-susfs-v230`.

Layout:

- `kernel/` — the kernel module (Kbuild, supercall dispatch, SELinux hooks,
  SUSFS glue).
- `uapi/supercall.h` — the userspace-facing UAPI contract.
- `kernel/Kbuild` — where the version numbers are computed.

## Versioning

Three separate numbers, deliberately not tied to each other:

1. `KSU_VERSION` (`kernel/Kbuild`) — numeric versionCode the manager compares
   against. Derived from the git commit count plus a constant.
2. `KSU_VERSION_SUFFIX` (`kernel/Kbuild`) — fork iteration. Bump it by one
   whenever this branch is updated or repackaged so the reported version
   advances even when the upstream commit count does not. It is appended to
   `KSU_VERSION` and also to `KSU_VERSION_TAG`.
3. `KERNEL_SU_UAPI_VERSION` (`uapi/supercall.h`) — the UAPI contract version.
   The manager requires `kernelUAPIVersion == managerUAPIVersion` to be
   considered full-featured, so this must match the manager build. It is
   currently pinned to 5 for the v3.4.x manager; do not change it unless the
   manager is updated in lockstep.

SUSFS version is read at build time from the *kernel tree*, not this repo:
`kernel/Kbuild` and `kernel/Makefile` read `$(srctree)/include/linux/susfs.h`
and extract `SUSFS_VERSION`. That header lives in `M62-backport`. If the banner
shows a stale SUSFS version, check the kernel tree, not this repo.

## Rule: bump the fork version when adapting upstream changes

When you rebase or pull in new upstream KernelSU-Next commits onto
`legacy-susfs-v230`:

1. Re-apply the SUSFS glue and any legacy-kernel fixes.
2. Bump `KSU_VERSION_SUFFIX` by one.
3. Leave `KERNEL_SU_UAPI_VERSION` alone unless the manager is updated too.
4. Verify the kernel tree still builds.

Then, in `star-star-dev/M62-backport`, update the `KernelSU` submodule pointer
to the new commit here and bump the kernel/package version in the same change.
That repo's `AGENTS.md` documents the version scheme; the short version is that
the kernel version tracks the bundled SUSFS release, and CI fails if the two
drift apart.

## Conventions

- `kernel/Kbuild` prints build-time banners for the version, tag, and SUSFS
  version. Keep them working; they are the first thing to check when the
  manager reports a version mismatch.
- Do not push to `main`; work on a feature branch. The repository default
  branch is `legacy-susfs-v2`, and the active line of work is
  `legacy-susfs-v230`.
