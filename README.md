# Security Blade Report — Check Point SmartConsole Extension

Reports on security blade activation status across all managed gateways.

## Files

| File | Purpose |
|------|---------|
| `manifest.json` | Extension metadata and panel registration |
| `index.html` | Main panel UI (all logic is self-contained) |
| `shield.svg` | Panel icon shown in SmartConsole sidebar |

## Features

- **Overview tab** — summary metrics (total gateways, connected/warning/disconnected counts), blade coverage progress bar, and a per-blade activation grid showing how many gateways have each blade enabled.
- **Gateways tab** — sortable table of all gateways with active blade badges and coverage percentage per gateway. Click any row to open a detail panel.
- **Blades tab** — per-blade breakdown grouped by category, showing enabled/disabled gateway counts and coverage.
- **Detail panel** — slide-in panel for any gateway showing active vs inactive blades by category.
- **Filtering** — search by gateway name, IP, type, or blade name; filter by connection status or specific blade.
- **CSV export** — exports all gateways with a column per blade (1/0) and coverage percentage.

## Installation

### SmartConsole R81.20 / R82

1. In SmartConsole, open **Manage & Settings > Blades > Extensions**.
2. Click **Add Extension** and select the folder containing these files.
3. The panel labelled **Blade Activation** will appear in the left sidebar.

### Manual / standalone

The panel also runs as a plain HTML file in any browser for testing — just open `index.html` directly. It uses representative demo data; in a real deployment, replace the `GATEWAY_TEMPLATES` array in `index.html` with data fetched from the Management API (`/web_api/show-simple-gateways` + `/web_api/show-simple-cluster-members`).

## Connecting to the Management API

Replace the static `GATEWAY_TEMPLATES` constant with a real API call:

```javascript
async function fetchGateways(sessionId) {
  const base = window.location.origin; // SmartConsole injects the mgmt server origin
  const res = await fetch(`${base}/web_api/show-simple-gateways`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', 'X-chkp-sid': sessionId },
    body: JSON.stringify({ limit: 500, details_level: 'full' })
  });
  const data = await res.json();
  return data.objects; // map .enabled-features to the blades array
}
```

SmartConsole extensions receive the active session token via `window.smartConsole.getSessionId()`.

## Blade IDs

The 16 blades tracked and their internal IDs:

| ID | Name | Category |
|----|------|----------|
| fw | Firewall | Network |
| vpn | IPsec VPN | VPN |
| nat | NAT | Network |
| ips | IPS | Threat Prevention |
| av | Anti-Virus | Threat Prevention |
| ab | Anti-Bot | Threat Prevention |
| te | Threat Emulation | Threat Prevention |
| tex | Threat Extraction | Threat Prevention |
| urlf | URL Filtering | Access Control |
| appc | App Control | Access Control |
| ctnt | Content Awareness | Access Control |
| ident | Identity Awareness | Access Control |
| mobl | Mobile Access | VPN |
| qos | QoS | Network |
| dlp | DLP | Data |
| comply | Compliance | Management |
