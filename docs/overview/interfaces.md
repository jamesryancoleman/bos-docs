# gRPC Service Interfaces

All core BOS services are defined in `services/common/common.proto` and communicate over gRPC. Services use a port convention based on `0xB05` (2821).

| Service        | Port | Purpose |
|----------------|------|---------|
| `System`       | 2821 | System management (service health, logs) |
| `Sysmod`       | 2822 | Semantic system model (devices, points, spaces) |
| `DeviceControl`| 2823 | Protocol-agnostic driver I/O |
| `History`      | 2824 | Time-series historian |
| `Forecast`     | 2825 | Model-generated point forecasts |
| `EventBus`     | 2826 | Kafka-backed pub/sub event bus |
| `Scheduler`    | 2827 | Container execution and scheduling |
| `HealthCheck`  | —    | Liveness probe, implemented by all services |

---

## DeviceControl

Protocol-agnostic driver interface. The main I/O gateway for physical devices.

| RPC | Description |
|-----|-------------|
| `Get` | Read one or more point values by URI |
| `Set` | Write one or more point values by URI |
<!-- | `ClearCache` | Flush the xref and driver resolution caches |
| `GetJobAccesses` | Return all point reads/writes logged for a given transaction ID | -->

---

## Sysmod

The system model and semantic registry — an RDF graph over devices, points, and spaces.

| RPC | Description |
|-----|-------------|
| `QueryDevices` | Filter devices by name, type, or location |
| `QueryPoints` | Filter points by name, type, location, or parent type |
| `SuggestPoints` | Find points matching a preferred Brick class, with a fallback floor class |
| `BasicQuery` | Run an arbitrary SPARQL-style query; returns triples |
| `GetName` | Resolve a point or device URI to its human-readable label |
| `GetDriver` | Resolve a point URI to its driver |
| `GetDriverXref` | Resolve a point URI to its protocol-specific xref address |
| `MakeDevice` | Register a new device in the model |
| `MakePoint` | Register a new point in the model |
| `MakeDriver` | Register a new driver in the model |
| `MakeSpace` | Register a new spatial location in the model |
| `Update` | Patch an entity's triples (deletions + additions in one call) |
| `Delete` | Remove an entity by URI, triple pattern, or query |

---

## History

Time-series historian.

| RPC | Description |
|-----|-------------|
| `GetHistory` | Fetch rows for a list of point keys over a time range |
| `GetSampleRate` | Read the current sample rate for a point |
| `SetSampleRate` | Configure how often a point is sampled |
<!-- | `RefreshRates` | Force the historian to reload its sampling config |
| `RefreshNames` | Force the historian to reload its point name cache | -->

---

## Scheduler

Runs and schedules containerized workloads.

| RPC | Description |
|-----|-------------|
| `Run` | One-shot container execution; returns stdout and exit code |
| `RunningJobs` | List currently active jobs |
| `Stop` | Cancel one or more running jobs |
| `GetJobDetail` | Full job record including params and point accesses |
| `CompletedJobs` | Session-scoped history of finished jobs |
| `RegisterCron` | Schedule a container to run on a cron expression |
| `CronTable` | List all registered cron jobs |
| `UnregisterCron` | Remove a cron job by UUID |
| `SetCronEnabled` | Enable or disable a cron job without removing it |
| `RegisterHandler` | Bind a container to fire when an event topic arrives |
| `EventHandlers` | List all registered event handlers |
| `UnregisterHandler` | Remove an event handler by UUID |
| `Library` | Browse available app images with metadata and env var specs |
| `DeleteApp` | Remove an app image from the library |
| `Get` / `Set` | Generic shared memory reads and writes |

---

## Forecast

Stores and retrieves model-generated forecasts for points.

| RPC | Description |
|-----|-------------|
| `Get` | Retrieve forecast entries for a point URI, optionally filtered by ID or time window |
| `Set` | Store a new forecast (model name, version, values with target timestamps, metadata) |

---

## EventBus

Kafka-backed pub/sub event bus.

| RPC | Description |
|-----|-------------|
| `Publish` | Emit an event to a topic with a typed payload and optional partition key |
| `Subscribe` | Server-streaming: receive live events from one or more topics, with server-side filtering and configurable start position (latest, earliest, offset, or timestamp) |
| `Replay` | Bounded server-streaming replay of a single topic from a past timestamp, offset, or event ID |

---

## HealthCheck

| RPC | Description |
|-----|-------------|
| `Ping` | Liveness probe (`Empty → Empty`), implemented by all services |
