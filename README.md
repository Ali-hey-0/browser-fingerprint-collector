
# 🕵️ Advanced JavaScript Infostealer – Full Technical Analysis

<p align="center">
  <img src="https://img.shields.io/badge/severity-critical-red" alt="Severity: Critical"/>
  <img src="https://img.shields.io/badge/type-infostealer-orange" alt="Type: Infostealer"/>
  <img src="https://img.shields.io/badge/obfuscation-heavy-black" alt="Obfuscation: Heavy"/>
  <img src="https://img.shields.io/badge/purpose-educational_only-blue" alt="Purpose: Educational Only"/>
</p>

> **This document is a detailed forensic dissection of a real-world, multi-layered browser‑based data‑harvesting script.  
> It is intended strictly for security researchers, analysts, and educators working on client‑side threat defence.**

---

## ⚠️ CRITICAL WARNING

**DO NOT EXECUTE THIS CODE** on any system that contains sensitive data or is connected to a production network.  
The script will:

- Read your **clipboard, cookies, localStorage, and sessionStorage**  
- Attempt to capture your **screen and camera**  
- Log your **keystrokes and mouse movements**  
- Enumerate **USB, MIDI, GPU, and gamepad devices**  
- Leak your **public and local IP addresses**  
- Send everything encrypted to a remote attacker‑controlled server  

If you must analyse it, use an **air‑gapped, isolated virtual machine** with no personal accounts, and ensure all network traffic is monitored.

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

This repository contains the **fully reconstructed and annotated source** of a sophisticated JavaScript infostealer. The script is typically embedded in phishing landing pages or compromised legitimate websites. Once executed in a victim’s browser, it silently collects a comprehensive digital fingerprint and exfiltrates it over **WebSocket, HTTP POST, or Telegram Bot API**.

**Key capabilities:**
- Over **25 individual data‑collection modules** that can be enabled/disabled by the attacker.
- **Layered obfuscation** with string encoding and variable renaming.
- **Custom XOR encryption** for all transmitted payloads.
- **Real‑time event listeners** for clipboard, battery, network, and sensor changes.
- **Visual captures** of the user’s screen and camera as JPEG images.
- **Self‑defence mechanisms** that pause collection if a browser automation framework (WebDriver) or ad‑blocker is detected.

---

## Deobfuscation & Configuration

The script uses two primary layers of obfuscation:

### 1. String & Variable Obfuscation
- The attacker’s C2 proxy URL is stored as a base64‑encoded string and decoded at runtime:  
  `atob("aHR0cHM6Ly95b3VyLXByb3h5LmNvbS9hcGk=")` → `https://your-proxy.com/api`
- All meaningful variable names are replaced with short, random identifiers (`_0x3c4d`, `_0x5e6f`, etc.).
- Core functions like the encryption routine (`_0x2e4f`) are expressed with single‑letter parameters.

### 2. XOR Encryption
All data leaving the browser is encrypted using a simple XOR cipher with a hardcoded key (`x-s-k`).  
The function `_0x2e4f(data, key)`:
```javascript
function encrypt(payload, key) {
  const json = JSON.stringify(payload);
  let result = '';
  for (let i = 0; i < json.length; i++) {
    result += String.fromCharCode(json.charCodeAt(i) ^ key.charCodeAt(i % key.length));
  }
  return btoa(result);
}
```

The resulting base64 blob is then sent to the C2 server.

3. Central Configuration Object

The entire behaviour of the stealer is driven by the configuration object _0x3c4d:

Field Purpose Example Value
w WebSocket C2 endpoint wss://s.0/3 (obfuscated placeholder)
a HTTP API endpoint (proxy) https://your-proxy.com/api
t.e Use Telegram exfiltration true/false
t.b Telegram bot token 123456:ABC...
t.c Telegram chat ID -1001234567890
m Collection modules on/off map See table below
i.b Base loop delay (ms) 120000 (2 minutes)
i.j Random jitter (ms) 2000
i.h High‑sensitivity module cooldown (ms) 300000 (5 minutes)
k XOR encryption key "x-s-k"

This modular design lets the attacker fine‑tune which sensors are activated, how often data is sent, and which exfiltration channel is used – all without modifying the core logic.

---

Collection Modules – Full Reference

All modules are controlled via the _0x3c4d.m flags. Each module implements smart caching to avoid redundant calls within the cooldown window (i.h).

Flag Module Data Collected Live Updates Requires HTTPS
p Permissions State of camera, geolocation, microphone, notifications, MIDI, USB No No
i IP & Geolocation Public IP, city, region, country, ISP, coordinates, timezone, ASN No No
d Device Fingerprint User‑agent, platform, languages, screen size, color depth, timezone, hardware concurrency, memory, touch support, WebDriver flag, referrer, window size, orientation No No
f Canvas/WebGL/Audio Canvas image, WebGL renderer & vendor, AudioContext sample rate, available fonts, WebAssembly support No No
g GPU GPU device name, vendor, architecture via navigator.gpu No No
c Navigation Visited URLs (performance entries) No No
geo High‑Accuracy GPS Latitude, longitude, accuracy; reverse‑geocoded address Yes (on new fix) Yes
b Battery Level (%), charging status Yes (on change) No
n Network Effective type (4g, etc.), downlink, RTT Yes (on change) No
s Sensors Accelerometer (x,y,z), gyroscope (alpha, beta, gamma) No (one‑time) No
cl Clipboard Current clipboard text Yes (on paste event) Yes
st Storage Cookies, localStorage, sessionStorage (full dump) No No
w WebRTC Local IP Private IP addresses via ICE candidates Yes (per candidate) No
int Interaction Up to 10 keystrokes and mouse coordinates with timestamps Yes (each event) No
mic Microphone Activity Audio analyser data length (not raw audio) No Yes
sc Screen Capture JPEG screenshot of the entire screen No Yes
cam Camera Capture Camera availability, count, and a snapshot from the user‑facing camera No Yes
gp Gamepad List of connected gamepads (id, button count) No No
mi MIDI Devices Names of connected MIDI inputs No No
u USB Devices Vendor ID and product name of connected USB devices No No
x WebXR Immersive‑VR support flag No No
h History window.history.length No No
e Browser Extensions Detects uBlock Origin and AdBlock by probing extension resources No No

Note: Modules marked with Yes under HTTPS require a secure context (window.location.protocol === "https:"). In insecure HTTP, those modules are silently skipped, and a warning is appended to the log.

---

Data Exfiltration Architecture

The stealer uses a multi‑channel exfiltration strategy, ensuring redundancy and the ability to bypass network restrictions.

1. WebSocket (Primary, Real‑Time)

· A persistent WebSocket connection is opened to _0x3c4d.w.
· All collected data (both periodic snapshots and live event updates) is XOR‑encrypted and sent as soon as available.
· The WebSocket reconnects automatically with an exponential backoff on failure.

2. HTTP POST (Fallback / Bulk)

· If the Telegram path is not enabled (t.e = false), the full encrypted payload is POSTed to the proxy API endpoint _0x3c4d.a.
· The request includes a spoofed User‑Agent header (Mozilla/5.0 (Windows NT 10.0) or similar) to blend in.

3. Telegram Bot API (Alternative)

· When t.e = true, a human‑readable text summary is formatted (max 4096 characters) and sent via sendMessage.
· Any captured screen or camera images are uploaded separately as photos using sendPhoto, with timestamps in the filenames.

All payloads are encrypted before transmission – even over HTTPS – using the XOR cipher with key "x-s-k". The C2 server must decrypt them to extract the plain‑text JSON.

---

Execution Flow & Persistence

The following sequence diagram illustrates the lifecycle of the stealer:

```
┌──────────────┐
│  Page Load   │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ _0xb1c2()    │  Open WebSocket
└──────┬───────┘
       │ onopen
       ▼
┌──────────────┐
│ _0xd3e4()    │  Collect all enabled modules
└──────┬───────┘   (respecting cache cooldowns)
       │
       ▼
┌──────────────┐
│ _0x2e4f()    │  Encrypt aggregated data
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Send via WS  │  + if Telegram: send message & images
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ _0x18a2()    │  SetTimeout loop (base + jitter)
└──────┬───────┘   Re‑calls _0xd3e4()
       │
       ▼
┌──────────────┐
│   Loop ...   │  (unless WebDriver or ad‑blocker detected)
└──────────────┘
```

· The base collection interval is 2 minutes (i.b = 120000 ms) with a random jitter of up to 2 seconds (i.j = 2000 ms).
· Modules that are resource‑intensive or highly invasive (geolocation, microphone, screen, camera) have a separate cooldown of 5 minutes (i.h = 300000 ms) to avoid raising suspicion.
· If the script detects navigator.webdriver === true or the presence of uBlock Origin (using the extension detection module), it permanently stops the collection loop – a technique used to evade sandbox analysis.

---

Auxiliary XLSX Parser (Phishing Context)

The code snippet at the top of the original file contains a standalone function loadFileData() that parses Excel (.xlsx) files using the SheetJS library. It is not part of the infostealer itself but is likely part of the phishing framework that delivers the script.

Functionality:

· Reads base64‑encoded XLSX data.
· Converts the first sheet to JSON, filters out blank rows.
· Heuristically detects the header row.
· Returns the result as a CSV string.

This suggests the malicious script is deployed on a phishing page that also harvests uploaded documents or processed financial/spreadsheet data.

---

Defensive Countermeasures

Organisations and individuals can mitigate this threat through a combination of browser hardening, network controls, and user education.

🔒 Browser Configuration (Recommended)

Setting Recommendation
JavaScript Disable on untrusted sites (use uBlock Origin / NoScript)
Permissions Block clipboard, camera, microphone, sensors, and location by default
WebRTC Disable via browser flags or extensions (e.g., chrome://flags/#disable-webrtc)
Third‑party cookies Block entirely
GPU / USB / MIDI These require user gestures or permissions – never grant to unknown sites
WebDriver Ensure it is not exposed in normal browsing profiles

🛡️ Content Security Policy (CSP)

A strict CSP can prevent the script from connecting to attacker‑controlled domains and from loading inline scripts:

```
Content-Security-Policy: default-src 'self'; connect-src 'self'; script-src 'self'; img-src 'self' data:; style-src 'self' 'unsafe-inline';
```

This would block the WebSocket, the Telegram API, and the proxy fetch. However, if the script is already inline (as in this example), CSP must be combined with a nonce or hash approach – or better, disallow inline scripts entirely.

👤 User Awareness

· Never click suspicious links in emails or messages.
· Treat any website that requests camera, screen sharing, or location access with extreme suspicion.
· Regularly clear browser storage and use a privacy‑focused browser (e.g., Firefox with resistFingerprinting enabled).

---

Detection Opportunities for Blue Teams

Security operations teams can look for the following indicators:

🌐 Network IOCs

· WebSocket connections to short, suspicious domains (e.g., wss://s.0/3).
· Outbound HTTP POST requests to unusual /api endpoints with base64‑encoded bodies.
· Communication with Telegram Bot API (api.telegram.org) from a browser user‑agent (highly anomalous for a bot).

💻 Host‑Based Detection

· JavaScript attempting to access navigator.mediaDevices.getDisplayMedia or getUserMedia in a page that does not normally use those features.
· Rapid enumeration of many different browser APIs (permissions, GPU, USB, MIDI) in a single short script.
· Use of the WebDriver property check to abort execution – a sign of sandbox evasion.

📦 Code‑Level Signatures

· The XOR encryption pattern: String.fromCharCode(... ^ ...) followed by btoa.
· Heavy use of obfuscated single‑letter variable names inside a single <script> block.
· The presence of the string "x-s-k" as a hardcoded key (though easily changed by the attacker).

YARA rule example (for static analysis of decrypted scripts):

```yara
rule JavaScript_Infostealer_XOR_Encryption {
  strings:
    $xor = "fromCharCode" and "^" and "btoa"
    $key = "x-s-k" wide ascii
  condition:
    $xor and $key
}
```

---

Legal & Ethical Notice

This analysis is provided solely for educational and defensive security research.
The code described herein is malicious and its use against any system without explicit, written authorisation is illegal and unethical.
By accessing this document, you agree:

· Not to deploy, distribute, or modify the script for any unauthorised purpose.
· To use the information exclusively to enhance your organisation’s detection and prevention capabilities.
· To handle the original code with the same precautions as any live malware sample.

The author(s) of this analysis assume no liability for misuse. If you are unsure how to safely handle the material, consult your legal team and security policy.

---

Conclusion

This JavaScript infostealer is a prime example of how modern, API‑rich browsers can be weaponised to extract an alarming breadth of personal and device information with just a few hundred lines of obfuscated code. Its modular design, multi‑channel exfiltration, and self‑preservation logic make it a potent threat that demands a layered defence strategy.

Studying the script in depth provides valuable insights into:

· The current state of client‑side attack tooling.
· The importance of strict browser permission models.
· The need for robust network and endpoint detection mechanisms.

Use this knowledge to protect users and systems, never to harm them.

---

For further questions or to contribute detection rules, please open an issue in the repository – but remember, share responsibly.

