# Anvil Uplink Setup and Usage Guide

## Overview
Anvil Uplink lets local Python scripts connect to your Anvil app. For MyBizz, always use **Server Uplink** so the script can call `@anvil.server.callable` functions and access server-side logic.

**Server vs Client Uplink**
- **Server Uplink:** Full server privileges (required for server functions and Data Tables access).
- **Client Uplink:** Same restrictions as client code (not suitable for MyBizz server-side work).

Reference: https://anvil.works/docs/uplink

## Setup Configurations

### Prerequisites
1. An Anvil account with an active application
2. Python 3.7+ installed locally
3. Anvil Uplink package installed: `pip install anvil-uplink`

### Environment Configuration
Set your Uplink key as an environment variable.

**Windows (PowerShell):**
```powershell
$env:ANVIL_UPLINK_KEY = "your-uplink-key-here"
```

**Linux/macOS:**
```bash
export ANVIL_UPLINK_KEY="your-uplink-key-here"
```

**Important:** Never hardcode your Uplink key in scripts or commits.

### Required Files (Repo)
- `scripts/uplink_connect.py` — Simple connect + wait
- `scripts/test_uplink.py` — Smoke test server call
- `scripts/uplink.py` — Long-running uplink process

## Quick Start (Recommended)

### Step 1: Verify install
```bash
python -m pip show anvil-uplink
```

### Step 2: Start simple connection
```bash
python scripts/uplink_connect.py
```
This will connect and wait until you press Enter to disconnect.

### Step 3: Run the smoke test
```bash
python scripts/test_uplink.py
```
The test script calls the server function `uplink_smoketest_20260217_module` and prints the status.

## Server Smoke Test Function
Add this to a **server module** in the Anvil editor:

```python
import anvil.server

@anvil.server.callable
def uplink_smoketest_20260217_module() -> dict:
    """Minimal Uplink smoke test."""
    return {"status": "success"}
```

## Programmatic Connection Example
```python
import anvil.server
import os

uplink_key = os.environ.get("ANVIL_UPLINK_KEY")
if not uplink_key:
    raise RuntimeError("ANVIL_UPLINK_KEY not set")

anvil.server.connect(uplink_key)
try:
    result = anvil.server.call("uplink_smoketest_20260217_module")
    print(result)
finally:
    anvil.server.disconnect()
```

## Troubleshooting

### 1) Missing module errors
If you see errors like:
```
No module named 'stripe'
```
Remove or comment out imports for modules not enabled in Anvil, or enable the service in the Anvil app.

### 2) Function not registered
If you see:
```
No server function matching "..." has been registered
```
Confirm the function exists in a **server module**, then **save + restart** the server runtime.

### 3) Hidden BOM / control character
If you see:
```
invalid non-printable character U+FEFF
```
Remove the hidden BOM in the Anvil editor by retyping the first line, then save.

## Best Practices
1. Always disconnect after operations complete
2. Keep uplink keys in environment variables
3. Use smoke tests before running full integration tasks
4. Restart server runtime after changes in the Anvil editor
5. Log operations when running integration scripts

## Security Considerations
1. Never commit Uplink keys to version control
2. Rotate keys regularly in the Anvil dashboard
3. Use Server Uplink only for trusted scripts


