---
name: tesla
version: "2.0.0"
description: Full Tesla control from the terminal - dashboard, scheduled preconditioning, energy monitoring, charge station finder, multi-vehicle management via Fleet API.
author: mvanhorn
license: MIT
repository: https://github.com/mvanhorn/clawdbot-skill-tesla
homepage: https://developer.tesla.com/docs/fleet-api
metadata:
  openclaw:
    emoji: "🚗"
    requires:
      env:
        - TESLA_EMAIL
    primaryEnv: TESLA_EMAIL
    tags:
      - tesla
      - vehicle
      - iot
      - fleet-api
      - electric-vehicle
      - ev
      - charging
      - supercharger
      - powerwall
      - solar
      - energy
      - automotive
    triggers:
      - tesla
      - my car
      - my vehicle
      - battery level
      - charge status
      - lock my car
      - unlock my car
      - car location
      - where is my car
      - climate control
      - precondition
      - supercharger
      - powerwall
      - solar panels
      - energy usage
      - fleet api
      - honk horn
      - flash lights
      - charge my car
---

# Tesla

Full Tesla control from OpenClaw. Dashboard view, scheduled preconditioning, energy monitoring, charge station finder, and multi-vehicle management - all via the Tesla Fleet API.

> **Fleet API (2026):** Tesla has deprecated the legacy Owner API. All vehicles now use the Fleet API at
> `fleet-api.prd.na.vn.cloud.tesla.com`. Vehicles on firmware 2024.26+ require the Vehicle Command Protocol
> (VCP). The `tesla-fleet-api` Python package handles both paths automatically.

---

## Setup

### Prerequisites

- Python 3.10+
- A Tesla account with at least one vehicle, Powerwall, or solar system
- `TESLA_EMAIL` environment variable set to your Tesla account email

### First-time authentication

```bash
TESLA_EMAIL="you@email.com" python3 {baseDir}/scripts/tesla.py auth
```

Steps:
1. The script displays a Tesla login URL
2. Open it in your browser, sign in, and authorize the application
3. Copy the callback URL from your browser's address bar (starts with `https://auth.tesla.com/void/callback?...`)
4. Paste it back into the terminal
5. Token is cached at `~/.tesla_cache.json` for ~30 days with automatic refresh

### Environment variables

| Variable | Required | Description |
|---|---|---|
| `TESLA_EMAIL` | Yes | Your Tesla account email address |

Token storage: `~/.tesla_cache.json` (auto-created on first auth, auto-refreshes)

---

## Vehicle Command Protocol (VCP)

Vehicles running firmware 2024.26 or later require VCP for sending commands. Without VCP configured, status queries will work but commands (lock, unlock, climate, charge start/stop) will fail with a `vehicle_command_protocol_required` error.

### How to check if VCP is needed

```bash
python3 {baseDir}/scripts/tesla.py status
```

If you see a VCP error in the output, your vehicle needs VCP setup.

### VCP setup

1. Generate a public/private key pair on your machine:
   ```bash
   openssl ecparam -name prime256v1 -genkey -noout -out private_key.pem
   openssl ec -in private_key.pem -pubout -out public_key.pem
   ```

2. Register your public key with Tesla's servers. This requires creating a third-party application at https://developer.tesla.com/ and uploading your `public_key.pem` to a `.well-known` endpoint on your registered domain.

3. Pair your key with each vehicle. Sit inside the vehicle with the key card on the center console, then run:
   ```bash
   python3 {baseDir}/scripts/tesla.py vcp-pair --key private_key.pem
   ```
   Tap "Approve" on the vehicle's touchscreen when prompted.

4. Once paired, commands are end-to-end encrypted between your machine and the vehicle. The `tesla-fleet-api` package handles signing automatically when it finds your key.

> **Note:** If you are only reading vehicle data (status, location, charge state), VCP is not required. VCP is only needed for sending commands.

---

## Dashboard

Get a unified view of all your vehicles in one command:

```bash
python3 {baseDir}/scripts/tesla.py dashboard
```

Output includes for each vehicle:
- Name, model, and state (online/asleep/offline)
- Battery level and estimated range
- Charging state and time remaining (if charging)
- Location as a human-readable address (reverse geocoded from GPS)
- Lock status (locked/unlocked)
- Climate state (on/off, cabin temperature)
- Software version and any pending updates

Example output:
```
--- Dashboard ---

1. Snowflake (Model Y - Online)
   Battery: 78% (245 mi)    Charging: Not charging
   Location: 4521 Mercer Way, Mercer Island, WA 98040
   Locked: Yes    Climate: Off (68F inside)
   Software: 2025.48.3 (up to date)

2. Stella (Model 3 - Asleep)
   Battery: 92% (310 mi)    Charging: Complete
   Location: 1200 NE 45th St, Seattle, WA 98105
   Locked: Yes    Climate: Off
   Software: 2025.48.3 (update available: 2025.50.1)
```

---

## Multi-Vehicle Management

### List all vehicles

```bash
python3 {baseDir}/scripts/tesla.py list
```

Shows all vehicles on your account with name, VIN, model, state, and battery level.

### Select a specific vehicle

Use `--car` or `-c` with any command to target a specific vehicle by its display name:

```bash
python3 {baseDir}/scripts/tesla.py --car "Snowflake" status
python3 {baseDir}/scripts/tesla.py -c "Stella" lock
python3 {baseDir}/scripts/tesla.py --car "Snowflake" climate on
```

Without `--car`, commands target your first vehicle.

### Set a default vehicle

If you have multiple vehicles and mostly control one, set a default:

```bash
python3 {baseDir}/scripts/tesla.py set-default "Stella"
```

The default persists in `~/.tesla_cache.json`. Override it anytime with `--car`.

---

## Commands Reference

### Vehicle status

```bash
# Full status for default vehicle
python3 {baseDir}/scripts/tesla.py status

# Status for a specific vehicle
python3 {baseDir}/scripts/tesla.py --car "Snowflake" status

# Raw JSON output (useful for scripting)
python3 {baseDir}/scripts/tesla.py status --json
```

### Lock and unlock

```bash
python3 {baseDir}/scripts/tesla.py lock
python3 {baseDir}/scripts/tesla.py unlock
python3 {baseDir}/scripts/tesla.py --car "Stella" lock
```

### Climate control

```bash
# Turn climate on/off
python3 {baseDir}/scripts/tesla.py climate on
python3 {baseDir}/scripts/tesla.py climate off

# Set temperature (Fahrenheit by default)
python3 {baseDir}/scripts/tesla.py climate temp 72

# Set temperature in Celsius
python3 {baseDir}/scripts/tesla.py climate temp 22 --celsius

# Seat heater (0=off, 1=low, 2=medium, 3=high)
python3 {baseDir}/scripts/tesla.py climate seat-heater driver 2
python3 {baseDir}/scripts/tesla.py climate seat-heater passenger 1

# Steering wheel heater
python3 {baseDir}/scripts/tesla.py climate steering-heater on
python3 {baseDir}/scripts/tesla.py climate steering-heater off
```

### Charging

```bash
# Check charge status
python3 {baseDir}/scripts/tesla.py charge status

# Start/stop charging
python3 {baseDir}/scripts/tesla.py charge start
python3 {baseDir}/scripts/tesla.py charge stop

# Set charge limit (percentage)
python3 {baseDir}/scripts/tesla.py charge limit 80
python3 {baseDir}/scripts/tesla.py charge limit 100

# Open/close charge port
python3 {baseDir}/scripts/tesla.py charge port-open
python3 {baseDir}/scripts/tesla.py charge port-close
```

### Location

```bash
# Get location as address + GPS coordinates + Google Maps link
python3 {baseDir}/scripts/tesla.py location

# GPS coordinates only (no reverse geocoding)
python3 {baseDir}/scripts/tesla.py location --gps-only

# Example output:
# Snowflake Location:
#   Address: 4521 Mercer Way, Mercer Island, WA 98040
#   GPS: 47.5707, -122.2220
#   Map: https://www.google.com/maps?q=47.5707,-122.2220
#   Speed: Parked
#   Heading: 185 (S)
```

The location command reverse geocodes GPS coordinates to a human-readable street address using the Nominatim API (OpenStreetMap, no API key needed). Falls back to raw GPS if geocoding fails.

### Honk and flash

```bash
python3 {baseDir}/scripts/tesla.py honk
python3 {baseDir}/scripts/tesla.py flash
python3 {baseDir}/scripts/tesla.py --car "Snowflake" honk
```

### Wake

```bash
python3 {baseDir}/scripts/tesla.py wake
```

Wakes a sleeping vehicle. Most commands auto-wake, but you can use this to pre-wake before sending multiple commands.

### Trunk and frunk

```bash
# Open rear trunk
python3 {baseDir}/scripts/tesla.py trunk open

# Open frunk (front trunk)
python3 {baseDir}/scripts/tesla.py frunk open
```

### Windows

```bash
# Vent all windows slightly
python3 {baseDir}/scripts/tesla.py windows vent

# Close all windows
python3 {baseDir}/scripts/tesla.py windows close
```

---

## Scheduled Commands

Schedule preconditioning or charge windows so your car is ready when you need it.

### Scheduled preconditioning (departure time)

```bash
# Precondition by 7:30 AM tomorrow
python3 {baseDir}/scripts/tesla.py schedule precondition 07:30

# Precondition by a specific time (24h format)
python3 {baseDir}/scripts/tesla.py schedule precondition 17:00

# Cancel scheduled preconditioning
python3 {baseDir}/scripts/tesla.py schedule precondition off

# Precondition for a specific vehicle
python3 {baseDir}/scripts/tesla.py --car "Stella" schedule precondition 06:45
```

When scheduled, the vehicle will start climate control and (if plugged in) battery preconditioning before the departure time so the cabin is comfortable and the battery is warm for maximum range.

### Charge window (off-peak charging)

```bash
# Charge only between 11 PM and 6 AM (off-peak rates)
python3 {baseDir}/scripts/tesla.py schedule charge-window 23:00 06:00

# Remove charge window (charge anytime)
python3 {baseDir}/scripts/tesla.py schedule charge-window off
```

### View all schedules

```bash
python3 {baseDir}/scripts/tesla.py schedule list
```

---

## Energy Monitoring (Powerwall / Solar)

If your Tesla account includes Powerwall or solar products, you can monitor them from the same skill.

### List energy sites

```bash
python3 {baseDir}/scripts/tesla.py energy list
```

Shows all energy products on your account (Powerwall, Solar Roof, standalone solar).

### Energy status

```bash
# Current energy flow
python3 {baseDir}/scripts/tesla.py energy status

# Example output:
# Home Energy Status:
#   Solar: 4.2 kW generating
#   Powerwall: 87% (10.4 kWh stored)
#   Grid: Exporting 1.8 kW
#   Home: Using 2.4 kW
#   Mode: Self-Powered
```

### Powerwall controls

```bash
# Set backup reserve percentage
python3 {baseDir}/scripts/tesla.py energy reserve 20

# Set operating mode
python3 {baseDir}/scripts/tesla.py energy mode self-powered
python3 {baseDir}/scripts/tesla.py energy mode time-based
python3 {baseDir}/scripts/tesla.py energy mode backup-only

# Toggle storm watch
python3 {baseDir}/scripts/tesla.py energy storm-watch on
python3 {baseDir}/scripts/tesla.py energy storm-watch off
```

### Energy history

```bash
# Daily energy production/consumption for the last 7 days
python3 {baseDir}/scripts/tesla.py energy history --days 7

# Monthly summary
python3 {baseDir}/scripts/tesla.py energy history --months 1
```

---

## Charge Station Finder

Find nearby Tesla Superchargers and destination chargers.

```bash
# Find Superchargers near your vehicle's current location
python3 {baseDir}/scripts/tesla.py chargers nearby

# Find chargers near a specific address
python3 {baseDir}/scripts/tesla.py chargers nearby --address "Seattle, WA"

# Find chargers near GPS coordinates
python3 {baseDir}/scripts/tesla.py chargers nearby --lat 47.6062 --lon -122.3321

# Filter by charger type
python3 {baseDir}/scripts/tesla.py chargers nearby --type supercharger
python3 {baseDir}/scripts/tesla.py chargers nearby --type destination

# Show availability (stalls in use vs total)
python3 {baseDir}/scripts/tesla.py chargers nearby --availability
```

Example output:
```
Superchargers near Snowflake (4521 Mercer Way, Mercer Island, WA):

1. Tesla Supercharger - Bellevue, WA (3.2 mi)
   1234 Bellevue Way NE - 12 stalls (8 available)
   250 kW - Open 24/7
   Navigate: https://www.google.com/maps/dir/?api=1&destination=47.6101,-122.2015

2. Tesla Supercharger - Renton, WA (5.8 mi)
   800 Rainier Ave S - 8 stalls (5 available)
   150 kW - Open 24/7
   Navigate: https://www.google.com/maps/dir/?api=1&destination=47.4799,-122.2034
```

### Navigate to charger

```bash
# Send navigation to the nearest Supercharger directly to your vehicle
python3 {baseDir}/scripts/tesla.py chargers navigate 1
```

---

## Error Recovery

### Authentication expired

If your token has expired (after ~30 days without use), you will see:
```
Error: Token expired or revoked
```

Fix: Re-authenticate:
```bash
TESLA_EMAIL="you@email.com" python3 {baseDir}/scripts/tesla.py auth
```

### Vehicle asleep

Vehicles go to sleep after ~15 minutes of inactivity to save battery. The script automatically wakes the vehicle before sending commands, but if wake fails:
```
Error: Vehicle did not wake up within 30 seconds
```

Fix: Try again - the vehicle may need more time. The script retries up to 3 times with 10-second intervals:
```bash
python3 {baseDir}/scripts/tesla.py wake
# Then retry your command
python3 {baseDir}/scripts/tesla.py status
```

If the vehicle is in a deep sleep state (e.g., parked for days), it may take up to 2 minutes to wake. The script handles this automatically with exponential backoff.

### VCP not configured

If you see:
```
Error: vehicle_command_protocol_required
```

Your vehicle's firmware requires VCP for commands. See the [VCP setup section](#vehicle-command-protocol-vcp) above. Note that read-only operations (status, location, charge status) still work without VCP.

### Command timeout

If a command times out:
```
Error: Command timed out after 30 seconds
```

The vehicle may be in an area with poor cellular connectivity. The command may still execute - check status before retrying. Common in underground parking or remote areas.

### Rate limiting

The Tesla Fleet API enforces rate limits. If you see:
```
Error: 429 Too Many Requests
```

Wait 60 seconds before retrying. Avoid polling status more frequently than once per minute.

### Vehicle offline

If the vehicle shows as "offline", it has no cellular connection. Commands cannot be sent until it reconnects. This can happen in:
- Underground parking garages
- Remote areas without cell coverage
- When the 12V battery is very low

No fix except waiting for the vehicle to regain connectivity.

---

## Example Chat Usage

Ask naturally and the skill handles the rest:

- "Show me my Tesla dashboard"
- "Is my Tesla locked?"
- "Lock Stella"
- "What's Snowflake's battery level?"
- "Where is my Model Y?"
- "Turn on the AC in Stella and set it to 72"
- "Honk the horn on Snowflake"
- "Start charging my car"
- "Set the charge limit to 80%"
- "Open the frunk"
- "Precondition my car for 7:30 AM"
- "Find the nearest Supercharger"
- "How much solar am I generating right now?"
- "What's my Powerwall battery at?"
- "Set Powerwall to self-powered mode"
- "Schedule charging between 11 PM and 6 AM"
- "Vent the windows on Stella"
- "What software version is my car running?"
- "Flash the lights on Snowflake"
- "Set the seat heater to high"

---

## API Reference

This skill uses the Tesla Fleet API:
- **Endpoint:** `fleet-api.prd.na.vn.cloud.tesla.com`
- **Auth:** OAuth2 via `TESLA_EMAIL` with token cached locally
- **Python package:** `tesla-fleet-api` (handles Fleet API + VCP automatically)
- **Docs:** https://developer.tesla.com/docs/fleet-api

The Fleet API replaced the legacy Owner API (tesla-api.timdorr.com) which is no longer functional.

---

## Privacy and Security

- All credentials are stored locally on your machine only
- OAuth2 refresh token cached in `~/.tesla_cache.json`
- No data is sent to any third party
- VCP commands are end-to-end encrypted between your machine and the vehicle
- Reverse geocoding uses OpenStreetMap Nominatim (sends only GPS coordinates, no account data)
- Tokens auto-refresh for ~30 days; re-auth required if unused for longer
