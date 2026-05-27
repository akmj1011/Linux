# RHEL – DNF update troubleshooting (GPG check failure / cache metadata issue)

## System
- OS: Red Hat Enterprise Linux
- Package manager: dnf

## Problem

While installing Python packages and running `dnf update`, the operation failed due to a GPG signature verification error.

The package manager was unable to verify package signatures, which blocked package installation and system updates.

I investigated whether the issue was caused by:
- stale or corrupted dnf metadata
- cached package corruption
- outdated release package metadata
- repository configuration issues

---

## Troubleshooting steps

### 1. Clear downloaded RPM packages

```bash
dnf clean packages
```

Why:

Cleared previously downloaded RPM packages in case corrupted package files were causing the failure.

---

### 2. Verify RHEL release package state

```bash
rpm -q redhat-release redhat-release-eula redhat-release-notes || true
```

Why:

Validated that core RHEL release packages were installed correctly.

---

### 3. Clear all dnf metadata and cache

```bash
sudo dnf clean all
```

Why:

Removed cached repository metadata and package cache.

---

### 4. Remove remaining dnf cache manually

```bash
sudo rm -rf /var/cache/dnf
```

Why:

Force removed leftover cache files after `dnf clean all`.

Used in case stale metadata remained on disk.

---

### 5. Verify enabled repositories

```bash
sudo subscription-manager repos --list-enabled
```

Why:

Verified that the correct Red Hat repositories were enabled and the system subscription was valid.

---

### 6. Check disk space

```bash
df -HT
```

Why:

Confirmed sufficient disk space was available for downloads and updates.

This ruled out incomplete package downloads caused by low disk space.

---

### 7. Refresh release packages while bypassing GPG verification temporarily

```bash
sudo dnf update --refresh --nogpgcheck redhat-release redhat-release-eula
```

Why:

The original failure occurred because GPG signature verification was failing during package installation.

`--nogpgcheck` was used temporarily during troubleshooting to bypass signature validation and refresh the release packages and repository metadata.

This allowed package metadata to refresh successfully before retrying normal updates.

---

### 8. Re-verify release packages

```bash
rpm -q redhat-release redhat-release-eula
```

Why:

Confirmed that release packages updated successfully after refresh.

---

### 9. Rebuild dnf metadata cache

```bash
sudo dnf makecache
```

Why:

Rebuilt repository metadata cache from enabled repositories using fresh metadata.

---

### 10. Retry full system update

```bash
sudo dnf update
```

Why:

Retested package management normally after cache cleanup and release package refresh.

Update completed successfully.

---

## Root cause

The issue was caused by GPG signature verification failures while installing Python-related packages with `dnf`.

Likely contributing causes:
- stale or corrupted dnf cache
- outdated repository metadata
- release package metadata requiring refresh

Refreshing the release packages and rebuilding the dnf metadata cache resolved the issue.

---

## Resolution

Issue resolved after:

- clearing cached RPM packages
- clearing dnf metadata cache
- manually removing `/var/cache/dnf`
- verifying enabled repositories
- refreshing `redhat-release` packages
- rebuilding dnf metadata cache
- retrying package update normally

After these steps:

```bash
sudo dnf update
```

completed successfully without errors.

---

## What I learned

- GPG signature verification failures can sometimes be related to stale repository metadata rather than invalid packages.
- `dnf clean all` may not always remove everything, and manual cleanup of `/var/cache/dnf` can help.
- `subscription-manager repos --list-enabled` is useful to validate repository access during troubleshooting.
- Temporarily using `--nogpgcheck` can help refresh package metadata during debugging, but normal package operations should be re-tested afterward with GPG validation enabled.
- Verifying `redhat-release` package state is useful when troubleshooting repository or update failures on RHEL.
