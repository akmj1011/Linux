# Resolving GPG Check Error in RHEL

## Overview
This document describes the steps taken to troubleshoot and resolve a GPG signature verification error encountered while installing or updating packages using YUM/DNF on Red Hat Enterprise Linux (RHEL).

## Issue
During package installation or repository synchronization, the following error was observed:

```bash
GPG check FAILED
Public key for <package-name>.rpm is not installed
