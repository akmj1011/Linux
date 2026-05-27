# RHEL – DNF update troubleshooting (cache / metadata issue)

## System
- OS: Red Hat Enterprise Linux
- Package manager: dnf

## Problem

`dnf update` was failing during package refresh/update.

I suspected cached metadata corruption or an issue with release packages.

---

## Troubleshooting steps

### 1. Clear downloaded RPM packages

```bash
dnf clean packages
```

Why:
Clears previously downloaded RPM package files to remove potentially corrupted downloads.

---

### 2. Verify release package state

```bash
rpm -q redhat-release redhat-release-eula redhat-release-notes || true
```

Why:
Checks whether core RHEL release packages are installed correctly.

---

### 3. Clear all dnf metadata/cache

```bash
sudo dnf clean all
```

Why:
Removes cached repository metadata and package information.

---

### 4. Remove dnf cache manually

```bash
sudo rm -rf /var/cache/dnf
```

Why:
Force removes remaining cache files if `dnf clean all` does not fully clear them.

---

### 5. Verify enabled repositories

```bash
sudo subscription-manager repos --list-enabled
```

Why:
Confirms the required Red Hat repositories are enabled and subscription access is valid.

---

### 6. Check disk space

```bash
df -HT
```

Why:
Ensures enough disk space is available for package downloads and updates.

---

### 7. Refresh release packages

```bash
sudo dnf update --refresh --nogpgcheck redhat-release redhat-release-eula
```

Why:
Refreshes repository metadata and updates release packages.

Note:
`--nogpgcheck` was used temporarily during troubleshooting.

---

### 8. Re-verify package state

```bash
rpm -q redhat-release redhat-release-eula
```

Why:
Confirms release packages updated successfully.

---

### 9. Rebuild metadata cache

```bash
sudo dnf makecache
```

Why:
Downloads fresh metadata from enabled repositories.

---

### 10. Retry full system update

```bash
sudo dnf update
```

Why:
Confirms the issue is resolved with a normal update.

---

## Root cause

Likely stale or corrupted dnf cache/metadata, or outdated release package metadata.

---

## Resolution

Issue resolved after:
- clearing dnf cache
- validating repository configuration
- refreshing release packages
- rebuilding repository metadata cache

---

## What I learned

- `dnf clean all` removes most metadata, but `/var/cache/dnf` may still need manual cleanup.
- `subscription-manager repos --list-enabled` is useful to verify repository access.
- Checking `redhat-release` package health can help identify RHEL repo/version-related issues before a full update.
