

# Podman / systemd Unit Files – SECTION CHEAT SHEET
# (HTL DevOps – enterprise-grade mental model)

================================================================================
[Unit]        → IDENTITY, DEPENDENCIES, START ORDER
================================================================================
Purpose:
- Describes WHAT this unit is
- Defines dependency relationships
- Controls startup/shutdown order

Typical responsibilities:
- Ordering (before / after)
- Hard/soft dependencies
- Human-readable metadata

Common directives:
- Description=        # Short explanation of the unit
- Documentation=      # Reference URL or docs
- After=              # Start AFTER these units
- Before=             # Start BEFORE these units
- Requires=           # Hard dependency (fails if missing)
- Wants=              # Soft dependency
- PartOf=             # Stop/start together with another unit

Important:
- NO container logic here
- Used by ALL systemd units (not Podman-specific)

--------------------------------------------------------------------------------

================================================================================
[Container]   → CONTAINER DEFINITION (Podman / Quadlet ONLY)
================================================================================
Purpose:
- Declaratively defines the container itself
- Replaces `podman run` arguments

Typical responsibilities:
- Image selection
- Environment variables
- Volume mounts
- Ports and networks
- Labels (e.g. Traefik)

Common directives:
- Image=              # Container image
- ContainerName=      # Explicit container name
- Exec=               # Command to run inside container
- Environment=        # ENV variables
- Volume=             # Volume mounts
- Network=            # Podman network
- PublishPort=        # Port mappings
- Label=              # Metadata (Traefik, monitoring, etc.)
- User=               # Container user
- WorkingDir=         # Work directory inside container

Important:
- Exists ONLY in `.container` files
- Ignored by systemd, parsed by Podman Quadlet
- Pure container description, no lifecycle control

--------------------------------------------------------------------------------

================================================================================
[Service]     → RUNTIME CONTROL & SUPERVISION (systemd)
================================================================================
Purpose:
- Controls HOW the container is executed and supervised
- Defines restart behavior and shutdown handling

Typical responsibilities:
- Restart on failure
- Timeout handling
- Kill signals
- Resource limits (optional)

Common directives:
- Type=               # Service type (usually notify or simple)
- Restart=            # no | on-failure | always
- RestartSec=         # Delay before restart
- TimeoutStartSec=    # Startup timeout
- TimeoutStopSec=     # Shutdown timeout
- KillMode=           # process | control-group
- KillSignal=         # Signal for stop
- ExecStartPre=       # Command before start
- ExecStopPost=       # Command after stop

Important:
- Does NOT define container contents
- systemd-centric behavior only

--------------------------------------------------------------------------------

================================================================================
[Install]     → ENABLEMENT & TARGETS
================================================================================
Purpose:
- Defines WHEN this unit is enabled
- Controls persistence across reboots or user logins

Typical responsibilities:
- Boot-time activation
- User-session activation

Common directives:
- WantedBy=            # Target that wants this unit
- RequiredBy=          # Hard enable dependency
- Alias=               # Alternative unit name

Typical targets:
- multi-user.target   # System-wide boot
- default.target      # User session (rootless Podman)

Important:
- Without [Install], unit can NOT be enabled
- Only affects `systemctl enable`

--------------------------------------------------------------------------------

================================================================================
MENTAL MODEL (MEMORIZE THIS)
================================================================================
[Unit]      → WHEN & WITH WHAT
[Container] → WHAT is executed
[Service]   → HOW it is supervised
[Install]   → WHEN it is enabled
================================================================================
