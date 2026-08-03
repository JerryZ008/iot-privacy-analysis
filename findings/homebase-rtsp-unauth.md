# Critical Finding — Unauthenticated RTSP Access on Eufy HomeBase 2
 
**Severity:** Critical (CVSS 3.1 base 9.8)
**Class:** Missing Authentication for Critical Function (CWE-306)
**Status:** Verified on own device · reported to vendor · mitigated
 
---
 
## Summary
 
During empirical validation of a home IoT privacy assessment, the RTSP service on
port 554 of a Eufy HomeBase 2 (surveillance hub) was found to accept connections
and expose all RTSP methods **without any authentication**. Any device on the local
network can reach the streaming interface unauthenticated, creating a path to
unauthorised access of camera video streams.
 
All testing was performed on hardware I own, on my own network, and the finding was
reported to the vendor with the mitigations below applied.
 
---
 
## Verification
 
Testing was done with a single `curl` request against the RTSP port. A correctly
secured RTSP server should challenge with `401 Unauthorized`; this device returned
`200 OK` and advertised its full method set.
 
```bash
curl -i rtsp://<HOMEBASE_IP>:554/
```
 
Observed response:
 
```
< RTSP/1.0 200 OK
< CSeq: 1
< Public: OPTIONS, DESCRIBE, SETUP, TEARDOWN, PLAY, GET_PARAMETER
```
 
Expected response from a secured server:
 
```
RTSP/1.0 401 Unauthorized
WWW-Authenticate: Digest realm="IP Camera"
```
 
The `200 OK` with a full public method list (including `DESCRIBE`, `SETUP`, `PLAY`)
confirms the service is reachable and controllable without credentials.
 
---
 
## Risk Assessment
 
**CVSS 3.1 vector:** `AV:A/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N`
 
| Factor | Rating | Rationale |
|---|---|---|
| Attack Vector | Adjacent (A) | Requires local-network access (e.g. Wi-Fi) |
| Attack Complexity | Low (L) | No special conditions |
| Privileges Required | None (N) | No authentication whatsoever |
| User Interaction | None (N) | Fully passive to the victim |
| Confidentiality | High (H) | Potential exposure of live/recorded video |
 
The base score sits at 9.8 when the confidentiality impact of a surveillance feed is
taken into account.
 
---
 
## Impact
 
An attacker with local-network access (a guest, a neighbour with the Wi-Fi password,
or a device already compromised on the LAN) could potentially:
 
- Access live or recorded video streams without credentials
- Reconstruct household activity patterns and occupancy
- Combine this with other metadata (e.g. smart-lock timing) to build a high-confidence
  behavioural profile
This finding materially raised the project's overall risk assessment for the HomeBase,
and is the reason it — rather than the smart lock — became the environment's most
severe issue on empirical validation.
 
---
 
## Mitigations Applied
 
**Immediate (0–1 hour)**
- Block port 554 to the device at the router firewall
- Disable RTSP / local streaming in the vendor app where the option exists
- Isolate the device on a dedicated IoT VLAN with no lateral access to trusted devices
**Verification of fix**
 
```bash
# Port should now be filtered:
nc -zv <HOMEBASE_IP> 554        # expect: connection refused / timeout
 
# Or, if still reachable, auth should now be required:
curl -i rtsp://<HOMEBASE_IP>:554/   # expect: 401 Unauthorized
```
 
**Longer term**
- Firmware update check and vendor follow-up
- Move to devices supporting authenticated / encrypted streaming (RTSPS)
---
 
## Responsible Disclosure
 
The finding was reported to the vendor's security contact with the device model,
firmware version, reproduction steps and observed response. This is a known class of
issue for the product family — related public references:
 
- CVE-2020-13895 — Eufy RTSP unauthorised access (related class)
- CVE-2021-32934 — Eufy camera authentication bypass (related class)
---
 
## What This Demonstrates
 
- Turning a passive privacy assessment into active verification rather than stopping at assumptions
- Correctly scoring a real vulnerability with CVSS and mapping it to CWE
- Prioritising defensive mitigations and verifying the fix
- Following responsible-disclosure practice on owned hardware
---
 
*Finding produced during a coursework IoT security assessment, Master of Cyber
Security, UNSW Canberra (ADFA). All testing performed on owned devices.*
