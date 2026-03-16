# Tesla Skill for OpenClaw

Complete Tesla Fleet API control from the terminal - all 66 vehicle commands, energy/Powerwall management, vehicle sharing, charging history, navigation, media controls, V2G/Powershare, and multi-vehicle management.

## What it does

- **All 66 vehicle commands** - doors, charging, climate, security, navigation, media, and more
- **Dashboard** - unified view of all vehicles: battery, range, charging, location, lock status, climate, sentry
- **Climate** - AC, heat, seat heaters/coolers, steering wheel heater, bioweapon defense, dog/camp mode, cabin overheat protection
- **Charging** - start/stop, set limits/amps, charge schedules, port control, charging history and invoices
- **Security** - sentry mode, valet mode, PIN to drive, speed limit, guest mode, remote start
- **Navigation** - send addresses, GPS coordinates, Superchargers, or multi-stop routes to your car
- **Media** - play/pause, next/prev track, favorites, volume control
- **Vehicle sharing** - invite drivers, manage access, revoke invitations
- **Vehicle info** - specs, options, release notes, alerts, service data, warranty
- **Energy monitoring** - Powerwall charge level, solar generation, grid flow, operating mode, time-of-use, storm watch
- **V2G/Powershare** - vehicle-to-grid status for Cybertruck and compatible vehicles
- **Charge station finder** - nearby Superchargers and destination chargers with availability
- **Multi-vehicle** - list, select default, operate on specific cars by name
- **Other** - honk, flash, boombox, HomeLink (garage door), rename vehicle, software updates

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
- "Turn on the AC in Stella and set it to 72"
- "Put my car in dog mode"
- "Turn on sentry mode"
- "Set a speed limit of 65 mph"
- "Navigate to Pike Place Market"
- "Play next track"
- "Turn on the seat heater for the driver"
- "Set the charge amps to 16"
- "Schedule charging between 11 PM and 6 AM on weekdays"
- "Find the nearest Supercharger"
- "How much solar am I generating?"
- "What's my Powerwall at?"
- "Invite user@email.com to drive my Tesla"
- "Show my charging history"
- "What are my car's specs?"
- "Install the software update now"
- "What's my Powershare status?"

## Commands (sample)

```bash
python3 scripts/tesla.py dashboard                      # All vehicles at a glance
python3 scripts/tesla.py status                          # Vehicle status
python3 scripts/tesla.py lock / unlock                   # Door lock/unlock
python3 scripts/tesla.py climate on / off / temp 72      # Climate control
python3 scripts/tesla.py climate keeper dog              # Dog mode
python3 scripts/tesla.py climate seat-heater driver 2    # Seat heater
python3 scripts/tesla.py charge start / stop / limit 80  # Charging
python3 scripts/tesla.py charge amps 16                  # Set charging amps
python3 scripts/tesla.py charge schedule add 23:00 06:00 # Charge schedule
python3 scripts/tesla.py sentry on / off                 # Sentry mode
python3 scripts/tesla.py valet on --pin 1234             # Valet mode
python3 scripts/tesla.py speed-limit set 70              # Speed limit
python3 scripts/tesla.py navigate "Seattle, WA"          # Send nav destination
python3 scripts/tesla.py navigate --supercharger         # Nav to Supercharger
python3 scripts/tesla.py media next / prev / volume 5    # Media controls
python3 scripts/tesla.py sharing invite user@email.com   # Share vehicle access
python3 scripts/tesla.py info specs / warranty / alerts  # Vehicle info
python3 scripts/tesla.py energy status                   # Solar + Powerwall
python3 scripts/tesla.py energy mode self-powered        # Powerwall mode
python3 scripts/tesla.py charging history                # Charging sessions
python3 scripts/tesla.py powershare status               # V2G status
python3 scripts/tesla.py honk / flash / boombox play     # Fun stuff
python3 scripts/tesla.py homelink                        # Garage door
python3 scripts/tesla.py software-update start           # Install update
python3 scripts/tesla.py --car "Stella" status           # Target specific car
```

## How it works

Uses the Tesla Fleet API at `fleet-api.prd.na.vn.cloud.tesla.com`. Authenticates via OAuth2 with `TESLA_EMAIL`. Credentials stored locally only. Refresh token cached in `~/.tesla_cache.json`. No data sent to third parties. Vehicles on firmware 2024.26+ use the Vehicle Command Protocol (VCP) for end-to-end encrypted commands.

Covers all 66 vehicle commands, 6 sharing endpoints, 6 vehicle info endpoints, 12 energy endpoints, and 3 charging history endpoints.

See [SKILL.md](SKILL.md) for full documentation including VCP setup, every command with examples, deprecation warnings, error recovery, and energy monitoring details.

## License

MIT
