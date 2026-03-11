# Driver Quickstart

A driver is a **reverse proxy**: it speaks gRPC toward the BOS platform and speaks a device- or protocol-specific language (BACnet, Modbus, Kasa, HTTP, etc.) toward physical equipment. From the platform's perspective all drivers look identical — they are gRPC servers that implement two methods.

## The two-method contract

Every driver implements the `DeviceControl` gRPC service:

| Method | Direction | What it does |
|--------|-----------|--------------|
| `Get(keys[]) → pairs[]` | platform → driver → device | Read one or more values |
| `Set(pairs[]) → pairs[]` | platform → driver → device | Write one or more values |

That is the entire public surface. Nothing else is required.

## Keys are xrefs, not point URIs

Client applications address data using BOS point URIs:

```
bos://localhost/dev/12/pts/1
```

The Device Control service (`devctrl`) resolves these against the system model before forwarding the request. By the time a key reaches your driver it is already an **xref** — a protocol-specific URI that encodes everything needed to locate the value on the physical device:

```
kasa://192.168.1.42:9999/power
bacnet://10.0.0.5/analog-input/1/present-value
modbus://10.0.0.6:502/holding/40001
```

Your driver does not need to know about BOS point URIs at all. Just parse the xref and issue the device request.

## Data flow

```
[app]
  │  bos.get(["bos://localhost/dev/12/pts/1"])    ← point URI
  ▼
[devctrl]
  │  Get(["kasa://192.168.1.42:9999/power"])      ← xref (after lookup)
  ▼
[kasa driver]
  │  HTTP / proprietary protocol
  ▼
[physical device]
```

The translation from point URI to xref is owned entirely by `devctrl`. The driver never sees a point URI.

## Statelessness

Drivers are designed to be stateless across requests. There is no session, no cache, no in-memory device registry that must stay warm. Each Get/Set call is self-contained — the xref carries all the addressing information.

The one exception is **boot-time configuration** — credentials, API tokens, and other secrets that cannot live in the xref. Load these once at startup from shared memory:

```python
config = bos.load(["global:my_driver_token"])
TOKEN = config["global:my_driver_token"]
```

Shared memory is the canonical location for configuration that needs to change without rebuilding the image.

## Deployment

Each driver ships as a Docker image. The platform starts one container per driver type; a single instance typically handles requests for all devices of that protocol. The container exposes a single gRPC port — nothing else.

## Writing a driver

1. Implement the `DeviceControl` gRPC service (`Get` + `Set`)
2. Parse the xref URI on each request to extract host, port, and addressing parameters
3. Issue the device-specific call and map the result back into the response message
4. On startup, load any credentials or configuration from shared memory
5. Package as a Docker image

See [services/drivers/grpc-example/](../../services/drivers/grpc-example/) for a minimal working implementation and [services/drivers/kasa/](../../services/drivers/kasa/) for a real-world example with URI parsing.

## Assigning a driver to a device
In OpenBOS drivers are canonically associated with devices via the `"https://openbos.org/schema/bos#hasDriver">` predicate. You can add a driver reference to a device via the add cli command or the web-app. In the future, we may add the ability to also assisign drivers at the point level so that a logical device can use an abitrary combination of drivers.