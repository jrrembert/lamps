# Portable App Health Light Array — Outstanding Design Questions

Status: Open  
Owner: Ryan Rembert  
Document version: 0.1  
Last updated: 2026-05-22

## Purpose

This document tracks every unresolved design decision extracted from the technical requirements document. Each item maps to a `[TODO #N]` placeholder in the TRD/spec.

## Question Register

### TODO #1 — Define “business” / monitored unit

**Question:** What exact entity does one LED represent?

**Decision needed:** Pick one primary entity type for v1.

**Known candidate meanings:**

- Customer account
- Customer deployment
- Product
- Application
- Service group
- Environment
- Company / business line
- Internal SLA unit

**Why it matters:** This affects LED labels, Health Gateway grouping, status rollups, config schema, and user mental model.

**Blocks:**

- TRD section 1
- TRD section 4
- TRD section 10
- Health Gateway schema
- Acceptance test fixture data

**Owner:** Ryan  
**Status:** Open

---

### TODO #2 — LED count and physical layout

**Question:** How many monitored units must the v1 device display at once, and what physical layout must be used?

**Decision needed:** Set exact LED count, rows, columns, LED order, and max payload item count.

**Known candidate layouts:**

- 8 LEDs, 2 x 4
- 12 LEDs, 3 x 4
- 16 LEDs, 4 x 4
- 24 LEDs, 4 x 6

**Why it matters:** This drives enclosure size, power budget, BLE payload size, LED mapping, config validation, and manufacturing complexity.

**Blocks:**

- TRD section 6.2
- TRD section 9.4
- TRD section 10.1
- Hardware BOM
- Enclosure design

**Owner:** Ryan  
**Status:** Open

---

### TODO #3 — Health data sources

**Question:** Which systems provide health data for v1?

**Decision needed:** Identify every production data source the Health Gateway must read.

**Data source details needed:**

- System name
- API endpoint
- Authentication method
- Network access requirements
- Query cadence
- Rate limits
- Required fields
- Failure behavior

**Known source categories:**

- Custom `/health` endpoints
- Datadog
- Prometheus
- Grafana
- Sentry
- CloudWatch
- Statuspage
- Internal database
- Incident management system
- Deploy system

**Why it matters:** This defines Health Gateway implementation scope and whether the Laptop Agent needs private-network access.

**Blocks:**

- TRD section 12.2
- TRD section 17.3
- Gateway implementation
- Agent implementation beyond simple gateway polling

**Owner:** Ryan  
**Status:** Open

---

### TODO #4 — Health Gateway URL and deployment location

**Question:** Where will the Health Gateway run, and what hostname will the device use?

**Decision needed:** Define production base URL, staging base URL, local development URL, and TLS certificate strategy.

**Details needed:**

- Production hostname
- Staging hostname
- Local dev strategy
- TLS certificate authority
- Public internet exposure
- VPN requirement
- Request path prefix

**Why it matters:** The firmware needs a fixed gateway contract, token audience, TLS behavior, and provisioning flow.

**Blocks:**

- TRD section 8.2
- TRD section 11.2
- TRD section 12.1
- Wi-Fi integration tests
- OTA staging tests if hosted together

**Owner:** Ryan  
**Status:** Open

---

### TODO #5 — Direct Wi-Fi access constraints

**Question:** What health information can the device access directly over Wi-Fi without the Laptop Agent?

**Decision needed:** Decide whether direct Wi-Fi polling is allowed to reach production health data from public networks.

**Details needed:**

- Public gateway access policy
- VPN-only systems
- IP allowlist requirements
- Device token permissions
- Coffee shop / coworking Wi-Fi behavior
- Captive portal behavior

**Why it matters:** This determines whether Wi-Fi mode is a full standalone mode or a limited mode.

**Blocks:**

- Source precedence behavior validation
- Security review
- Gateway auth strategy

**Owner:** Ryan  
**Status:** Open

---

### TODO #6 — Laptop Agent platform and multi-host behavior

**Question:** Which desktop platforms must the Laptop Agent support, and can multiple laptops pair with one device?

**Decision needed:** Choose supported v1 platforms and pairing capacity.

**Platform choices:**

- macOS
- Linux
- Windows

**Multi-host choices:**

- One bonded host only
- Multiple bonded hosts with latest connection active
- Multiple bonded hosts with explicit host selection

**Why it matters:** BLE APIs, packaging, reconnect behavior, support burden, and security model differ by platform.

**Blocks:**

- TRD section 9.3
- TRD section 11.1
- TRD section 17.2
- Agent packaging
- BLE test matrix

**Owner:** Ryan  
**Status:** Open

---

### TODO #7 — Battery target and battery hardware

**Question:** What battery life, battery capacity, and battery reporting accuracy are required?

**Decision needed:** Define target runtime and battery measurement hardware.

**Inputs needed:**

- Target runtime in hours
- Max device thickness
- Max device weight
- USB-C charge current
- Battery percentage precision
- Battery safety requirements
- Use while charging requirement

**Why it matters:** LED count, brightness cap, regulator choice, charger IC, enclosure size, and firmware power modes depend on this.

**Blocks:**

- TRD section 3.1
- TRD section 6.3
- Hardware BOM
- Enclosure design
- Battery acceptance tests

**Owner:** Ryan  
**Status:** Open

---

### TODO #8 — Product name and device identity format

**Question:** What product name and device ID format will be used?

**Decision needed:** Pick naming convention for device name, BLE advertisement name, device ID, and serial number.

**Details needed:**

- Product name
- BLE advertisement prefix
- Device ID format
- Serial number format
- Human-readable name format

**Why it matters:** This affects firmware config, provisioning, Gateway paths, logs, and support workflows.

**Blocks:**

- Config defaults
- Gateway URL path
- BLE discovery UX
- Manufacturing labels

**Owner:** Ryan  
**Status:** Open

---

### TODO #9 — Privacy mode

**Question:** Does the device need a privacy mode to avoid exposing customer or business health in public spaces?

**Decision needed:** Define privacy mode behavior.

**Known candidate behaviors:**

- Disable all LEDs
- Show aggregate status only
- Replace red/yellow/green with neutral colors
- Hide labels only
- Require button press to reveal status for a short time

**Why it matters:** The device is intended for travel and coworking use. Visible red status could expose sensitive operational information.

**Blocks:**

- TRD section 13
- LED rendering
- Button behavior
- Config schema

**Owner:** Ryan  
**Status:** Open

---

### TODO #10 — Labeling strategy

**Question:** How will users know which LED maps to which monitored unit?

**Decision needed:** Choose the v1 labeling strategy.

**Known candidate strategies:**

- Printed overlay
- Removable paper insert
- Physical engraved labels
- No labels; fixed memorized order
- Companion config view in Laptop Agent

**Why it matters:** Without labels, the device is only useful when the mapping is memorized.

**Blocks:**

- Enclosure design
- User setup workflow
- Config UX
- Acceptance testing

**Owner:** Ryan  
**Status:** Open

---

### TODO #11 — Poll cadence, TTL, and BLE payload size

**Question:** What update cadence and data freshness policy are required?

**Decision needed:** Define polling interval, TTL, maximum staleness, payload size limit, and BLE chunking threshold.

**Defaults currently present in TRD:**

- Poll interval: 15 seconds
- TTL: 30 seconds
- HTTP timeout: 5 seconds

**Details needed:**

- Maximum acceptable delay from incident to red LED
- Minimum acceptable battery life
- Gateway rate limit
- BLE payload max size
- Number of monitored units

**Why it matters:** Cadence affects battery life, network usage, perceived responsiveness, BLE chunking, and gateway load.

**Blocks:**

- TRD section 8.1
- TRD section 9.4
- Freshness tests
- Battery tests

**Owner:** Ryan  
**Status:** Open

---

### TODO #12 — Health status computation rules

**Question:** What exact business health rules produce green, yellow, red, maintenance, muted, and unknown?

**Decision needed:** Define status computation for each monitored unit type.

**Metric inputs to define:**

- Latency threshold
- Error rate threshold
- Availability threshold
- Recent deploy state
- Incident state
- Dependency health
- Synthetic check result
- Queue depth
- Background job health
- Custom business metrics

**Why it matters:** The device is only as useful as the status rollup. The firmware intentionally does not compute these statuses.

**Blocks:**

- TRD section 12.3
- Health Gateway implementation
- Test fixture generation
- Acceptance tests

**Owner:** Ryan  
**Status:** Open

---

### TODO #13 — Manufacturing intent and production volume

**Question:** Is v1 a one-off personal build, a small batch, or a manufacturable product?

**Decision needed:** Define intended production quantity and quality bar.

**Known candidate quantities:**

- 1 personal prototype
- 5–10 internal prototypes
- 25–100 small batch
- 100+ manufacturable product

**Why it matters:** PCB design, enclosure method, component sourcing, test fixture design, certification planning, and documentation depth depend on this.

**Blocks:**

- TRD section 3.2
- Hardware development process
- Manufacturing test plan
- Compliance plan

**Owner:** Ryan  
**Status:** Open

---

### TODO #14 — Enclosure size, diffuser, and industrial design

**Question:** What physical size, material, and visual design should the device have?

**Decision needed:** Define enclosure dimensions, diffuser material, LED spacing, and finish.

**Details needed:**

- Max width
- Max depth
- Max height
- Target weight
- Diffuser material
- LED visibility in daylight
- Desk angle
- Pocket/bag durability
- Label accommodation

**Why it matters:** LED layout, battery size, power switch placement, button placement, and manufacturing method depend on enclosure constraints.

**Blocks:**

- TRD section 6.2
- Hardware BOM
- Mechanical CAD
- User acceptance tests

**Owner:** Ryan  
**Status:** Open

---

### TODO #15 — OTA hosting and release channel

**Question:** Where will firmware update manifests and signed firmware images be hosted?

**Decision needed:** Pick OTA hosting location, release channel naming, and rollout process.

**Details needed:**

- Manifest URL
- Firmware binary URL format
- Signing key ownership
- Staging channel
- Production channel
- Rollback policy
- Update frequency

**Why it matters:** OTA touches firmware security, release automation, and recovery behavior.

**Blocks:**

- TRD section 15
- Release pipeline
- Security review
- Production deployment

**Owner:** Ryan  
**Status:** Open

---

### TODO #16 — Operational ownership

**Question:** Who owns firmware releases, Health Gateway operation, Laptop Agent releases, and incident response for the device stack?

**Decision needed:** Assign owners and escalation paths.

**Areas requiring ownership:**

- Firmware repository
- Hardware repository
- Gateway service
- Agent package
- OTA signing key
- Production device registry
- Monitoring
- Support

**Why it matters:** Release and deployment tasks need accountable owners before production use.

**Blocks:**

- TRD section 17.1
- Deployment plan
- Runbook
- Security review

**Owner:** Ryan  
**Status:** Open

---

### TODO #17 — Provisioning flow

**Question:** How will the device receive Wi-Fi credentials, device token, Gateway URL, and monitored unit config?

**Decision needed:** Define exact provisioning UX.

**Known candidate flows:**

- BLE provisioning from Laptop Agent
- USB serial provisioning CLI
- Captive portal provisioning
- Pre-flashed config
- QR code assisted setup

**Why it matters:** Provisioning affects user experience, firmware complexity, secret handling, and factory setup.

**Blocks:**

- TRD section 10.3
- Config write path
- Security testing
- User setup guide

**Owner:** Ryan  
**Status:** Open

---

### TODO #18 — Gateway authentication and token lifecycle

**Question:** What authentication scheme will the device and Laptop Agent use with the Health Gateway?

**Decision needed:** Define token type, token scope, token issuance, rotation, revocation, and storage.

**Known candidate approaches:**

- Static device bearer token
- Short-lived token from provisioning service
- mTLS client certificate
- Signed request with device key
- OAuth device flow for the Laptop Agent

**Why it matters:** This is central to security, lost-device handling, and production operation.

**Blocks:**

- TRD section 12.4
- TRD section 13
- Gateway implementation
- Provisioning
- Security review

**Owner:** Ryan  
**Status:** Open

---

### TODO #19 — Initial monitored unit inventory

**Question:** Which exact monitored units must be present in the first working build?

**Decision needed:** Provide the initial list of monitored unit IDs, labels, and LED index order.

**Required fields:**

- `id`
- `label`
- `led_index`
- `status_source`
- `enabled`

**Why it matters:** Firmware config, Gateway fixtures, acceptance tests, and demos need concrete entities.

**Blocks:**

- Device config
- Gateway fixtures
- Demo script
- Acceptance test data

**Owner:** Ryan  
**Status:** Open

---

### TODO #20 — Exact color palette and gamma curve

**Question:** What exact RGB values and brightness behavior should be used for each status?

**Decision needed:** Define RGB values, gamma correction curve, night brightness, and animation curves.

**Known color concerns:**

- Colorblind accessibility
- Daylight visibility
- Eye strain
- Ambient nighttime use
- Meaning of blue and purple

**Why it matters:** The TRD defines status behavior, but exact color values and animation curves must be locked for testable rendering.

**Blocks:**

- TRD section 7.7
- LED renderer tests
- UX acceptance tests

**Owner:** Ryan  
**Status:** Open

---

### TODO #21 — Incident attention behavior

**Question:** Should red status create extra attention behavior beyond LED color?

**Decision needed:** Define whether red status should blink, pulse, escalate, stay steady, or respect quiet hours.

**Known candidate behaviors:**

- Steady red
- Slow blink
- Fast blink after threshold
- No blinking in privacy mode
- Quiet hours dimming

**Why it matters:** The device must be useful without becoming distracting.

**Blocks:**

- LED rendering
- Config schema
- Acceptance tests

**Owner:** Ryan  
**Status:** Open

---

### TODO #22 — Wi-Fi network behavior

**Question:** How many Wi-Fi networks should the device remember, and how should it behave on captive portals?

**Decision needed:** Define network credential capacity and failure behavior.

**Known candidate behaviors:**

- Store one network
- Store multiple networks
- Support laptop hotspot as primary travel mode
- Ignore captive portals
- Expose captive portal setup mode

**Why it matters:** Travel use depends on predictable network behavior.

**Blocks:**

- TRD section 8.1
- Config schema
- Provisioning
- Wi-Fi tests

**Owner:** Ryan  
**Status:** Open

---

### TODO #23 — Diagnostics retention and privacy

**Question:** What diagnostics should persist, and for how long?

**Decision needed:** Define persistent diagnostic fields, retention length, and export behavior.

**Diagnostic data candidates:**

- Boot count
- Last reset reason
- Wi-Fi failure count
- BLE disconnect count
- Last rejected payload reason
- Last OTA result
- Battery cycle data
- Health payload metadata

**Why it matters:** Diagnostics help support but can leak sensitive operational metadata.

**Blocks:**

- TRD section 14
- Security review
- Support workflow

**Owner:** Ryan  
**Status:** Open

---

### TODO #24 — Compliance and regulatory target

**Question:** What regulatory and compliance requirements apply to the device?

**Decision needed:** Define whether the device remains a personal prototype or targets sale/distribution.

**Areas to review:**

- FCC / CE radio requirements
- Battery shipping requirements
- RoHS
- WEEE
- Company security review
- Customer data handling review

**Why it matters:** Compliance scope changes hardware choices, labeling, documentation, and production planning.

**Blocks:**

- Manufacturing plan
- Packaging
- Distribution
- Security review

**Owner:** Ryan  
**Status:** Open

---

### TODO #25 — Factory reset confirmation behavior

**Question:** What exact visual confirmation pattern must precede factory reset?

**Decision needed:** Define reset confirmation sequence and cancellation behavior.

**Known candidate behaviors:**

- Red countdown across LEDs
- Triple red flash followed by reset
- Require second button press
- Require continued hold until countdown completes

**Why it matters:** Factory reset deletes credentials, bonded host data, config, and diagnostics. Accidental reset must be prevented.

**Blocks:**

- TRD section 6.4
- TRD section 16.4
- Firmware button handling tests

**Owner:** Ryan  
**Status:** Open

---

## Priority Order

Resolve in this order:

1. TODO #1 — Define monitored unit.
2. TODO #2 — LED count and layout.
3. TODO #3 — Health data sources.
4. TODO #4 — Gateway URL and deployment location.
5. TODO #12 — Health computation rules.
6. TODO #17 — Provisioning flow.
7. TODO #18 — Gateway authentication.
8. TODO #6 — Laptop Agent platform.
9. TODO #7 — Battery target.
10. TODO #19 — Initial monitored unit inventory.

## Development Can Begin Before These Are Resolved

The following implementation work can begin immediately:

- ESP-IDF project skeleton.
- LED renderer with simulated layout.
- Status payload parser.
- Freshness rules.
- Hysteresis rules.
- BLE GATT service skeleton.
- Python BLE write prototype.
- Static Health Gateway fixture server.
- Firmware unit test harness.

## Development Must Not Begin Before Resolution

The following implementation work must wait:

- PCB layout.
- Enclosure design.
- Battery subsystem.
- Production Gateway integration.
- Production authentication.
- Provisioning UX.
- OTA hosting.
- Agent packaging.
- Manufacturing test design.

