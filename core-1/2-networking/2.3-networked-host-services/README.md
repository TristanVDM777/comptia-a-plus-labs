# Windows Time and NTP Troubleshooting Lab

## Objective

**CompTIA A+ 220-1201 Core 1 - Objective 2.3: Summarize services provided by networked hosts**

This lab demonstrates how to diagnose and restore Windows time synchronization using the Windows Time service and Network Time Protocol (NTP).

## Scenario

Windows was not synchronized with its configured network time source. Initial checks either failed while the Windows Time service was stopped or reported the local hardware clock instead of an NTP server.

Sanitized symptoms included:

```text
Source: Local CMOS Clock
Leap Indicator: 3 (not synchronized)
Stratum: 0 (unspecified)
Last Successful Sync Time: unspecified
```

## Troubleshooting Process

### 1. Check the Windows Time service

```powershell
Get-Service W32Time
```

The `W32Time` service was stopped. Starting it from a standard PowerShell session was not permitted, so PowerShell was reopened with administrator privileges.

```powershell
Start-Service W32Time
Get-Service W32Time
```

The service then reported a `Running` state.

### 2. Check the current time source and synchronization state

```powershell
w32tm /query /source
w32tm /query /status
```

Although the service was now running, Windows still reported `Local CMOS Clock` and had not completed a successful network synchronization.

### 3. Inspect the configured NTP peer and client settings

```powershell
w32tm /query /peers
w32tm /query /configuration
```

The inspection confirmed that:

- the Windows NTP client was enabled;
- the client type was `NTP`;
- `time.windows.com,0x9` was configured as the peer; and
- the peer was in a `Pending` state.

This ruled out a missing NTP peer configuration. The remaining issue was that the client had not yet obtained usable time data after the service started.

### 4. Force peer rediscovery and resynchronization

```powershell
w32tm /resync /rediscover
```

### 5. Verify the result

```powershell
w32tm /query /source
w32tm /query /status
```

Sanitized verification output:

```text
Source: time.windows.com,0x9
Leap Indicator: 0 (no warning)
Stratum: 5 (secondary reference - synchronized by NTP)
Last Successful Sync Time: [confirmed; exact timestamp omitted]
```

## Root Cause and Issue Progression

The problem developed in stages:

1. Time queries could not operate normally because the Windows Time service was stopped.
2. Administrative privileges were required to start the service.
3. Once running, the service had a valid NTP configuration but had not synchronized; it remained on the local CMOS clock while the configured peer was pending.
4. Rediscovering the peer and forcing a resync completed synchronization with `time.windows.com`.

No NTP server change or permanent configuration change was required.

## Result

Windows moved from an unsynchronized local-clock state to a healthy NTP-synchronized state using the existing `time.windows.com` configuration.

## What I Learned

- NTP keeps clocks consistent across networked systems, which supports reliable log correlation, authentication, certificates, scheduled tasks, and troubleshooting.
- A configured NTP server does not prove that synchronization is currently working; the source, status, and last successful synchronization must be verified.
- Starting a stopped service may restore the required component without immediately completing synchronization.
- Service state, permissions, configuration, peer state, corrective action, and final verification should be checked separately.
- `w32tm` provides built-in tools for inspecting and troubleshooting Windows time synchronization.

## Skills Demonstrated

- Windows service inspection and recovery
- Administrator privilege awareness
- NTP client and peer inspection
- Command-line troubleshooting with `w32tm`
- Root-cause analysis through staged verification
- Privacy-conscious technical documentation

## Privacy Note

Public evidence was limited to the technical details needed to demonstrate the result. The computer name, user information, source-server IP address, and exact synchronization timestamp were omitted.
