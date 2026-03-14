# Web App Overview

The BOS web app (SPOG) is the primary operations interface for an OpenBOS deployment. It runs on port 5001 and provides a unified UI for managing the system model, running and scheduling apps, and administering the platform — without requiring any command-line knowledge.

---

## System Model Explorer

The Explorer lets you browse and edit the semantic model of your building — the graph of spaces, equipment, and control points that everything else in BOS builds on.

- **Graph view** — interactive Cytoscape visualization of the full RDF graph. Click any node to open an inspector showing its 1-hop neighborhood, semantic type, and live value (for points).
- **List views** — filterable card grids for Locations, Equipment, and Points. Filter by name, URI, or Brick class. Multi-select for bulk operations.
- **CRUD** — create spaces, equipment, and points from guided forms with Brick class pickers. Delete or update any entity inline.
- **Live read/write** — read and write point values directly from the inspector panel, backed by Device Control.
- **Drag to terminal** — drag any entity card onto the embedded terminal to paste its BOS URI at the cursor.

---

## App Orchestrator

The Orchestrator is the control panel for containerized BOS apps.

- **Active apps** — table of currently running containers with status, uptime, and transaction ID. Stop any app from the UI.
- **Scheduled apps** — cron-based scheduling. Enable/disable schedules without removing them. See next and last run times.
- **Event handlers** — register apps to fire when a specific topic is published to the EventBus. See last-triggered timestamps and status.
- **Run history** — completed and failed runs with timestamps and exit status.
- **Launch wizard** — drag an app from the library sidebar onto any view to open a wizard that generates the environment variable form from the app's `env_spec`. Control-point fields get Brick class pickers for semantic point selection.
- **Auto-refresh** — configurable polling interval (5 s to 5 min) with a live countdown indicator.
- **Live event stream** — real-time feed of all EventBus messages streamed over WebSocket.

---

## App Library

A browser for the Docker images installed on the system.

- View all available BOS apps with their description, author, and configuration schema.
- Delete images with a confirmation gate (requires typing the image name).
- Link to the App Builder to create new apps.

---

## App Builder

An in-browser tool for authoring and building BOS apps without leaving the UI.

- Write Python source code with a code editor.
- Define the app's metadata and `env_spec` — the configuration schema that drives the launch wizard.
- Build a Docker image in-browser and stream the build log in real time over WebSocket.
- The generated image is immediately available in the library and orchestrator.

---

## Settings

System administration and service configuration.

- **Default space** — set the global space context used across the UI for filtering and suggestions. Persists in the browser.
- **Service addresses** — reference panel of all core service ports (Sysmod, DevCtrl, History, Orchestrator, EventBus).
- **Maintenance actions** — flush the DevCtrl value cache, apply sample rate changes to the historian, refresh the historian's point name table.
- **External tools** — resolved URLs for Grafana, Jupyter, Redis UI, and Oxigraph.

---

## Embedded Terminal

A persistent terminal panel (toggle with `` Ctrl+` ``) that connects to the `cli` Docker container over WebSocket.

- Full PTY with resize support — runs a real bash session.
- Drag BOS entity URIs from the Explorer directly into the terminal cursor.
- Available on every page — no context switching needed.

---

## External Tool Integration

The nav drawer provides quick links to the other tools in the BOS ecosystem:

| Tool | Purpose |
|------|---------|
| **Grafana** | Time-series dashboards backed by the BOS historian |
| **Jupyter** | Interactive notebooks for data exploration and prototyping |
| **Redis UI** | Inspect shared memory key-value store |
| **Oxigraph** | SPARQL query interface for the system model RDF graph |
| **Documentation** | This site |

## Further Readings
Coming soon...
