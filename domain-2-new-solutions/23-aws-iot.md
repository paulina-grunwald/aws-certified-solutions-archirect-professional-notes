# AWS IoT

> **Family of managed services for connecting, organizing, processing, and acting on data from IoT devices at scale — from MCU-class sensors to industrial equipment and connected vehicles.**

Maps to: **Domain 2.5 — Design high-performance architectures** (compute / data selection for IoT workloads)

---

## Service Portfolio

| Service | Purpose | Status |
|---|---|---|
| **IoT Core** | MQTT / HTTPS / WebSockets broker; device registry; rules engine | Active |
| **IoT Device Management** | Onboard, organize, monitor, remotely manage devices at scale | Active |
| **IoT Greengrass** | Edge runtime — local compute, ML inference, MQTT broker on-device | Active |
| **IoT Device Defender** | Security audit + anomaly detection for fleets | Active |
| **IoT SiteWise** | Industrial / OPC-UA equipment data ingestion + asset modeling | Active |
| **IoT FleetWise** | Vehicle telemetry (connected cars, fleets) | Active |
| **IoT TwinMaker** | Digital twins (3D scenes + live data overlays) | Active |
| **IoT Events** | Event detection from sensor streams (state machines) | **End of support May 20, 2026** — no new customers since May 2025 |
| **IoT Analytics** | Analytics pipeline for IoT data | **Deprecated** — migrate to Athena / Glue / OpenSearch |
| **IoT 1-Click** | Single-purpose device trigger (the IoT button) | **Discontinued Dec 16, 2024** |
| **IoT Things Graph** | Visual workflow builder for devices | Discontinued |

> ⚠️ **Exam still tests Events and 1-Click distinctions** even though both are EOL — the underlying concepts (event detection vs simple button trigger vs full device management) are still relevant for service-selection questions.

---

## AWS IoT Core

The foundation of every IoT workload on AWS.

- **Device Gateway**: managed MQTT, MQTT-over-WebSockets, HTTPS, and LoRaWAN endpoints
- **Message broker**: pub/sub at scale (millions of devices, billions of messages)
- **Device registry**: catalog of things, thing types, thing groups, with attributes and metadata
- **Authentication**: X.509 client certificates, AWS SigV4, Cognito identities, custom authorizers
- **Authorization**: IoT policies (JSON, similar to IAM but evaluated per MQTT topic / action)
- **Rules engine**: SQL-like rules that route incoming messages to: Lambda, DynamoDB, S3, SNS, SQS, Kinesis, Firehose, Step Functions, OpenSearch, Timestream, CloudWatch, EventBridge, IoT Events (legacy), republish to MQTT
- **Device Shadow**: persistent virtual representation of a device's state (desired vs reported); enables apps to read/write state even when device is offline
- **Jobs**: deploy remote operations (firmware updates, config changes) to fleets
- **Fleet Indexing**: search and aggregate device state via OpenSearch-style queries
- **Basic Ingest**: optimized path for sending data directly to rules without paying for MQTT messaging

### IoT Core for LoRaWAN
- Connect LoRaWAN devices and gateways without operating a Network Server
- Useful for low-power, wide-area-network sensors (agriculture, asset tracking)

---

## AWS IoT Device Management

> **Onboard, organize, monitor, and remotely manage IoT devices at scale.** Built on top of IoT Core's device registry.

- **Bulk onboarding**: import thousands of devices in one operation (CSV / JSON manifests)
- **Jobs**: deploy and track remote tasks (firmware updates, config, reboot) across thousands of devices
- **Fleet Hub**: console for browsing, searching, and acting on the device fleet
- **Secure tunneling**: open temporary, encrypted bidirectional connections (e.g., SSH) to a device behind a firewall — without exposing public ports
- **Remote actions**: send commands and receive responses (vs Jobs which are deployment-style)
- **Fleet Indexing**: query device state across the fleet (`"all devices reporting battery < 10%"`)
- **Software Package Catalog**: manage versioned software packages and deploy via Jobs

**Exam pattern**: scenario says **"securely onboard, organize, monitor, and remotely manage thousands of IoT devices"** → **IoT Device Management**.

---

## AWS IoT Greengrass

> **Edge runtime — extend AWS IoT to devices so they can act locally even when disconnected from the cloud.**

- Open-source edge runtime; runs Lambda, container components, ML models locally
- Local MQTT broker for device-to-device messaging without round-tripping to cloud
- **Component model**: bundle code + dependencies + lifecycle scripts as reusable components
- Deploy to fleets via the cloud (Greengrass core devices)
- Local data caching when offline; sync to cloud when reconnected
- **Stream Manager**: buffer and stream local data to Kinesis / S3 / IoT Analytics on a schedule
- **Machine Learning Inference**: run SageMaker-trained models locally on edge (GPU support on supported devices)
- **Greengrass V2** is the current major version (V1 EOL'd 2023)

**Use cases**: manufacturing floor, retail POS, vehicles, remote sites with unreliable connectivity, latency-sensitive industrial control loops.

> 💡 **Greengrass vs Outposts**: Greengrass runs on **your own commodity hardware** (Raspberry Pi to industrial gateways). Outposts is **AWS-supplied hardware** in your facility. Greengrass is software; Outposts is infrastructure.

---

## AWS IoT Device Defender

> **Security audit, anomaly detection, and continuous monitoring for IoT fleets.**

- **Audit**: continuously check configurations against IoT security best practices (over-permissive policies, reused certificates, etc.)
- **Detect**: machine-learning-based anomaly detection on device behavior (data volume, connection patterns, ports used)
- **ML Detect**: trains models on normal device behavior, flags deviations
- **Rules Detect**: customer-defined rules (e.g., alert if device sends > 100 MB / hour)
- Integrates with SNS and Lambda for automated response
- Supports both fleet-wide and per-device scoping

**Exam pattern**: scenario says **"detect compromised IoT devices in a large fleet"** or **"audit IoT security configuration"** → **IoT Device Defender**.

---

## AWS IoT SiteWise

> **Industrial IoT — collect, organize, analyze data from industrial equipment** (PLCs, OPC-UA servers, historians).

- **SiteWise Gateway**: software gateway that runs on a local server (or Greengrass) to read OPC-UA, Modbus, EtherNet/IP from PLCs and forward to AWS
- **Asset modeling**: define hierarchies (Site → Production Line → Machine → Sensor) and compute derived metrics (OEE, performance, throughput)
- **Storage tiers**: hot (recent data, fast access), cold (S3 long-term archival via SiteWise managed)
- **SiteWise Monitor**: managed web portal for operators to visualize asset data
- **Alarms**: integrate with CloudWatch Alarms after IoT Events end-of-life
- **Edge mode**: run SiteWise locally on Greengrass for low-latency / disconnected operation
- Time-series + asset model native; no need to build your own data store

**Exam pattern**: scenario says **"industrial equipment data with OPC-UA, KPIs, asset hierarchy"** → **IoT SiteWise**.

---

## AWS IoT FleetWise

> **Vehicle telemetry — connected cars and commercial fleets.**

- Designed for **vehicle CAN bus, OBD-II, J1939, automotive Ethernet** signals
- **Signal catalog**: standardized definitions of vehicle signals across makes/models
- **Decoder manifest**: maps raw CAN data to standardized signals
- **Vision system data**: camera and lidar metadata ingestion for ADAS / AV development
- **Edge agent**: runs on vehicle's in-vehicle computer; collects, filters, uploads on configurable triggers
- **Campaigns**: define what data to collect, from which vehicles, on what conditions

**Use cases**: predictive maintenance, ADAS / AV development, fleet management, EV battery analytics.

**Exam pattern**: scenario mentions **"connected vehicles," "CAN bus," "fleet telemetry"** → **IoT FleetWise**.

---

## AWS IoT TwinMaker

> **Digital twins — combine 3D scenes with live data from connected systems.**

- Build digital representations of physical systems (factories, buildings, energy assets)
- Pull data from **SiteWise, Kinesis Video, S3, custom connectors**
- Integrate with **Amazon Managed Grafana** for dashboards
- 3D scene composer for spatial visualization
- Useful for operations centers, virtual walkthroughs, simulation

**Exam pattern**: scenario mentions **"digital twin," "3D virtual representation of physical assets"** → **IoT TwinMaker**.

---

## AWS IoT Events (End of Support May 2026)

> **Detect and respond to events from IoT sensors and applications** using stateful detector models (state machines).

- Detector models react to telemetry: enter / leave states, trigger actions (Lambda, SNS, IoT topics, DynamoDB, etc.)
- Common use cases: temperature exceeds threshold → alert; equipment idle for 1 hour → schedule maintenance
- Stateful — remembers prior events and timing

### End-of-life timeline
- **No new customers since May 20, 2025**
- **End of support May 20, 2026**
- AWS recommends migrating detector models to **Kinesis + Lambda + DynamoDB + EventBridge Scheduler**
- For SiteWise alarms, migrate to **CloudWatch Alarms**

> ⚠️ **Still tested on exam** for service-selection questions. Recognize "detect state changes from IoT sensors and trigger alerts" as IoT Events on the exam, but in real-world architecture choose the modern alternative.

---

## AWS IoT Analytics (End of Support)

> **Analytics pipeline for IoT data** — collect, process, enrich, store, query, and visualize.

- **Status**: deprecated; AWS recommends migrating to **Athena + Glue + S3 + OpenSearch + QuickSight**
- Exam may still reference it; modern architecture uses the alternative stack

---

## AWS IoT 1-Click (Discontinued Dec 16, 2024)

> **Trigger AWS Lambda from simple, single-purpose devices** (the original "IoT button").

- **Status**: DISCONTINUED — APIs, console, and mobile apps no longer functional as of Dec 16, 2024
- For new equivalent solutions, use **IoT Core + custom buttons** or commercial off-the-shelf buttons that publish MQTT directly

> ⚠️ **Exam still uses 1-Click as a distractor** — the answer is typically "1-Click is wrong because it's for single-purpose buttons, not fleet management." The exam tests whether you can disqualify it.

---

## Decision Matrix — Which AWS IoT Service?

| Scenario | Best Service |
|---|---|
| Connect devices, route messages, manage device state | **IoT Core** |
| Onboard, organize, monitor, remotely manage many devices | **IoT Device Management** |
| Run code / ML at the edge with intermittent connectivity | **IoT Greengrass** |
| Detect compromised devices, audit fleet security | **IoT Device Defender** |
| Industrial equipment data (OPC-UA, PLCs, historians) | **IoT SiteWise** |
| Vehicle telemetry (CAN bus, fleets, ADAS) | **IoT FleetWise** |
| Digital twin / 3D visualization of physical system | **IoT TwinMaker** |
| Detect state changes in sensors and trigger alerts (legacy) | **IoT Events** (EOL May 2026) → migrate to Lambda + EventBridge |
| Single-purpose button trigger (legacy) | **IoT 1-Click** (discontinued) → use IoT Core |
| Long-term IoT analytics (legacy) | **IoT Analytics** (deprecated) → Athena + Glue + S3 |

---

## Common SAP-C02 Scenario Patterns

### "Securely onboard, organize, monitor, and remotely manage IoT devices"
→ **AWS IoT Device Management** (the screenshot example). NOT IoT 1-Click (single-purpose), NOT IoT Events (stateful event detection, not device management).

### "Detect anomalies / detect state of sensor data and trigger maintenance alerts"
→ Historically **IoT Events**; for new designs use **Kinesis + Lambda + EventBridge + DynamoDB** (Events EOL May 2026). Exam may still expect IoT Events.

### "Run ML inference locally on factory floor with unreliable internet"
→ **IoT Greengrass** (with SageMaker-trained model deployed as a component).

### "Connect industrial PLCs to AWS with OPC-UA, model assets as hierarchy"
→ **IoT SiteWise** (with SiteWise Gateway running on-prem or via Greengrass).

### "Detect compromised IoT devices across a 100,000-device fleet"
→ **IoT Device Defender** (ML Detect or Rules Detect).

### "Open temporary SSH tunnel to a device behind a corporate firewall"
→ **IoT Device Management — Secure Tunneling** (no need to expose ports).

### "Stream connected vehicle data to AWS for fleet analytics"
→ **IoT FleetWise**.

### "Real-time 3D visualization of factory equipment with live data overlay"
→ **IoT TwinMaker** (with SiteWise and Managed Grafana).

---

## Integration with Other AWS Services

- **Rules Engine → Kinesis Data Streams** for fan-out to analytics / Lambda / OpenSearch
- **Rules Engine → S3** for cheap raw data lake
- **Rules Engine → Timestream** for time-series queries
- **Rules Engine → SQS / SNS** for application integration
- **Rules Engine → DynamoDB** for direct device-to-table writes
- **Rules Engine → Step Functions** for complex workflows triggered by device events
- **Rules Engine → EventBridge** for general-purpose event distribution (replaces some IoT Events use cases)
- **Greengrass + Lambda / Containers** for local edge logic
- **Greengrass + SageMaker** for edge ML inference
- **SiteWise + Managed Grafana / QuickSight** for industrial dashboards

---

## Security Considerations

- **X.509 certificates** are the default device auth — provisioning via "Fleet Provisioning" templates
- **IoT policies** are evaluated per MQTT topic and action (subscribe / publish / receive / connect)
- Use **`thingname`** policy variables to scope a device's permissions to its own topics: `iot:Publish` on `iot:topic/${iot:Connection.Thing.ThingName}/*`
- **Per-device certificates** (not shared) — IoT Device Defender flags reused certificates as audit failures
- Use **AWS IoT Device Defender** to continuously audit
- For edge: **Greengrass** uses its own X.509 certs and IAM roles via the AWS IoT Core credential provider
- **Just-in-time provisioning (JITP)** — auto-register devices on first connect with valid CA-signed cert
- **Fleet Provisioning by claim** — devices use a shared bootstrap cert, then receive unique cert on first connect

---

## Exam Tips

- **IoT Core** is the foundation — message broker, device registry, rules engine, device shadow. Most IoT scenarios start here.
- **IoT Device Management** is the answer for: onboarding, fleet organization, remote management, jobs, secure tunneling
- **IoT Greengrass** is the answer for: local processing, edge ML, offline operation, low-latency control loops
- **IoT Device Defender** is the answer for: security audits, anomaly detection in fleets
- **IoT SiteWise** is the answer for: industrial equipment, OPC-UA, asset hierarchies, OEE / KPIs
- **IoT FleetWise** is the answer for: vehicles, CAN bus, fleets
- **IoT TwinMaker** is the answer for: digital twins, 3D visualization
- For service-selection questions, **disqualify legacy/EOL services as distractors**: 1-Click (single button), Events (state detection — EOL 2026), Analytics (deprecated)
- **Rules Engine** is the integration glue — it routes MQTT messages to 20+ AWS services without needing custom code
- **Device Shadow** is for offline-capable apps — read/write desired state even when device is disconnected
- **Secure Tunneling** is the answer for "SSH into device behind firewall without opening ports"
- **Greengrass V2 component model** is more flexible than V1 (containers + Lambda + arbitrary processes)
- **Authentication options** in IoT Core: X.509 (default), SigV4, Cognito, custom authorizer Lambda
- For "many devices, scale to millions" scenarios → IoT Core handles the connection scaling natively

---

## Exam Traps

- IoT 1-Click was **discontinued Dec 2024** — if it appears as a choice for fleet management, it is wrong
- IoT Events reaches **end of support May 2026** — for new designs, prefer Kinesis + Lambda + EventBridge, though it still appears as the exam answer for state-detection scenarios
- IoT Analytics is **deprecated** — the replacement stack is Athena + Glue + S3
- IoT Core is the **protocol broker + registry**; IoT Device Management adds **fleet operations** on top — they are separate services
- Greengrass is **software on your hardware**; Outposts is **AWS-supplied hardware** in your facility — don't confuse them
- Greengrass **V2 is the current version** (V1 reached EOL June 2023) — V2 has a different architecture
- MQTT messages have a **128 KB payload limit** — for larger data, use Basic Ingest with S3 directly via the rules engine
- IoT policies are evaluated **per MQTT topic** and use IoT-specific action names (`iot:Publish`, not `s3:PutObject`) — they are not IAM policies despite looking similar
- Device Shadow is **per-device** — for fleet-wide queries, use Fleet Indexing instead
- The Rules Engine routes **individual messages** — for aggregation and windowing, use Kinesis Data Analytics / Managed Service for Apache Flink downstream
- SiteWise is **specifically for industrial OPC-UA / PLC data** — it is not a general-purpose IoT service
- FleetWise is **automotive-only** (CAN bus, vehicle fleets) — don't pick it for generic device telemetry
- TwinMaker is a **visualization overlay** on existing data sources (SiteWise, S3, Kinesis Video) — it does not ingest data itself
- Device-to-device auth uses **X.509 certificates** — Cognito Identity Pools can grant IoT permissions to mobile / browser apps but not for device-to-device communication
- IoT Core supports **connect-publish-disconnect** patterns — there is no "always-on connection" requirement. The broker holds messages briefly for QoS 1 delivery
