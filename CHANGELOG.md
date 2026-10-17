# Changelog / История изменений

This changelog is reconstructed from **four source snapshots supplied by the operator**, not from Git history or a full production issue tracker. Every published script is an **anonymized adaptation**; the original binaries, secrets, identities and infrastructure information are absent.

## v1.3 — installer process reliability

- Replaced `Start-Process -Wait -PassThru` with `System.Diagnostics.ProcessStartInfo` / `System.Diagnostics.Process`.
- Disabled shell execution, hid the installer window and used an explicit working directory.
- Added PID logging, a **600-second timeout**, a termination attempt on timeout, and explicit exit-code handling.
- Changed installer command from `/S /NoMaster /ApplyConfig=...` to `/S /NoMaster`, retaining the subsequent explicit `veyon-cli config import`.
- The installed-version guard already existed in earlier versions: **the change is not an automatic upgrade feature**.
- Production file still had the stale comment `pilot v1.2`; fixed only in the public example.

## v1.2 — reliable effective configuration validation

- Switched from individual `veyon-cli config get` calls to exporting the effective configuration as JSON.
- Worked around the observed behavior where a Veyon 4.11.3 port read could be empty despite a correct effective port value.
- Added temporary export storage with explicit ACLs for SYSTEM and Administrators and cleanup in `finally`.
- Validated exact authorized AD operator group, restricted access setting, domain user groups, disabled automatic firewall exception and TCP/11100.
- Added warnings for mismatched exported values or export errors.
- Uploaded `Install-Veyon.ps1` is **byte-identical** to the uploaded v1.2 source.

## v1.1 — firewall normalization and recovery

- Normalized `/255.255.255.0` vs `/24` firewall remote-address notation before comparing allowlists.
- Detected an existing temporary TCP/11100 block from an interrupted deployment and verified that it is an inbound TCP deny rule for the expected port.
- Prevented the already-current fast path while the temporary block remained, requiring full verification before cleanup.
- Avoided creating a duplicate temporary block on a retry.

## v1.0 — baseline deployment

- Restricted scope to Windows client OS; required the SYSTEM identity for live deployment.
- Central manifest, hard-pinned SHA256 for installer and configuration, architecture-specific package selection and local-cache re-verification.
- Authenticode validation of the Veyon installer.
- Named mutex to prevent parallel executions.
- Checked version, service health, configuration and scoped Windows Firewall TCP/11100 rule.
- Temporary deny rule during fresh installation and conservative failure handling.
- Idempotency marker and timestamped operational logs.

## What is not established by these source files

- No proof of fleet-wide compliance or general production uptime.
- No proof that any of the four adapted scripts was independently executed on a Windows host after anonymization.
- No automated rollback of the Veyon installation to an earlier version.
- No tested support for upgrading from arbitrary installed Veyon versions.