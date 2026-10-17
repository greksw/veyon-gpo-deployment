# GPO deployment and verification checklist

This is a practical guide to reproducing the design, **not a ready-made production installer**.

## Setup

1. Use an administrative test OU with representative Windows 10/11 clients.
2. Store audited Veyon installer binaries and a complete, reviewed exported configuration in a read-restricted distribution path such as `\\ad.example.com\NETLOGON\Software\Veyon`. Control **write access** to the repository; otherwise a compromised administrator or SYSVOL write path could distribute malicious software.
3. Produce and independently verify SHA256 of x86, x64 installers and the exact config file. Substitute the three placeholder values in the public script and manifest; do not use sample hashes.
4. Review allowed management source ranges, security groups, Veyon Logon Authentication and Access Control policy with an explicit denial test.
5. Create a **computer-side** GPP Scheduled Task under `NT AUTHORITY\SYSTEM`, with highest available privileges and a command resembling:

```text
powershell.exe -NoProfile -NonInteractive -ExecutionPolicy Bypass -File "\\ad.example.com\NETLOGON\Software\Veyon\Scripts\Install-Veyon.ps1"
```

The command requires adapting to your actual UNC path. A policy-level execution bypass does not replace application control or script review.

## Pre-flight checks

```powershell
# On the target workstation:
gpupdate /target:computer /force
gpresult /scope computer /r
Get-ScheduledTask -TaskName 'Corp-Veyon-AutoDeploy' -ErrorAction SilentlyContinue
Get-ScheduledTaskInfo -TaskName 'Corp-Veyon-AutoDeploy' -ErrorAction SilentlyContinue
Get-Service VeyonService -ErrorAction SilentlyContinue
Get-NetFirewallRule | Where-Object DisplayName -match 'Veyon'
```

The task name is illustrative. The domain and the full exported Veyon config from the original case are not provided here.

## Runtime checks

- Expected version = 4.11.3.0, target service = `VeyonService`, inbound port TCP/11100.
- Log success alone is insufficient: compare installed version, service state, exported effective config and firewall allowlist.
- After failure, an active temporary deny rule may be the intended fail-safe. Review logs and config before removing it.
- Inspect PowerShell exit code and Task Scheduler `LastTaskResult`; a task listed as Ready says nothing about the last attempted installation.
- For connectivity issues distinguish DNS failure, TCP reachability, Windows Security 4625, Veyon authentication and Veyon Access Control denial.

## Version management

v1.3 **pins one Veyon product version**. A controlled version upgrade requires changing the chosen product version, installer artifacts, manifest, hashes, and assessing whether the older version should be removed/upgraded. Do not assume this sample is a generalized auto-update pipeline.

## Security review

Before public reuse, audit the exact Veyon configuration schema for the version in question, special characters in paths, CLI behavior and certificate issuer verification. The sample `examples/config.example.json` demonstrates only checked properties and must **not** be imported as a full configuration.
