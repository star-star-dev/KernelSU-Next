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
   against its own. Derived from the git commit count plus a constant. The
   manager warns when the kernel reports a *higher* code than the manager, so
   this number must stay in the same range as the manager's, and slightly below
   it is the healthy state. Never append digits to it.
2. `KSU_VERSION_SUFFIX` (`kernel/Kbuild`) — fork iteration, derived from git as
   the number of commits since the nearest reachable tag. No manual bump: every
   commit past the branch's release tag raises it, and re-pointing the tag
   restarts it at zero. It is appended to `KSU_VERSION_TAG` only, never to
   `KSU_VERSION`. Override with `KSU_VERSION_SUFFIX=<n>` to pin it.
3. `KERNEL_SU_UAPI_VERSION` (`uapi/supercall.h`) — the UAPI contract version.
   The manager requires `kernelUAPIVersion == managerUAPIVersion` to be
   considered full-featured, so this must match the manager build. It is
   currently pinned to 5 for the v3.4.x manager; do not change it unless the
   manager is updated in lockstep.

## The reported tag must resolve from this branch

`KSU_VERSION_TAG` comes from `git describe --tags --abbrev=0`, which only
considers tags reachable from the current branch. Each branch therefore needs
its *own* tag series; a tag that exists on another branch will not be found.
This bit the `legacy-susfs-v230` branch: it was rebuilt independently from
`upstream/legacy`, so it did not descend from `v3.4.2-legacy`, and `describe`
fell back to the much older `v3.1.0-legacy` — which is why the manager showed
a kernel tag from a different release line.

Tag each release with `<manager version>-legacy-susfs-<branch>`, matching the
existing `legacy-susfs-v2` convention (`v3.2.0-legacy-susfs-v2`), e.g.
`v3.4.2-legacy-susfs-v230`. Create it on the release commit and push the tag
(`git push origin <tag>`); CI clones with tags, so `describe` will then resolve
correctly.

SUSFS version is read at build time from the *kernel tree*, not this repo:
`kernel/Kbuild` and `kernel/Makefile` read `$(srctree)/include/linux/susfs.h`
and extract `SUSFS_VERSION`. That header lives in `M62-backport`. If the banner
shows a stale SUSFS version, check the kernel tree, not this repo.

## Rule: no manual version bump when adapting upstream changes

When you rebase or pull in new upstream KernelSU-Next commits onto
`legacy-susfs-v230`:

1. Re-apply the SUSFS glue and any legacy-kernel fixes.
2. Leave `KSU_VERSION_SUFFIX` alone; it follows the commit count. Re-point the
   branch's release tag (`v3.4.2-legacy-susfs-v230`) only when you want the
   reported tag to restart its iteration.
3. Leave `KERNEL_SU_UAPI_VERSION` alone unless the manager is updated too.
4. Verify the kernel tree still builds.

Then, in `star-star-dev/M62-backport`, update the `KernelSU` submodule pointer
to the new commit here. That repo's `AGENTS.md` documents the version scheme;
its kernel version tracks the bundled SUSFS release and its patch level is a
commit count, so neither side needs a manual bump.

## Conventions

- `kernel/Kbuild` prints build-time banners for the version, tag, and SUSFS
  version. Keep them working; they are the first thing to check when the
  manager reports a version mismatch.
- Do not push to `main`; work on a feature branch. The repository default
  branch is `legacy-susfs-v2`, and the active line of work is
  `legacy-susfs-v230`.
- Never put AI/agent attribution in commits: no `Co-authored-by: openhands`,
  no `openhands@all-hands.dev` author/committer/tagger, no "generated with"
  trailers. Use the repo's normal identity
  (`star-star-dev <xingyuan53@qq.com>`). Automated attribution on a personal
  account risks the account being flagged, so keep history indistinguishable
  from the maintainer's own commits.
