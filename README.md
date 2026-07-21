# Zabbix MikroTik Custom Dashboard Template

![Zabbix](https://img.shields.io/badge/Zabbix-7.4-red)
![SNMP](https://img.shields.io/badge/protocol-SNMPv2-blue)
![MikroTik](https://img.shields.io/badge/vendor-MikroTik-orange)
![RouterOS](https://img.shields.io/badge/OS-RouterOS%20v7-brightgreen)
![License](https://img.shields.io/badge/license-MIT-green)

A comprehensive, ready-to-use Zabbix 7.4 dashboard wrapper template designed for **MikroTik Routers** (tested with MikroTik hEX series / RouterOS v7).

This template unifies system hardware monitoring, RouterOS 7 BGP session states, and network interface metrics into a clean, multi-tab dashboard layout. It leverages Zabbix Low-Level Discovery (LLD) and dynamic Honeycomb widgets to visualize your network topology and BGP routing status out of the box.

---

## What this solves

Standard MikroTik SNMP templates collect a wealth of data, but raw metrics are often scattered across hundreds of items, host graphs, and discovery rules. Monitoring overall system health, BGP peer states, and interface statistics requires building custom dashboards from scratch.

This template solves that by providing a structured, 3-tab dashboard wrapper:

- **System Health:** Quick overview of hardware model, serial number, firmware version, uptime, CPU core utilization, disk space, and dual memory usage visualization (both relative utilization percentage and absolute bytes usage).
- **BGP Peer Monitoring:** Visual BGP session health using Zabbix Honeycomb widgets (`Session uptime` and `Session State`) alongside received prefix counts per peer.
- **Dynamic Interface Traffic:** Automatic multi-column graph prototypes displaying traffic (Bits received/sent) and interface errors/discards for every discovered interface (LAN, WAN, bridges, VLANs).

---

## Project status

This template is stable, fully functional, and tested against MikroTik hEX (RB750Gr3) running RouterOS v7 with BGP v7 over SNMP and external check monitoring scripts.

---

## Template Architecture & Dependencies

This template acts as a **Dashboard Wrapper / Aggregator Template**. It links to parent templates and arranges their metrics onto a centralized, responsive dashboard.

### Linked Parent Templates:
1. `MikroTik hEX by SNMP` (or equivalent standard MikroTik SNMP template providing `system.hw.*`, `vm.memory.*`, `hrStorageSize`, and interface LLD).
2. `RouterOS 7 BGP by external check` (or your custom BGP monitoring template providing item keys matching `*BGP*uptime*`, `*BGP*session state*`, and prefix count prototypes).

```text
                  +-----------------------------------+
                  |  MikroTik Custom Dashboard        |
                  |  (Dashboard Wrapper Template)     |
                  +-----------------+-----------------+
                                    |
            +-----------------------+-----------------------+
            |                                               |
            v                                               v
+-----------------------+                       +-----------------------+
| MikroTik hEX by SNMP  |                       | RouterOS 7 BGP by     |
| (Hardware & IF LLD)   |                       | external check        |
+-----------------------+                       +-----------------------+
```

---

## Current limitations

- Designed specifically for **Zabbix 7.4+** due to native **Honeycomb widget** and updated dashboard schema support.
- Requires parent templates (`MikroTik hEX by SNMP` and BGP monitoring template) to be present in Zabbix before import.
- BGP Honeycomb widgets rely on item naming conventions containing `*BGP*uptime*` and `*BGP*session state*`.
- Memory utilization graph uses standard SNMP OIDs (`vm.memory.total` and `vm.memory.used`); systems with non-standard SNMP storage indexes may require updating the graph item keys.

---

## Screenshots

![Overview dashboard](docs/main.png)
![BGP](docs/bgp.png)
![Network](docs/network.png)

---

## Features

### System & Hardware Overview
- Hardware Model (`system.hw.model`)
- Firmware Version (`system.hw.firmware`)
- Serial Number (`system.hw.serialnumber`)
- System Uptime (`system.hw.uptime[hrSystemUptime.0]`)
- CPU Core Utilization Graph Prototype (`{#SNMPINDEX}: CPU utilization`)
- CPU Temperature SVG Graph (`*Temperature*`)
- Disk Space Usage Graph Prototype (`Disk-{#SNMPINDEX}: Disk space usage`)
- **Dual Memory Graphs:**
  - `Mikrotik: Memory utilization` (% utilized)
  - `Memory usage` (Absolute Bytes: Total vs. Used)

### BGP Monitoring Tab
- **Session Uptime Honeycomb:** Visual status tiles showing peer uptime dynamically labeled with `{ITEM.NAME}`.
- **Session State Honeycomb:** Interactive state tiles with status color thresholds (Green = Established / 1, Red = Down / 0).
- **Received Prefix Count:** Dynamic graph prototypes for total prefixes received per BGP peer (`{#BGP_NAME}`).

### Interface Traffic Tab
- **Automated Network Traffic Graphs:** Automatically generated 3-column grid for all discovered interfaces.
- **Metrics Included:**
  - Inbound Traffic (`Bits received`) - Filled green region
  - Outbound Traffic (`Bits sent`) - Red line
  - Inbound / Outbound Errors and Discarded Packets

---

## Tested with

- Zabbix Server 7.4
- RouterOS 7.x
- MikroTik hEX (RB750Gr3)
- SNMPv2c
- Native Zabbix Honeycomb and Graph Prototype Widgets

---

## Repository structure

```text
zabbix-mikrotik-custom-dashboard/
├── README.md
├── templates/
│   └── zbx_export_templates.yaml
└── LICENSE
```

---

## Requirements

- Zabbix 7.4 or newer
- MikroTik router with RouterOS v7 and SNMP enabled
- Parent templates imported into Zabbix:
  - `MikroTik hEX by SNMP` (or `MikroTik by SNMP`)
  - `RouterOS 7 BGP by external check`

---

## Installation

### 1. Enable SNMP on MikroTik

Connect to your MikroTik router via WinBox or SSH and enable SNMP:

```routeros
/snmp set enabled=yes contact="Admin" location="Server Room"
/snmp community add name=public addresses=0.0.0.0/0
```

### 2. Verify Parent Templates

Ensure that your Zabbix server has the parent templates imported:
- `MikroTik hEX by SNMP`
- `RouterOS 7 BGP by external check`

> **Note:** If your base SNMP template has a different name (e.g., `MikroTik by SNMP`), you can edit the `templates` section in `zbx_export_templates.yaml` prior to import or adjust the template linkage in Zabbix.

### 3. Import the Dashboard Template

1. Navigate to **Data collection → Templates → Import**.
2. Select `templates/zbx_export_templates.yaml`.
3. Click **Import**.

### 4. Assign Template to Host

1. Go to **Data collection → Hosts**.
2. Select your MikroTik host.
3. Under **Templates**, add `MikroTik Custom Dashboard`.
4. Click **Update**.

---

## Dashboard Layout Details

### Page 1: Main (System Overview)

| Widget | Type | Width / Height | Description |
|---|---|---|---|
| Firmware Version | Item | 16 x 2 | Current RouterOS / Firmware version |
| Hardware Model | Item | 24 x 2 | Router model name |
| Serial Number | Item | 18 x 2 | Hardware serial number |
| Uptime | Item | 14 x 2 | System uptime in days/hours |
| CPU Temperature | SVG Graph | 18 x 4 | Hardware CPU temperature graph |
| CPU Utilization | Graph Prototype | 54 x 4 | Multi-core CPU utilization breakdown |
| Disk Space Usage | Graph Prototype | 27 x 5 | Storage usage per partition |
| Memory Utilization | Graph | 36 x 5 | Memory usage percentage (%) |
| Memory Usage | Graph | 36 x 5 | Memory total vs used in absolute bytes |

### Page 2: BGP

| Widget | Type | Width / Height | Description |
|---|---|---|---|
| Session Uptime | Honeycomb | 14 x 5 | BGP peer uptime tiles using `*BGP*uptime*` |
| Session State | Honeycomb | 15 x 5 | BGP peer state tiles (0=Down [Red], 1=Established [Green]) |
| Received Prefix Count | Graph Prototype | 43 x 5 | Graph of prefixes received per BGP peer |

### Page 3: Interfaces

| Widget | Type | Width / Height | Description |
|---|---|---|---|
| Network Traffic | Graph Prototype | 71 x 15 | Grid layout (3 rows) of all interface traffic and error graphs |

---

## Custom Graphs Included

### Memory Usage Graph
- **Name:** `Memory usage`
- **Y-Axis MIN:** Fixed (0)
- **Items:**
  - `vm.memory.total[hrStorageSize.Memory]` (Dashed Red Line)
  - `vm.memory.used[hrStorageUsed.Memory]` (Gradient Green Line)

---

## Troubleshooting

### Honeycomb widgets show "No data"
- Ensure item names in your BGP monitoring template match the wildcard filter:
  - Uptime filter: `*BGP*uptime*`
  - State filter: `*BGP*session state*`
- Check **Monitoring → Latest data** to confirm BGP items are active and receiving values.

### Template import error: "Cannot find template"
- Verify that `MikroTik hEX by SNMP` and `RouterOS 7 BGP by external check` exist in your Zabbix instance under the exact names specified in the YAML `templates` block.

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
