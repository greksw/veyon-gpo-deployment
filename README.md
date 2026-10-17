# Veyon GPO Deployment — PowerShell case study

[Русская документация](README.ru.md) · [Changelog](CHANGELOG.md) · [История версий](CHANGELOG.ru.md)

A reproducible **source-study and deployment pattern**, based on an October 2026 Windows workstation pilot: a GPO Scheduled Task running as `NT AUTHORITY\SYSTEM`, a pinned Veyon 4.11.3 installer, SHA256 and Authenticode checks, version and configuration validation, firewall scoping and a recovery-aware temporary TCP/11100 deny rule.

**Do not run the templates unmodified.** The repository is an **anonymized adaptation**, not the original production script. All `history/` versions have invalid hash placeholders and RFC 5737 documentation IPs. Even `-WhatIfCheck` must not be treated as a completely side-effect-free execution: the code can create local folders or ACLs while gathering its baseline.

## Files

- `scripts/Install-Veyon.ps1` — current v1.3 **public example** (not deployment-ready).
- `scripts/Veyon-Diagnostics.ps1` — read-only local/DNS/TCP/Task Scheduler diagnostics.
- `history/Install-Veyon-v1.0.sample.ps1` … `v1.3.sample.ps1` — sanitized historical stages. **Legacy examples, not recommended for direct deployment.**
- `examples/manifest.example.json` — illustrative artifact manifest and allowlist, **not actual checksums**.
- `examples/config.example.json` — **partial schematic** of the Veyon configuration checks, not a full Veyon export suitable for `config import`.
- `docs/gpo-setup.md` — setup/verification checklist.

## Architecture

```text
AD GPO / GPP Scheduled Task
        -> LOCAL SYSTEM
        -> PowerShell deployment script
        -> central package + JSON manifest
        -> SHA256 and Authenticode
        -> local cache -> Veyon Service
        -> restricted firewall TCP/11100 -> verification
```

Scope: **Windows 10/11 workstations only**; the script skips Windows Server and domain controllers. It installs the Veyon service **without Veyon Master**. It is **not a generic auto-updater**: an already installed version other than the pinned version triggers an error and a manual review.

## Security boundaries and gotchas

- `-WhatIfCheck` is an application-specific option, **not** native PowerShell `-WhatIf` semantics. It performs validations and may change local diagnostic folder/ACL state.
- The deployment script refuses to proceed while example domain/hash placeholders are present. Replace placeholders only after security review, using verified installer artifacts, a least-privilege AD operator group and approved management addresses.
- Do not commit a live Veyon export if it contains `Core/InstallationID`, password material or any other environment identifiers. The config example here is **not** a complete deployable export.
- An installer Authenticode signature is checked; pinned signer-subject matching is not a replacement for verifying the actual publisher/certificate policy.
- An aborted installation can intentionally leave a restrictive TCP/11100 deny rule. It should only be removed through a verified recovery procedure.
- Windows authentication of domain-joined targets and non-domain controllers belongs to a separate Veyon troubleshooting case. Matching local/domain credentials in a lab is not an AD trust relationship and is not a password reuse recommendation.

## Related notes

- [RU: Veyon authentication and deployment case](https://notes.shandrov.ru/ru/notes/windows/veyon-workgroup-ad-authentication-gpo-diagnostics/)
- [EN: Veyon authentication and deployment case](https://notes.shandrov.ru/notes/windows/veyon-workgroup-ad-authentication-gpo-diagnostics/)

## Provenance

The changelog records **changes visible in the supplied scripts**, not dates, deployment coverage or production validation not contained in those files. v1.2 and the separately supplied `Install-Veyon.ps1` were byte-identical. The production v1.3 header still said `pilot v1.2`; only the *public example* corrects that.

## License

No open-source license has yet been selected. Do not assume unrestricted reuse or redistribution until a license is explicitly chosen.