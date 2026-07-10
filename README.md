# 🕵️ Advanced JavaScript Infostealer – Full Technical Analysis

<div align="center">

![Severity](https://img.shields.io/badge/severity-critical-red?style=for-the-badge)
![Type](https://img.shields.io/badge/type-infostealer-orange?style=for-the-badge)
![Obfuscation](https://img.shields.io/badge/obfuscation-heavy-black?style=for-the-badge)
![Purpose](https://img.shields.io/badge/purpose-educational_only-blue?style=for-the-badge)

**A detailed forensic dissection of a real-world, multi-layered browser-based data-harvesting script.**

> *Intended strictly for security researchers, analysts, and educators working on client-side threat defence.*

</div>

---

## ⚠️ CRITICAL WARNING

> **DO NOT EXECUTE THIS CODE** on any system that contains sensitive data or is connected to a production network.

### This script will:

- 📋 Read your **clipboard, cookies, localStorage, and sessionStorage**
- 📹 Attempt to capture your **screen and camera**
- ⌨️ Log your **keystrokes and mouse movements**
- 🎮 Enumerate **USB, MIDI, GPU, and gamepad devices**
- 📍 Leak your **public and local IP addresses**
- 📤 Send **everything encrypted** to a remote attacker-controlled server

### Safe Analysis Protocol

✅ Use an **air-gapped, isolated virtual machine** with:
- No personal accounts or credentials
- Monitored network traffic
- No internet connectivity to production systems

---

## 📑 Table of Contents

1. [Overview](#overview)
2. [Deobfuscation & Configuration](#deobfuscation--configuration)
3. [Collection Modules – Full Reference](#collection-modules--full-reference)
4. [Data Exfiltration Architecture](#data-exfiltration-architecture)
5. [Execution Flow & Persistence](#execution-flow--persistence)
6. [Auxiliary XLSX Parser (Phishing Context)](#auxiliary-xlsx-parser-phishing-context)
7. [Defensive Countermeasures](#defensive-countermeasures)
8. [Detection Opportunities for Blue Teams](#detection-opportunities-for-blue-teams)
9. [Legal & Ethical Notice](#legal--ethical-notice)
10. [Conclusion](#conclusion)

---

## Overview

This repository contains the **fully reconstructed and annotated analysis** of a sophisticated JavaScript infostealer, typically embedded in phishing landing pages or compromised legitimate websites.

### Key Capabilities

| Capability | Details |
|---|---|
| **Collection Modules** | 25+ individual data-collection modules (configurable) |
| **Obfuscation** | Layered encoding with string encoding and variable renaming |
| **Encryption** | Custom XOR cipher for all transmitted payloads |
| **Real-Time Listeners** | Event hooks for clipboard, battery, network, sensors |
| **Visual Captures** | Screen & camera screenshots as JPEG images |
| **Self-Defence** | Pauses collection if WebDriver or ad-blocker detected |

---

## Deobfuscation & Configuration

The script employs two primary layers of obfuscation to evade analysis and detection.

### 1. String & Variable Obfuscation

```javascript
// Example: Base64-encoded C2 endpoint
atob("aHR0cHM6Ly95b3VyLXByb3h5LmNvbS9hcGk=")
// Decodes to: https://your-proxy.com/api

// Meaningful names are replaced with short random identifiers
_0x3c4d, _0x5e6f, _0x2e4f, _0x18a2
```

- **C2 proxy URL**: Stored as base64, decoded at runtime
- **Variable names**: Short, random identifiers (e.g., `_0x3c4d`, `_0x5e6f`)
- **Function names**: Obfuscated with minimal parameters

### 2. XOR Encryption

All data exfiltrated from the browser uses a simple XOR cipher with hardcoded key `x-s-k`:

```javascript
function encrypt(payload, key) {
  const json = JSON.stringify(payload);
  let result = '';

  for (let i = 0; i < json.length; i++) {
    result += String.fromCharCode(
      json.charCodeAt(i) ^ key.charCodeAt(i % key.length)
    );
  }

  return btoa(result);  // Base64 encode
}
```

**Flow**: Plaintext JSON → XOR encrypt → Base64 encode → Transmit

### 3. Central Configuration Object

The entire behavior is controlled by a single config object (`_0x3c4d`):

| Field | Purpose | Example |
|-------|---------|---------|
| `w` | WebSocket C2 endpoint | `wss://s.0/3` |
| `a` | HTTP API proxy endpoint | `https://your-proxy.com/api` |
| `t.e` | Enable Telegram exfiltration | `true` / `false` |
| `t.b` | Telegram bot token | `123456:ABC...` |
| `t.c` | Telegram chat ID | `-1001234567890` |
| `m` | Collection modules map | See collection modules table |
| `i.b` | Base loop delay (ms) | `120000` (2 min) |
| `i.j` | Random jitter (ms) | `2000` |
| `i.h` | High-sensitivity cooldown (ms) | `300000` (5 min) |
| `k` | XOR encryption key | `"x-s-k"` |

**Design benefit**: Attacker can fine-tune sensors, timing, and exfiltration method without redeploying code.

---

## Collection Modules – Full Reference

All modules are controlled via `_0x3c4d.m` flags and implement smart caching to avoid redundant API calls within cooldown windows.

### Module Matrix

| Flag | Module | Data Collected | Live Updates | HTTPS Only |
|------|--------|----------------|--------------|------------|
| `p` | **Permissions** | Camera, geolocation, microphone, notifications, MIDI, USB permissions state | ❌ | ❌ |
| `i` | **IP & Geolocation** | Public IP, city, region, country, ISP, coordinates, timezone, ASN | ❌ | ❌ |
| `d` | **Device Fingerprint** | User-agent, platform, languages, screen size, color depth, timezone, hardware concurrency, memory, touch support, WebDriver flag | ❌ | ❌ |
| `f` | **Canvas/WebGL/Audio** | Canvas image, WebGL renderer & vendor, AudioContext sample rate, fonts, WebAssembly support | ❌ | ❌ |
| `g` | **GPU** | GPU device name, vendor, architecture via `navigator.gpu` | ❌ | ❌ |
| `c` | **Navigation** | Visited URLs from performance entries | ❌ | ❌ |
| `geo` | **High-Accuracy GPS** | Latitude, longitude, accuracy; reverse-geocoded address | ✅ on new fix | ✅ |
| `b` | **Battery** | Battery level (%), charging status | ✅ on change | ❌ |
| `n` | **Network** | Effective type (4g, etc.), downlink, RTT | ✅ on change | ❌ |
| `s` | **Sensors** | Accelerometer (x,y,z), gyroscope (alpha, beta, gamma) | ❌ one-time | ❌ |
| `cl` | **Clipboard** | Current clipboard text | ✅ on paste | ✅ |
| `st` | **Storage** | Cookies, localStorage, sessionStorage (full dump) | ❌ | ❌ |
| `w` | **WebRTC** | Local IP addresses via ICE candidates | ✅ per candidate | ❌ |
| `int` | **Interaction** | Up to 10 keystrokes & mouse coordinates with timestamps | ✅ each event | ❌ |
| `mic` | **Microphone Activity** | Audio analyser data length (not raw audio) | ❌ | ✅ |
| `sc` | **Screen Capture** | JPEG screenshot of entire screen | ❌ | ✅ |
| `cam` | **Camera Capture** | Camera availability, count, snapshot from user-facing camera | ❌ | ✅ |
| `gp` | **Gamepad** | Connected gamepad list (id, button count) | ❌ | ❌ |
| `mi` | **MIDI** | Names of connected MIDI inputs | ❌ | ❌ |
| `u` | **USB Devices** | Vendor ID and product name | ❌ | ❌ |
| `x` | **WebXR** | Immersive-VR support flag | ❌ | ❌ |
| `h` | **History** | `window.history.length` | ❌ | ❌ |
| `e` | **Browser Extensions** | Detects uBlock Origin & AdBlock via extension probing | ❌ | ❌ |

### HTTPS Context Requirements

Modules marked with ✅ in **HTTPS Only** require:

```javascript
window.location.protocol === "https:"
```

In insecure HTTP contexts, these modules are silently skipped with a warning appended to logs.

---

## Data Exfiltration Architecture

The stealer uses a **multi-channel exfiltration strategy** to ensure redundancy and bypass network restrictions.

### 1. WebSocket (Primary, Real-Time) 🔴

```
Persistent Connection
│
├─ Endpoint: _0x3c4d.w (e.g., wss://s.0/3)
│
├─ All collected data XOR-encrypted
│
├─ Sent as soon as available
│
└─ Auto-reconnect with exponential backoff on failure
```

**Characteristics**:
- Low-latency, full-duplex communication
- Ideal for real-time event streaming (clipboard, interactions)
- Persistent connection survives across page navigations

### 2. HTTP POST (Fallback / Bulk) 🟠

```
POST Request
│
├─ Triggered when: t.e = false (Telegram disabled)
│
├─ Endpoint: _0x3c4d.a (e.g., https://your-proxy.com/api)
│
├─ Full encrypted payload in request body
│
├─ Spoofed User-Agent: Mozilla/5.0 (Windows NT 10.0)
│
└─ Response: ACK or task updates
```

**Characteristics**:
- Works over standard HTTPS
- Simpler firewall compatibility
- Bulk data transfer every collection interval

### 3. Telegram Bot API (Alternative) 🟡

```
sendMessage
│
├─ When: t.e = true
│
├─ Max 4096 chars per message
│
├─ Human-readable summary format
│
└─ Bot Token: t.b, Chat ID: t.c


sendPhoto (for media)
│
├─ Screen captures
│
├─ Camera snapshots
│
└─ Filename: timestamp-based
```

**Characteristics**:
- Highly evasive (uses legitimate Telegram API)
- Two-factor exfiltration: messages + images
- No rate limiting from attacker perspective

### Encryption in Transit

All payloads are encrypted **even over HTTPS** using XOR + Base64:

```
Plaintext JSON
     │
     ▼
   XOR (key: "x-s-k")
     │
     ▼
 Base64 Encode
     │
     ▼
  Transmit
     │
     ▼
C2 Server Decryption
```

---

## Execution Flow & Persistence

### Lifecycle Sequence

```
┌─────────────────────────────────────────────────────────────┐
│                      Page Load / Injection                  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
           ┌─────────────────────┐
           │    _0xb1c2()        │  Open WebSocket Connection
           │   (Connect to C2)   │
           └────────┬────────────┘
                    │
             [WS Ready Event]
                    │
                    ▼
           ┌─────────────────────┐
           │    _0xd3e4()        │  Collect All Enabled Modules
           │  (Respect cooldowns)│
           └────────┬────────────┘
                    │
                    ▼
           ┌─────────────────────┐
           │   _0x2e4f()         │  XOR Encrypt + Base64
           │  (Aggregate Data)   │
           └────────┬────────────┘
                    │
        ┌───────────┴───────────┐
        │                       │
        ▼                       ▼
   Send via WS          Send via HTTP POST
        │                  or Telegram API
        │                       │
        └───────────┬───────────┘
                    │
                    ▼
           ┌─────────────────────┐
           │   _0x18a2()         │  SetTimeout Loop
           │  (base + jitter)    │  base: 120s + jitter: 2s
           └────────┬────────────┘
                    │
        ┌───────────▼───────────┐
        │                       │
   ✅ Loop Continues       ❌ Stop If:
   (unless detected)       • navigator.webdriver = true
                           • uBlock Origin detected
                           • AdBlock detected
```

### Timing Configuration

| Parameter | Value | Purpose |
|-----------|-------|---------|
| **Base Interval** | 120,000 ms (2 min) | Primary collection loop delay |
| **Random Jitter** | ±2,000 ms | Avoid pattern detection |
| **High-Sensitivity Cooldown** | 300,000 ms (5 min) | Resource-intensive modules (camera, screen, geolocation) |

### Evasion Checks

```javascript
// Sandbox Detection
if (navigator.webdriver === true) {
  // Stop collection permanently
  collection_loop.stop();
}

// Ad-Blocker Detection (via extension probing)
if (extensionModule.detectAdBlocker()) {
  // Pause collection
  collection_loop.pause();
}
```

These checks help the malware avoid security researcher sandboxes and user-protected browsers.

---

## Auxiliary XLSX Parser (Phishing Context)

The code includes a standalone `loadFileData()` function that parses Excel (.xlsx) files using the SheetJS library. **This is not part of the infostealer itself** but indicates phishing deployment context.

### Functionality

```
loadFileData(base64EncodedXLSX)
  │
  ├─ Decode base64 → Binary
  ├─ Parse XLSX → JSON
  ├─ Filter blank rows
  ├─ Detect header row heuristically
  └─ Return as CSV string
```

### Phishing Implications

This suggests the malicious script is deployed on pages that:
- Harvest uploaded spreadsheet documents
- Extract financial or credential data
- Process employee/client lists
- Capture sensitive tabular information

**Common use cases**:
- Invoice data extraction
- Payroll information harvesting
- Client contact list theft

---

## Defensive Countermeasures

Organizations and individuals can mitigate this threat through browser hardening, network controls, and user education.

### 🔒 Browser Hardening

| Setting | Recommendation | Impact |
|---------|----------------|--------|
| **JavaScript** | Disable on untrusted sites (use uBlock Origin / NoScript) | Blocks inline script execution |
| **Permissions** | Block clipboard, camera, microphone, sensors, location by default | Denies sensor access entirely |
| **WebRTC** | Disable via browser flags or extensions (`chrome://flags/#disable-webrtc`) | Prevents IP leak via ICE candidates |
| **3rd-Party Cookies** | Block entirely | Reduces cross-site tracking |
| **GPU / USB / MIDI** | Never grant to unknown sites | Requires explicit user gesture |
| **WebDriver** | Ensure not exposed in normal browsing profiles | Hides automation signatures |

### 🛡️ Content Security Policy (CSP)

A strict CSP can **significantly reduce attack surface**:

```
Content-Security-Policy:
  default-src 'self';
  connect-src 'self';
  script-src 'self';
  img-src 'self' data:;
  style-src 'self' 'unsafe-inline';
```

**What this blocks**:
- ✅ WebSocket connections to attacker domains
- ✅ HTTP POST to external APIs
- ✅ Telegram API calls
- ⚠️ **Caveat**: If script is already inline, CSP must use nonce/hash approach

### 👤 User Awareness Training

| Behavior | Risk | Recommendation |
|----------|------|----------------|
| Clicking suspicious email links | **CRITICAL** | Never click shortened URLs or unfamiliar domains |
| Granting permissions instantly | **HIGH** | Any permission request should be treated with suspicion |
| Using same browser for sensitive work | **MEDIUM** | Segregate banking/finance to a hardened browser |
| Ignoring permission prompts | **MEDIUM** | Read what permissions are requested before approving |
| Infrequent cache/cookie clearing | **LOW** | Clear browser storage after untrusted browsing |

---

## Detection Opportunities for Blue Teams

Security operations teams should monitor for the following indicators across network, host, and code levels.

### 🌐 Network IOCs (Indicators of Compromise)

#### WebSocket Connections
```
Pattern:  wss:// connections to short or unknown domains
Example:  wss://s.0/3, wss://x.co/ws
Severity: 🔴 CRITICAL
Action:   Block + Alert
```

#### Suspicious HTTP POST Requests
```
Pattern:  POST /api with base64-encoded body
Example:  POST https://attacker.com/api
Body:     base64-encoded XOR-encrypted JSON
Severity: 🔴 CRITICAL
Action:   Block + Inspect
```

#### Telegram API from Browser
```
Pattern:  api.telegram.org requests from browser user-agent
Severity: 🔴 CRITICAL (highly anomalous)
Action:   Immediate investigation + isolation
Endpoint: api.telegram.org/botXXX/sendMessage
```

### 💻 Host-Based Detection

#### Browser API Abuse Signatures

| Signature | Indicator |
|-----------|-----------|
| `getDisplayMedia()` + `getUserMedia()` | Unauthorized screen/camera access attempt |
| Rapid API enumeration | Multiple permission queries in < 1 second |
| `navigator.webdriver` check | Sandbox evasion attempt |
| Extension probing | Scanning for security tools |
| Clipboard access loops | High-frequency clipboard reads |
| Storage enumeration | Enumerating cookies + localStorage + sessionStorage |

#### JavaScript Behavioral Indicators

```javascript
// Red flag: Permission enumeration
Permissions.query({ name: 'camera' })
Permissions.query({ name: 'microphone' })
Permissions.query({ name: 'geolocation' })
  // All within same execution context

// Red flag: WebDriver detection + abort
if (navigator.webdriver === true) { stop(); }

// Red flag: Extension detection
fetch('chrome-extension://...')
```

### 📦 Code-Level Signatures

#### XOR Encryption Pattern

```javascript
// Characteristic pattern:
String.fromCharCode(charCode ^ key.charCodeAt(index % length))
// Followed by:
btoa(result)  // Base64 encode
```

#### YARA Detection Rule

```yara
rule JavaScript_Infostealer_XOR_Encryption {
  meta:
    description = "Detects XOR-encrypted JavaScript infostealer"
    severity     = "critical"

  strings:
    $xor_op      = "fromCharCode" and "^" and "btoa"
    $key         = "x-s-k" wide ascii
    $obfuscation = /_0x[0-9a-f]{4,}/

  condition:
    $xor_op and $key and (#obfuscation > 5)
}
```

#### Deobfuscation Checklist

- [ ] Decode all base64 strings
- [ ] Replace obfuscated variable names with meaningful ones
- [ ] Reconstruct control flow
- [ ] Identify C2 endpoints
- [ ] Extract XOR encryption key
- [ ] Correlate with known IoCs

---

## Legal & Ethical Notice

### ⚖️ Important Disclaimer

This analysis is provided **solely for educational and defensive security research**.

The code described herein is **malicious**. Its use against any system without explicit, written authorization is:
- 🚫 **Illegal** under computer fraud and unauthorized access laws
- 🚫 **Unethical** and violates professional security standards
- 🚫 **Subject to criminal and civil prosecution**

### Your Obligations

By accessing this document, you agree to:

1. ✅ **NOT deploy, distribute, or modify** the script for any unauthorized purpose
2. ✅ **Use information exclusively** to enhance your organization's detection and prevention capabilities
3. ✅ **Handle the original code** with the same precautions as any live malware sample
4. ✅ **Report findings** to appropriate security teams and law enforcement when warranted
5. ✅ **Consult legal counsel** before conducting any analysis or testing

### Liability

The author(s) of this analysis assume **no liability** for misuse of this information. If you are unsure how to safely handle this material, consult your:
- 📋 Legal team
- 👔 Security policy officer
- 🏢 Organizational compliance department

---

## Conclusion

This JavaScript infostealer exemplifies how modern, API-rich browsers can be weaponized to extract an alarming breadth of personal and device information with just a few hundred lines of obfuscated code.

### Key Takeaways

#### For Security Researchers
- Modern browser APIs provide an unprecedented attack surface
- Obfuscation + encryption can be effective but is not impenetrable
- Multi-channel exfiltration provides attacker redundancy

#### For Defense Teams
- Implement strict CSP policies across all domains
- Monitor for WebSocket connections to unknown endpoints
- Train users to recognize permission request patterns
- Segment sensitive workflows to hardened browser profiles

#### For System Administrators
- Deploy network-level detection for Telegram API anomalies
- Block WebRTC IP leaks via browser policies
- Enforce ad-blocker and security extension deployment
- Monitor for rapid API enumeration patterns

### Broader Implications

Studying this script provides valuable insights into:

1. **Current state of client-side attack tooling** – highly modular and configurable
2. **Importance of strict browser permission models** – users need better defaults
3. **Need for robust network and endpoint detection** – multiple layers required
4. **Value of user education** – technical controls alone are insufficient

---

## Further Resources

- 📚 Open issues in the repository to contribute detection rules and indicators
- 🔍 Share findings responsibly with the security community
- 🛡️ Contribute hardening techniques and defensive strategies

**Remember: Use this knowledge to protect users and systems, never to harm them.**

---

<div align="center">

**Last Updated**: 2026
**Repository**: [browser-fingerprint-collector](https://github.com/Ali-hey-0/browser-fingerprint-collector)
**Purpose**: Educational Security Research Only

⚠️ **Handle with care. Treat as live malware.** ⚠️

</div>
