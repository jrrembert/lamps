# Portable App Health Light Array — Technical Requirements Document

Status: Draft pending open questions  
Owner: Ryan Rembert  
Document version: 0.1  
Last updated: 2026-05-22

## 1. Purpose

This document defines the implementation contract for the Portable App Health Light Array. The device displays application health using an RGB light array. Each light maps to one monitored unit. The definition of monitored unit is blocked by [TODO #1].

This document contains deterministic implementation requirements. Each unresolved decision appears as a numbered TODO linked to the Outstanding Design Questions document.

## 2. Product Summary

The product is a portable desk device with a microcontroller, wireless connectivity, and an RGB LED array. The device receives normalized health summaries and renders one status light per monitored unit.

The device supports two data paths:

1. Wi-Fi direct polling from the Health Gateway.
2. Bluetooth Low Energy ingestion from the Laptop Agent.

The device computes freshness, source precedence, hysteresis, and LED rendering. The device does not compute business health from raw metrics.

## 3. Scope

### 3.1 In Scope

- ESP32-S3 based portable hardware.
- RGB LED array rendering.
- Wi-Fi connection management.
- HTTPS polling of the Health Gateway.
- BLE GATT server for Laptop Agent ingestion.
- Local device configuration stored in encrypted non-volatile storage.
- BLE pairing gated by physical button interaction.
- Stale data handling.
- Status hysteresis.
- USB-C power.
- Battery support blocked by [TODO #7].
- Firmware OTA using signed images.
- Python prototype Laptop Agent.
- Health Gateway contract.
- Development, test, and deployment plan.

### 3.2 Out of Scope

- On-device Prometheus query execution.
- On-device Datadog query execution.
- On-device Sentry query execution.
- On-device incident management workflows.
- On-device text display.
- Audible alerts.
- Mobile application.
- Multi-user cloud account management.
- Manufacturing test fixture blocked by [TODO #13].

## 4. Definitions

| Term | Definition |
|---|---|
| Device | Portable App Health Light Array hardware running ESP32-S3 firmware. |
| Monitored Unit | The entity represented by one LED. Exact entity definition blocked by [TODO #1]. |
| Health Gateway | Backend HTTP service that emits normalized health summaries. |
| Laptop Agent | Local desktop process that sends normalized health summaries to the device through BLE. |
| Status | One of `green`, `yellow`, `red`, `unknown`, `maintenance`, `muted`. |
| Fresh Data | Status payload with age less than its `ttl_seconds` value. |
| Stale Data | Status payload with age greater than its `ttl_seconds` value. |

## 5. Architecture Decisions

### ADR-001: Microcontroller Platform

The device will use an ESP32-S3 microcontroller.

Consequences:

- Firmware will be implemented with ESP-IDF.
- Wi-Fi will use the ESP-IDF Wi-Fi stack.
- BLE will use the ESP-IDF NimBLE stack.
- Native USB will be used for development logs and factory flashing.

### ADR-002: Health Classification Location

Health classification will run outside the device.

Consequences:

- The Health Gateway emits final status values.
- The Laptop Agent emits final status values.
- The device validates payload shape, sequence, source, and freshness.
- The device renders statuses without raw metric interpretation.

### ADR-003: BLE Role

The device will be the BLE peripheral and GATT server. The Laptop Agent will be the BLE central and GATT client.

Consequences:

- The Laptop Agent initiates connections.
- The device advertises only during pairing mode and reconnect windows.
- The Laptop Agent writes status batches to the device.

### ADR-004: Data Contract

The Health Gateway and Laptop Agent will use the same normalized status batch schema.

Consequences:

- Device firmware has one health ingestion model.
- BLE ingestion and Wi-Fi polling feed the same internal state machine.
- Test fixtures use the same payloads across transports.

### ADR-005: Default Source Precedence

The device will use BLE data when a bonded Laptop Agent is connected. The device will use Wi-Fi direct polling when no bonded Laptop Agent is connected.

Consequences:

- BLE is the active source during local development sessions.
- Wi-Fi is the active source during standalone operation.
- Source switching never merges data from both paths.

## 6. Hardware Requirements

### 6.1 Core Board

| ID | Requirement |
|---|---|
| HW-001 | The device shall use an ESP32-S3 module. |
| HW-002 | The firmware shall target ESP-IDF. |
| HW-003 | The board shall expose USB-C for power and serial diagnostics. |
| HW-004 | The board shall include one physical button. |
| HW-005 | The board shall include a physical power switch. |
| HW-006 | The board shall expose test pads for 3.3 V, GND, UART TX, UART RX, EN, BOOT, LED data, and battery sense. |

### 6.2 LED Array

| ID | Requirement |
|---|---|
| LED-001 | The device shall use individually addressable RGB LEDs. |
| LED-002 | LED count and physical layout are blocked by [TODO #2]. |
| LED-003 | Each enabled monitored unit shall map to exactly one LED index. |
| LED-004 | LED brightness shall be firmware capped at 30 percent. |
| LED-005 | LED rendering shall never drive all LEDs at full white. |
| LED-006 | The enclosure shall include a diffuser over the LED array. Diffuser material and thickness are blocked by [TODO #14]. |

### 6.3 Power

| ID | Requirement |
|---|---|
| PWR-001 | The device shall accept USB-C power. |
| PWR-002 | The device shall support battery operation. Battery capacity is blocked by [TODO #7]. |
| PWR-003 | The firmware shall expose battery percentage when battery measurement hardware is present. Battery gauge approach is blocked by [TODO #7]. |
| PWR-004 | The device shall boot automatically when external power is applied and the physical switch is enabled. |

### 6.4 Button Behavior

| Action | Behavior |
|---|---|
| Short press | Cycle brightness across 5, 15, and 30 percent. |
| Double press | Toggle active mode between `auto`, `wifi`, and `ble`. |
| Hold 3 seconds | Enter BLE pairing mode for 120 seconds. |
| Hold 10 seconds | Factory reset after visual confirmation pattern. Confirmation pattern blocked by [TODO #25]. |

## 7. Firmware Requirements

### 7.1 Firmware Modules

The firmware shall contain these modules:

```text
firmware/
  app_main.c
  connectivity/
    wifi_manager.c
    ble_gatt_server.c
    provisioning.c
  health/
    health_model.c
    health_http_client.c
    health_ble_ingest.c
    stale_detector.c
    hysteresis.c
  display/
    led_driver.c
    renderer.c
    animations.c
    brightness.c
  storage/
    config_store.c
    secrets_store.c
  system/
    ota.c
    diagnostics.c
    watchdog.c
    power.c
```

### 7.2 Runtime Tasks

The firmware shall start these FreeRTOS tasks:

| Task | Responsibility |
|---|---|
| `connectivity_task` | Manage Wi-Fi state, BLE state, source precedence, reconnect timing. |
| `health_task` | Fetch Wi-Fi summaries, ingest BLE updates, apply sequence validation, apply freshness, apply hysteresis. |
| `led_task` | Render LED state at 30 FPS. |
| `button_task` | Interpret button gestures. |
| `diagnostics_task` | Track uptime, heap, Wi-Fi RSSI, BLE connection state, active source, last payload sequence. |
| `ota_task` | Execute firmware update checks and update installation. |

### 7.3 Source State Machine

The device shall maintain one active source.

```text
if configured_mode == "ble":
    active_source = BLE

if configured_mode == "wifi":
    active_source = WIFI

if configured_mode == "auto" and bonded_ble_agent_connected == true:
    active_source = BLE

if configured_mode == "auto" and bonded_ble_agent_connected == false:
    active_source = WIFI
```

The state machine shall never combine status items from BLE and Wi-Fi.

### 7.4 Status Model

The firmware shall maintain this internal status model:

```c
typedef enum {
    HEALTH_UNKNOWN = 0,
    HEALTH_GREEN = 1,
    HEALTH_YELLOW = 2,
    HEALTH_RED = 3,
    HEALTH_MAINTENANCE = 4,
    HEALTH_MUTED = 5
} health_status_t;
```

### 7.5 Freshness Rules

For each monitored unit:

| Condition | Rendered State |
|---|---|
| `age_seconds <= ttl_seconds` | Reported status. |
| `ttl_seconds < age_seconds <= ttl_seconds * 3` | Stale variant of last reported status. |
| `age_seconds > ttl_seconds * 3` | `unknown`. |
| Unit missing from latest payload | `unknown`. |

### 7.6 Hysteresis Rules

The firmware shall apply these transition rules per monitored unit:

| Transition | Required Samples |
|---|---:|
| `green` to `yellow` | 2 consecutive `yellow` samples |
| `green` to `red` | 2 consecutive `red` samples |
| `yellow` to `red` | 2 consecutive `red` samples |
| `red` to `yellow` | 2 consecutive `yellow` samples |
| `red` to `green` | 3 consecutive `green` samples |
| `yellow` to `green` | 3 consecutive `green` samples |
| any status to `maintenance` | 1 sample |
| any status to `muted` | 1 sample |
| any status to `unknown` | freshness rule |

The minimum visible dwell time shall be 5 seconds.

### 7.7 LED Rendering

| Status | Rendering |
|---|---|
| `green` | Steady green. |
| `yellow` | Yellow breathing animation with 2 second period. |
| `red` | Red blink with 1 second period. |
| `unknown` | Dim gray breathing animation with 4 second period. |
| `maintenance` | Steady blue. |
| `muted` | Dim purple. |
| stale `green` | Dim green breathing animation with 4 second period. |
| stale `yellow` | Dim yellow breathing animation with 3 second period. |
| stale `red` | Dim red breathing animation with 2 second period. |

RGB values and gamma curve are blocked by [TODO #20].

## 8. Wi-Fi Direct Polling

### 8.1 Wi-Fi Behavior

| ID | Requirement |
|---|---|
| WIFI-001 | The device shall store one Wi-Fi network credential set. Multi-network behavior is blocked by [TODO #22]. |
| WIFI-002 | The device shall use HTTPS for all Health Gateway calls. |
| WIFI-003 | The device shall validate the server certificate chain. |
| WIFI-004 | The device shall poll the Health Gateway every 15 seconds. Poll cadence is blocked by [TODO #11]. |
| WIFI-005 | The device shall time out each Health Gateway request after 5 seconds. |
| WIFI-006 | The device shall retain the last valid payload while freshness rules allow it. |

### 8.2 Health Gateway Endpoint

The device shall call:

```http
GET /iot/v1/device/{device_id}/summary
Authorization: Bearer {device_token}
Accept: application/json
```

Gateway hostname is blocked by [TODO #4].

### 8.3 Health Gateway Response

The Health Gateway shall return:

```json
{
  "version": 1,
  "sequence": 1842,
  "generated_at": "2026-05-22T19:42:31Z",
  "ttl_seconds": 30,
  "items": [
    {
      "id": "acme-prod",
      "label": "Acme",
      "status": "green",
      "message": "healthy"
    }
  ]
}
```

### 8.4 Response Validation

The firmware shall reject a response when any rule fails:

| Rule |
|---|
| `version` equals `1`. |
| `sequence` is greater than the last accepted sequence from the active source. |
| `generated_at` is valid RFC 3339 UTC timestamp text. |
| `ttl_seconds` is greater than 0 and less than 300. |
| `items` length is less than the configured LED capacity plus 1. |
| Each `id` matches a configured monitored unit. |
| Each `status` is a valid status enum value. |

Rejected responses shall increment `diagnostics.rejected_payload_count`.

## 9. BLE Ingestion

### 9.1 BLE Service

The device shall expose this custom BLE service:

```text
Service UUID: 7e2a0000-8f4d-4f6f-9f7a-000000000001
```

### 9.2 BLE Characteristics

| Characteristic | UUID | Properties | Purpose |
|---|---|---|---|
| Device Info | `7e2a0001-8f4d-4f6f-9f7a-000000000001` | Read | Firmware version, device ID, capabilities. |
| Status Batch | `7e2a0002-8f4d-4f6f-9f7a-000000000001` | Write | Laptop Agent sends status batch payloads. |
| Config | `7e2a0003-8f4d-4f6f-9f7a-000000000001` | Read, Write | Laptop Agent reads and updates device config. |
| Command | `7e2a0004-8f4d-4f6f-9f7a-000000000001` | Write | Laptop Agent sends commands. |
| Events | `7e2a0005-8f4d-4f6f-9f7a-000000000001` | Notify | Device emits acknowledgements and diagnostics. |

### 9.3 BLE Pairing

| ID | Requirement |
|---|---|
| BLE-001 | The device shall accept new BLE bonding only during pairing mode. |
| BLE-002 | Pairing mode shall be entered by holding the physical button for 3 seconds. |
| BLE-003 | Pairing mode shall expire after 120 seconds. |
| BLE-004 | The device shall accept status writes only from a bonded central. |
| BLE-005 | The device shall store exactly one bonded Laptop Agent. Multi-host behavior is blocked by [TODO #6]. |

### 9.4 BLE Status Payload

The Laptop Agent shall write UTF-8 JSON to the Status Batch characteristic.

```json
{
  "version": 1,
  "sequence": 1029,
  "generated_at": "2026-05-22T19:42:31Z",
  "ttl_seconds": 30,
  "items": [
    {
      "id": "acme-prod",
      "label": "Acme",
      "status": "green",
      "message": "healthy"
    }
  ]
}
```

The firmware shall apply the same validation rules used for Wi-Fi responses.

Large BLE payload chunking is blocked by [TODO #2] and [TODO #11].

## 10. Configuration

### 10.1 Device Config Shape

The device shall store this configuration object:

```json
{
  "schema_version": 1,
  "device_name": "Portable Health Array",
  "device_id": "pha-00000001",
  "mode": "auto",
  "brightness_percent": 15,
  "layout": {
    "rows": 0,
    "cols": 0,
    "orientation": "row-major"
  },
  "monitored_units": [
    {
      "id": "acme-prod",
      "label": "Acme",
      "led_index": 0,
      "enabled": true
    }
  ],
  "wifi": {
    "enabled": true
  },
  "direct_poll": {
    "path": "/iot/v1/device/{device_id}/summary",
    "interval_seconds": 15,
    "timeout_seconds": 5
  }
}
```

`layout.rows`, `layout.cols`, and monitored unit list are blocked by [TODO #2] and [TODO #1].

### 10.2 Secrets

The device shall store secrets separately from general configuration.

Stored secrets:

| Secret | Usage |
|---|---|
| Wi-Fi SSID | Network connection. |
| Wi-Fi password | Network connection. |
| Device token | Health Gateway authentication. |

Secrets shall be stored in encrypted NVS.

### 10.3 Provisioning

Provisioning flow is blocked by [TODO #17].

## 11. Laptop Agent

### 11.1 Runtime

The prototype Laptop Agent shall be implemented in Python.

The Laptop Agent shall:

| ID | Requirement |
|---|---|
| AGENT-001 | Connect to the bonded device through BLE. |
| AGENT-002 | Fetch normalized status batches from the Health Gateway. |
| AGENT-003 | Write status batches to the BLE Status Batch characteristic. |
| AGENT-004 | Reconnect after BLE disconnection. |
| AGENT-005 | Log connection state, last sequence, last write result, and payload validation failures. |

Supported desktop platforms are blocked by [TODO #6].

### 11.2 Agent Config

The prototype agent shall read:

```yaml
device_name: "Portable Health Array"
polling_interval_seconds: 15
gateway_base_url: "https://example.invalid"
device_id: "pha-00000001"
```

The real gateway base URL is blocked by [TODO #4].

## 12. Health Gateway

### 12.1 Gateway Contract

The Health Gateway shall expose the normalized summary endpoint defined in section 8.

### 12.2 Gateway Inputs

The Health Gateway input systems are blocked by [TODO #3].

### 12.3 Status Computation

The status computation rules are blocked by [TODO #12].

### 12.4 Authentication

Gateway authentication strategy is blocked by [TODO #18].

## 13. Security Requirements

| ID | Requirement |
|---|---|
| SEC-001 | All Wi-Fi health traffic shall use HTTPS. |
| SEC-002 | Firmware OTA images shall be signed. |
| SEC-003 | Device secrets shall be stored in encrypted NVS. |
| SEC-004 | BLE status writes shall require a bonded central. |
| SEC-005 | New BLE bonding shall require physical button pairing mode. |
| SEC-006 | Factory reset shall erase Wi-Fi credentials, device token, bonded central data, config, and diagnostics counters. |
| SEC-007 | The device shall never store raw health metric history. |
| SEC-008 | The device shall store only the latest accepted payload per active source. |
| SEC-009 | Privacy mode behavior is blocked by [TODO #9]. |

## 14. Diagnostics

The device shall expose diagnostics through serial logs and BLE Device Info.

Diagnostics fields:

```json
{
  "firmware_version": "0.1.0",
  "device_id": "pha-00000001",
  "uptime_seconds": 12345,
  "active_source": "wifi",
  "wifi_connected": true,
  "wifi_rssi": -55,
  "ble_connected": false,
  "last_sequence": 1842,
  "last_payload_age_seconds": 4,
  "rejected_payload_count": 0,
  "free_heap_bytes": 123456,
  "battery_percent": 82
}
```

Persistent diagnostics retention is blocked by [TODO #23].

## 15. OTA Requirements

| ID | Requirement |
|---|---|
| OTA-001 | The firmware shall support HTTPS OTA updates. |
| OTA-002 | OTA update checks shall run at boot and once every 24 hours. |
| OTA-003 | OTA manifest URL is blocked by [TODO #15]. |
| OTA-004 | OTA installation shall use dual-partition rollback. |
| OTA-005 | The device shall show a white progress sweep during OTA installation. |
| OTA-006 | The device shall mark new firmware valid only after successful boot, Wi-Fi init, BLE init, config load, and LED init. |

## 16. Test Plan

### 16.1 Firmware Unit Tests

| Test ID | Coverage |
|---|---|
| FW-UT-001 | Status enum parsing. |
| FW-UT-002 | Payload schema validation. |
| FW-UT-003 | Sequence rejection. |
| FW-UT-004 | Freshness state transitions. |
| FW-UT-005 | Hysteresis transitions. |
| FW-UT-006 | LED color mapping. |
| FW-UT-007 | Button gesture parsing. |
| FW-UT-008 | Config schema validation. |

### 16.2 Firmware Integration Tests

| Test ID | Coverage |
|---|---|
| FW-IT-001 | Wi-Fi connection with valid credentials. |
| FW-IT-002 | Health Gateway polling success. |
| FW-IT-003 | Health Gateway timeout. |
| FW-IT-004 | BLE pairing gated by button. |
| FW-IT-005 | BLE status write from bonded central. |
| FW-IT-006 | BLE status write rejection from unbonded central. |
| FW-IT-007 | OTA success path. |
| FW-IT-008 | OTA rollback path. |

### 16.3 Laptop Agent Tests

| Test ID | Coverage |
|---|---|
| AG-UT-001 | Gateway payload fetch. |
| AG-UT-002 | BLE device discovery. |
| AG-UT-003 | BLE reconnect. |
| AG-UT-004 | Status batch write. |
| AG-UT-005 | Invalid payload logging. |

### 16.4 Acceptance Tests

| Test ID | Scenario | Expected Result |
|---|---|---|
| AT-001 | Gateway returns green status. | Target LED renders steady green. |
| AT-002 | Gateway returns yellow status twice. | Target LED renders yellow breathing animation. |
| AT-003 | Gateway returns red status twice. | Target LED renders red blinking animation. |
| AT-004 | Gateway stops responding. | Target LED enters stale state, then unknown. |
| AT-005 | Laptop Agent connects. | Active source changes to BLE. |
| AT-006 | Laptop Agent disconnects. | Active source changes to Wi-Fi. |
| AT-007 | Button held 3 seconds. | Device enters pairing mode for 120 seconds. |
| AT-008 | Button held 10 seconds. | Device executes factory reset after confirmation pattern. |
| AT-009 | OTA image fails boot validation. | Device rolls back to previous image. |

## 17. Deployment Plan

### 17.1 Firmware

| Stage | Action |
|---|---|
| Dev | Build firmware with local secrets disabled. |
| Staging | Flash signed staging image to test devices. |
| Production | Publish signed image to OTA endpoint. |

Firmware release ownership is blocked by [TODO #16].

### 17.2 Laptop Agent

| Stage | Action |
|---|---|
| Dev | Run Python agent from source. |
| Staging | Package agent with pinned dependencies. |
| Production | Distribution channel blocked by [TODO #6]. |

### 17.3 Health Gateway

| Stage | Action |
|---|---|
| Dev | Serve static fixture payloads. |
| Staging | Integrate with staging health inputs. |
| Production | Integrate with production health inputs. |

Production health inputs are blocked by [TODO #3].

## 18. Linear Implementation Plan

### Epic 1: Firmware Foundation

| Task | Description |
|---|---|
| PHA-001 | Create ESP-IDF project skeleton. |
| PHA-002 | Implement config schema and encrypted NVS storage. |
| PHA-003 | Implement LED driver and renderer. |
| PHA-004 | Implement button gesture handler. |
| PHA-005 | Implement diagnostics model. |

### Epic 2: Health State Machine

| Task | Description |
|---|---|
| PHA-006 | Implement status payload parser. |
| PHA-007 | Implement payload validation. |
| PHA-008 | Implement freshness rules. |
| PHA-009 | Implement hysteresis rules. |
| PHA-010 | Implement active source precedence. |

### Epic 3: Wi-Fi Direct Path

| Task | Description |
|---|---|
| PHA-011 | Implement Wi-Fi connection manager. |
| PHA-012 | Implement HTTPS Health Gateway client. |
| PHA-013 | Implement gateway response ingestion. |
| PHA-014 | Implement Wi-Fi failure handling. |

### Epic 4: BLE Path

| Task | Description |
|---|---|
| PHA-015 | Implement BLE GATT service. |
| PHA-016 | Implement BLE pairing mode. |
| PHA-017 | Implement Status Batch characteristic. |
| PHA-018 | Implement Config characteristic. |
| PHA-019 | Implement Events characteristic. |

### Epic 5: Laptop Agent

| Task | Description |
|---|---|
| PHA-020 | Create Python agent project. |
| PHA-021 | Implement Gateway polling. |
| PHA-022 | Implement BLE discovery and connection. |
| PHA-023 | Implement Status Batch writes. |
| PHA-024 | Implement agent logging. |

### Epic 6: OTA and Release

| Task | Description |
|---|---|
| PHA-025 | Implement HTTPS OTA check. |
| PHA-026 | Implement signed firmware validation. |
| PHA-027 | Implement rollback validation. |
| PHA-028 | Create firmware build pipeline. |

### Epic 7: Test and Validation

| Task | Description |
|---|---|
| PHA-029 | Create firmware unit test suite. |
| PHA-030 | Create BLE integration test harness. |
| PHA-031 | Create Wi-Fi gateway fixture server. |
| PHA-032 | Execute acceptance test pass. |

## 19. Development Blockers

Development can begin on firmware skeleton, LED rendering, payload validation, and Laptop Agent BLE connection.

Development is blocked on these items:

- [TODO #1]
- [TODO #2]
- [TODO #3]
- [TODO #4]
- [TODO #6]
- [TODO #7]
- [TODO #11]
- [TODO #12]
- [TODO #17]
- [TODO #18]

## 20. Appendix: Canonical Status Batch Schema

```json
{
  "version": 1,
  "sequence": 1,
  "generated_at": "2026-05-22T00:00:00Z",
  "ttl_seconds": 30,
  "items": [
    {
      "id": "string",
      "label": "string",
      "status": "green",
      "message": "string"
    }
  ]
}
```

Allowed `status` values:

```text
green
yellow
red
unknown
maintenance
muted
```

