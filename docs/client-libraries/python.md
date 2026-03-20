# bospy Python Library

`bospy` is the Python client library for OpenBOS. It wraps the underlying gRPC services into a clean Python API for reading sensor data, querying the system model, scheduling apps, and publishing events.

## Core Concepts

OpenBOS exposes three distinct data planes:

**Control Points** are physical or virtual measurement/actuation points identified by URIs (e.g. `"bos://host/dev/1/pts/1"`). They are backed by BACnet-style registers and served by the DevCtrl service. Point URIs are discovered at runtime via the system model or passed in as environment variables — hardcoding them is an anti-pattern.

**Shared Memory** is a Redis-backed key-value store used to pass data between apps and workflows. Keys are namespaced (e.g. `"global:upper_deadband"`). It is served by the Orchestrator/Scheduler.

**System Model** is an RDF graph that describes devices, points, spaces, and their semantic types and locations. It is queried through the Sysmod service.

---

## Reading & Writing Data

These functions are available directly on the `bospy` module.

## System Model

Query and manage the semantic model of the building.

```python
import bospy

# Find points by type or location
pts = bospy.query_points(types="brick:Air_Temperature_Sensor")
pts = bospy.query_points(locations="Room 101")
pts = bospy.query_points(types="brick:Zone_Air_Temperature_Sensor", locations="Floor 2")

# Find devices
devs = bospy.query_devices(types="brick:VAV")

# Create entities
bospy.make_device("ahu-1", types="brick:AHU", locations="Mechanical Room")
bospy.make_point("supply-air-temp", device="ahu-1", types="brick:Supply_Air_Temperature_Sensor")
bospy.make_space("Room 101", kind="brick:Room", parents=["Floor 1"])

# Fetch historical data
history = bospy.get_history(pts, start="2024-01-01", end="2024-01-02")
df = bospy.get_history(pts, start="2024-01-01", pandas=True)

# Modify or remove entities
bospy.update_entity(uri, updates=[{"s": uri, "p": "brick:hasLocation", "o": "Room 202"}])
bospy.delete_node(uri)
```

---

### Control Points

Point URIs are discovered via `query_points()` or received as environment variables. Once you have a URI, pass it to `get()` or `set()`.

```python
import bospy

# Discover points from the system model
pts = bospy.query_points(types="brick:Zone_Air_Temperature_Sensor", locations="Floor 2")

# Read one or more control points
values = bospy.get(pts)              # {"bos://host/dev/1/pts/1": 72.4, ...}
values = bospy.get(pts[0])          # {"bos://host/dev/1/pts/1": 72.4}

# Read values without keys (single value or tuple)
temp = bospy.get_values(pts[0])     # 72.4
temp, rh = bospy.get_values(pts[0], pts[1])

# Write a control point
bospy.set(pts[0], 70.0)
bospy.set(pts, [70.0, 65.0])
```

### Shared Memory

```python
import bospy

# Read from shared memory
result = bospy.load("global:setpoint")             # {"global:setpoint": 72.0}
result = bospy.load(["global:upper", "global:lower"])

# Read values without keys
setpoint = bospy.load_values("global:setpoint")    # 72.0

# Write to shared memory
bospy.store("global:setpoint", 72.0)
bospy.store({"global:upper": 75.0, "global:lower": 68.0})

# Expose app outputs (positional and keyword)
bospy.store_output(result1, result2, status="ok")
# Writes: OUTPUT/$1, OUTPUT/$2, OUTPUT/status
```

---

## Container Lifecycle

Apps running as containers receive context from the Scheduler via environment variables. Call `load_env()` at startup to populate this context.

```python
import bospy

bospy.load_env()  # reads TXN_ID, TOKEN, IMAGE, args, and kwargs from env

# Read outputs from a previously-run container
args, kwargs = bospy.load_input(app_name="my-other-app")
```

`load_env()` is called automatically when `bospy` is imported.

---

## Orchestration

Control app execution and scheduling via the `orch` submodule.

```python
from bospy import orch

# Run an app immediately
resp = orch.run("my-app-image", "arg1", key="value")
# timeout=-1 returns immediately; timeout=0 waits forever; timeout=N waits N seconds

# Schedule with a cron expression
orch.schedule("my-app-image", "0 * * * *")           # every hour
orch.schedule("my-app-image", "2024-06-01T08:00:00") # once at a specific time

# Inspect running and scheduled jobs
jobs = orch.get_running_apps()
jobs = orch.get_scheduled_apps()
jobs = orch.get_completed_apps()

# Stop or unschedule
orch.stop_apps([job_id])
orch.unschedule_app(cron_id)

# Register an app to run when an event fires
orch.register_handler("my-app-image", topic="sensor/alert")
orch.get_event_handlers()
orch.unregister_handler(handler_id)
```

---

## Events

Publish and subscribe to events on the Kafka-backed event bus.

```python
from bospy import events
import asyncio

# Publish an event
events.publish("sensor/alert", msg="high temp detected", kind="alert")

# Subscribe to one or more topics (async generator)
async def listen():
    async for event in events.subscribe(["sensor/alert", "sensor/fault"]):
        print(event.topic, event.payload)

asyncio.run(listen())
```

---

## Configuration

By default, `bospy` connects to services using hostnames and a port convention based on `0xB05` (2821):

| Service      | Default Address      | Env Var             |
|--------------|----------------------|---------------------|
| System       | `system:2821`        | `SYS_ADDR`          |
| Sysmod       | `sysmod:2822`        | `SYSMOD_ADDR`       |
| DevCtrl      | `devctrl:2823`       | `DEVCTRL_ADDR`      |
| History      | `history:2824`       | `HISTORY_ADDR`      |
| Forecast     | `forecast:2825`      | `FORECAST_ADDR`     |
| Events       | `events:2826`        | `EVENTS_ADDR`       |
| Orchestrator | `orchestrator:2827`  | `ORCHESTRATOR_ADDR` |

Override any address before making calls:

```python
import bospy

bospy.config.set_devctrl_addr("localhost:2823")

# Or reload all from environment variables
bospy.config.from_env()

# Inspect current config
print(bospy.config.get_config())
```

---

## Quick Reference

| Function | Purpose | Service |
|---|---|---|
| `bospy.get(pts)` | Read control points | DevCtrl |
| `bospy.get_values(*pts)` | Read control point values (no keys) | DevCtrl |
| `bospy.set(pts, values)` | Write control points | DevCtrl |
| `bospy.load(keys)` | Read shared memory | Orchestrator |
| `bospy.load_values(*keys)` | Read shared memory values (no keys) | Orchestrator |
| `bospy.store(key, val)` | Write shared memory | Orchestrator |
| `bospy.store_output(*args, **kw)` | Write app outputs to shared memory | Orchestrator |
| `bospy.query_points(...)` | Semantic point search | Sysmod |
| `bospy.query_devices(...)` | Semantic device search | Sysmod |
| `bospy.get_history(pts, ...)` | Fetch historical time-series | History |
| `bospy.make_device(...)` | Create a device entity | Sysmod |
| `bospy.make_point(...)` | Create a point entity | Sysmod |
| `bospy.make_space(...)` | Create a spatial location | Sysmod |
| `orch.run(app, ...)` | Run an app container | Orchestrator |
| `orch.schedule(app, cron)` | Schedule an app | Orchestrator |
| `orch.register_handler(app, topic)` | Trigger app on event | Orchestrator |
| `events.publish(topic, msg)` | Publish an event | Events |
| `events.subscribe(topics)` | Stream incoming events | Events |
