# LS-DV-Router

Implementations of two classic intra-domain routing algorithms — **Link-State (LS)** and **Distance-Vector (DV)** routing — built and tested against a custom Python network simulator.

> Built as part of a Network-Centric Computing course assignment (Spring 2026). Repository shared for portfolio purposes.

## Overview

Intra-domain routing algorithms let routers within a single autonomous system independently discover and maintain shortest/lowest-cost paths to every other node, adapting automatically to link failures, link additions, and cost changes. This project implements both major approaches to that problem:

- **Distance-Vector Routing** — each router maintains and exchanges its own vector of costs to every known destination with its neighbors, updating routes as better paths are learned (with count-to-infinity mitigation).
- **Link-State Routing** — each router floods its local link-state information to the entire network, builds a full topology map, and computes shortest paths using Dijkstra's algorithm.

Both routers run as independent simulated instances in separate threads, communicating only through defined packet-passing interfaces — no shared memory or direct state access.

## Features

- **Distance-Vector Router (`DVrouter.py`)**
  - Maintains a distance vector to all known destinations
  - Periodic and event-triggered updates to neighbors
  - Count-to-infinity handling (bounded "infinity" heuristic)
- **Link-State Router (`LSrouter.py`)**
  - Reliable flooding of link-state advertisements with sequence numbers to discard stale updates
  - Full topology reconstruction and shortest-path computation via Dijkstra
- **Dynamic Topology Handling** — both routers correctly respond to `handle_new_link`, `handle_remove_link`, and link cost changes at runtime
- **Heartbeat-Based Updates** — periodic routing broadcasts ensure convergence even without detected local changes

## Architecture

Each router implements a common interface driven by the simulator:

| Method | Purpose |
|---|---|
| `__init__(addr, heartbeat_time)` | Initialize router state |
| `handle_packet(packet)` | Process incoming traceroute or routing packets |
| `handle_new_link(port, endpoint, cost)` | React to a new/updated link |
| `handle_remove_link(port)` | React to a link going down |
| `handle_timer(time_ms)` | Trigger periodic routing updates |
| `debug_string()` | Return router state for debugging/visualization |

## Getting Started

### Requirements
- Python 3
- Tkinter (only required for the graphical visualizer)
- (Optional) Docker, for a consistent test environment

### Running with the Visualizer
```bash
python3 visualize_network.py <networkFile.json> [DV|LS]
```

### Running Headless
```bash
python3 network.py <networkFile.json> [DV|LS]
```

### Running the Full Test Suite
```bash
python3 runAll.py
```

### Running via Docker
```bash
docker compose run --rm netcen-spring-2026
```

## Test Networks

Network topologies and link-change events are defined in JSON files (`test1.json`–`test5.json`), covering:
- Small, medium, and large network topologies
- Static topologies (no link changes) for baseline correctness
- Dynamic topologies with scheduled link additions/removals to test convergence and failure recovery

## Notes

- Routing packets are exchanged strictly through `self.send(port, packet)` — no direct access to other routers' internal state.
- Link-state flooding relies on sequence numbers rather than acknowledgments/retransmissions (single-hop delivery is assumed reliable).
- Distance-vector routing uses a bounded "infinity" heuristic (`infinity = 16`) to avoid the count-to-infinity problem.

## License

MIT
