# KernelSU-Next (vendored)

This directory is a copy of the `kernel/` directory of
[KernelSU-Next](https://github.com/KernelSU-Next/KernelSU-Next), branch
`legacy` (the non-GKI branch), commit `c81cdf51c3bb`
(`v3.4.0-legacy` plus 19 commits, including what was ported from dev 3.4.2).
`include/uapi` is a copy of the repository's top-level `uapi/` directory
(upstream links it in with a symlink), and `setup.sh` is left out.

The only local change is in `Kbuild`: upstream derives the version from the
git history of its own repository, which a vendored copy doesn't have, so the
values for that commit are pinned (`KSU_GIT_VERSION := 3024`,
`KSU_GIT_TAG := v3.4.0-legacy`, reported version 33313).

The kernel side is wired up in `drivers/Makefile`, `drivers/Kconfig` and
`arch/arm64/configs/vendor/RMX3511.config` (`CONFIG_KSU=y`, kprobes hooks).
`path_umount()` is backported in `fs/namespace.c`, so the build-time
`sed` fallbacks in `Kbuild` never touch the tree.

To update, replace the directory with a newer `kernel/` from the `legacy`
branch, copy `uapi/` to `include/uapi` again and redo the `Kbuild` pin.
