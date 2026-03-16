# 🚗 Tesla Skill for OpenClaw

Full Tesla control from the terminal - dashboard, scheduled preconditioning, energy monitoring, charge station finder, and multi-vehicle management via Fleet API.

## What it does

- **Dashboard** - unified view of all vehicles: battery, range, charging, location, lock status, climate
- **Vehicle control** - lock/unlock, climate, charging, trunk/frunk, windows
- **Scheduled commands** - precondition at departure time, off-peak charge windows
- **Multi-vehicle** - list, select default, operate on specific cars by name
- **Location** - human-readable address via reverse geocoding + GPS + map link
- **Energy monitoring** - Powerwall charge level, solar generation, grid flow, operating mode
- **Charge station finder** - nearby Superchargers and destination chargers with availability
- **Fun stuff** - honk the horn, flash the lights

## Quick start

### Install the skill

```bash
git clone https://github.com/mvanhorn/clawdbot-skill-tesla.git ~/.openclaw/skills/tesla
```

### Authenticate (one-time)

```bash
export TESLA_EMAIL="you@email.com"
python3 scripts/tesla.py auth
```

This opens a Tesla login URL. Sign in, authorize, paste the callback URL back. Token caches for ~30 days and auto-refreshes.

### Example chat usage

- "Show me my Tesla dashboard"
- "Is my Tesla locked?"
- "What's Snowflake's battery level?"
- "Where's my Model Y?"
- "Turn on the AC in Stella and set it to 72"
- "Precondition my car for 7:30 AM"
- "Find the nearest Supercharger"
- "How much solar am I generating?"
- "What's my Powerwall at?"
- "Set the charge limit to 80%"

## Commands

```bash
python3 scripts/tesla.py dashboard                      # All vehicles at a glance
python3 scripts/tesla.py list                            # List all vehicles
python3 scripts/tesla.py status                          # Vehicle status
python3 scripts/tesla.py lock                            # Lock
python3 scripts/tesla.py unlock                          # Unlock
python3 scripts/tesla.py climate on                      # AC on
python3 scripts/tesla.py climate temp 72                 # Set temp
python3 scripts/tesla.py charge status                   # Charge info
python3 scripts/tesla.py charge limit 80                 # Set charge limit
python3 scripts/tesla.py location                        # Address + GPS + map link
python3 scripts/tesla.py honk                            # Honk horn
python3 scripts/tesla.py flash                           # Flash lights
python3 scripts/tesla.py trunk open                      # Open trunk
python3 scripts/tesla.py frunk open                      # Open frunk
python3 scripts/tesla.py schedule precondition 07:30     # Precondition by 7:30 AM
python3 scripts/tesla.py schedule charge-window 23:00 06:00  # Off-peak charging
python3 scripts/tesla.py energy status                   # Solar + Powerwall status
python3 scripts/tesla.py chargers nearby                 # Nearby Superchargers
python3 scripts/tesla.py --car "Stella" status           # Target specific car
```

## How it works

Uses the Tesla Fleet API at `fleet-api.prd.na.vn.cloud.tesla.com`. Authenticates via OAuth2 with `TESLA_EMAIL`. Credentials stored locally only. Refresh token cached in `~/.tesla_cache.json`. No data sent to third parties. Vehicles on firmware 2024.26+ use the Vehicle Command Protocol (VCP) for end-to-end encrypted commands.

See [SKILL.md](SKILL.md) for full documentation including VCP setup, error recovery, and energy monitoring details.

## License

MIT
