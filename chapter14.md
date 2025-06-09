Of course. Here is the comprehensive Chapter Takeaways document for Chapter 14: High Availability (HA), following your specified structure and format.

# Chapter 14: High Availability (HA)

## 1. Chapter Summary

This chapter provides an in-depth guide to FortiGate's High Availability (HA) solution, which uses the FortiGate Clustering Protocol (FGCP) to link two or more devices into a single, resilient cluster. The primary goals of HA are to provide redundancy against device or link failures and, in some configurations, to increase performance. The chapter begins by introducing the two main HA operation modes: Active-Passive, where only one primary device actively processes all traffic while secondary devices remain on standby, and Active-Active, where the primary device can load-balance and distribute sessions to all members of the cluster, allowing them all to process traffic concurrently. The fundamental requirements for forming a cluster are detailed, emphasizing that all members must have the same hardware model, firmware version, licensing level, and operating mode to ensure stability and proper function.

The document then dives into the mechanics of cluster operation, starting with the crucial primary device election process. It explains the two election methodologies: the default behavior (override disabled), where uptime is a key factor, and the override-enabled behavior, where a manually set priority value takes precedence. This section clarifies how to interpret and influence the election to control which device becomes primary. The chapter thoroughly explains the synchronization process, detailing what information is shared between cluster members over the dedicated heartbeat interfaces. This includes the complete configuration (on initial join), incremental configuration changes, and operational runtime data such as session tables, ARP tables, and IPsec SAs. It also explicitly lists which device-specific settings, like the host name and HA priority, are *not* synchronized.

Finally, the chapter covers HA failover mechanisms and ongoing monitoring. It outlines the different types of failovers, including device failover (triggered by loss of heartbeat hello packets), link failover (triggered by a monitored interface going down), and more advanced failovers based on memory usage or SSD health. The role of virtual MAC addresses in enabling a seamless transition during a failover is explained, highlighting how the new primary adopts the same virtual MACs and uses gratuitous ARP to update the network. The chapter concludes with guidance on how to monitor the HA cluster's health and status through both the GUI and CLI, interpret synchronization checksums, manage individual members, and perform a zero-downtime firmware upgrade using the uninterruptible upgrade process.

## 2. Chapter Takeaways

### **Module 1: HA Operation Modes**

**What Is FortiGate HA?:**
*   Links two or more FortiGate devices into a cluster for redundancy and/or performance enhancement.
*   Uses **FGCP (FortiGate Clustering Protocol)** to discover members, elect a primary, and synchronize data.
*   **Cluster Roles:**
    *   **Primary:** The active device that manages the cluster, makes all processing decisions, and synchronizes its configuration and session information to secondaries.
    *   **Secondary (Standby):** Receives synchronization data from the primary and monitors its health, ready to take over if the primary fails.

**HA Operation Modes:**
*   **Active-Passive (A-P):**
    *   Only the primary FortiGate actively processes network traffic.
    *   All secondary devices are in a passive, standby state.
    *   Provides redundancy and failover.
*   **Active-Active (A-A):**
    *   All FortiGate devices in the cluster can process traffic.
    *   The primary device acts as a load balancer, distributing sessions to secondary members.
    *   Provides both redundancy and performance scaling.

**HA Requirements:**
*   **All members must have the same:**
    *   **Model:** Same hardware or VM model.
    *   **Firmware Version:** Identical FortiOS version.
    *   **Licensing:** All subscription licenses (e.g., FortiGuard) must be the same. The cluster operates at the lowest common licensing level.
    *   **Hard Drive Configuration:** Same number and size of drives/partitions.
    *   **Operating Mode:** The management VDOM must be in the same mode (NAT or Transparent).
*   **Configuration Setup:**
    *   All members must have the same HA Group ID, Group Name, and Password.
    *   Best practice is to use at least two heartbeat interfaces for redundancy.
*   **DHCP/PPPoE Interfaces:** Initially configure these interfaces with static IPs to prevent issues during cluster formation, then revert if necessary.

**Primary FortiGate Election:**
*   The process by which the cluster elects one device to be the primary.
*   **Override Disabled (Default Behavior):** The election criteria are checked in this order, and the process stops at the first tie-breaker.
    1.  **Highest number of monitored interfaces that are up.**
    2.  **Highest HA uptime** (by at least 5 minutes).
    3.  **Highest Priority** value (a number between 0-255, default is 128).
    4.  **Highest Serial Number.**
*   **Override Enabled (`config system ha > set override enable`):** Priority is considered before uptime, making it easier to designate a preferred primary.
    1.  **Highest number of monitored interfaces that are up.**
    2.  **Highest Priority** value.
    3.  **Highest HA uptime.**
    4.  **Highest Serial Number.**
*   **Forcing a Failover:**
    *   With override disabled, reset the primary's uptime: `diagnose sys ha reset-uptime`.
    *   With override enabled, increase the priority of a secondary device.

---

### **Module 2: HA Cluster Synchronization**

**Primary FortiGate Tasks:**
*   Broadcasts hello packets for member discovery and monitoring.
*   Synchronizes operation-related data to all secondary members.
*   In Active-Active mode, distributes sessions to secondary members.

**Secondary FortiGate Tasks:**
*   Broadcasts hello packets to announce its presence.
*   Receives and applies synchronized data from the primary.
*   Monitors the health of the primary.
*   In Active-Active mode, processes traffic distributed by the primary.

**Heartbeat Interface IP Addresses:**
*   Automatically assigned by FGCP based on the device's serial number.
*   The device with the highest serial number gets `169.254.0.1`, the next highest gets `169.254.0.2`, and so on.
*   These IPs are fixed to the device regardless of its primary/secondary role and are used exclusively for HA communication (non-routable).
*   The IP assignment only changes if a member joins or leaves the cluster.

**Heartbeat and Monitored Interfaces:**
*   **Heartbeat Interfaces:**
    *   Used to exchange sensitive HA information (hello packets, synchronization data).
    *   Must be physical ports. Best practice is to use at least two for redundancy and connect them via a dedicated switch or VLAN.
*   **Monitored Interfaces (Port Monitoring):**
    *   Interfaces whose link status is monitored for link failover.
    *   If a monitored interface on the primary goes down, it triggers a failover.
    *   Typically critical LAN or WAN interfaces. Do not monitor heartbeat interfaces.

**Configuration Synchronization:**
*   **Complete Synchronization:** Occurs when a new member joins the cluster and its configuration checksum does not match the primary's. The primary sends its entire configuration to the new member.
*   **Incremental Synchronization:** After the initial sync, only configuration changes are sent between members. This applies to both the primary and secondaries (if changes are made on a secondary).
*   **What IS Synchronized:**
    *   Most of the configuration (policies, objects, etc.).
    *   Runtime data: DHCP leases, ARP table, IPsec SAs, FortiGuard definitions.
    *   Firewall sessions (if `session-pickup` is enabled).
*   **What IS NOT Synchronized:**
    *   Device-specific settings: Host name, HA priority, HA override setting.
    *   HA management interface settings and its default route.
    *   System time (NTP should be used).
    *   Licenses (except FortiToken serials).
    *   GUI dashboard widgets.

**Session Synchronization:**
*   `config system ha > set session-pickup enable`
*   Provides seamless failover for active sessions, preventing users from having to re-establish connections.
*   **Synchronized by default:** Non-proxy TCP sessions (e.g., standard web browsing).
*   **Optional (`set session-pickup-connectionless enable`):** UDP and ICMP sessions.
*   **Not synchronized:** Local sessions (admin access, BGP/OSPF peerings), proxy-based sessions (except SIP ALG), and multicast sessions (only multicast routes are synced).
*   IPsec SAs are always synced; session-pickup for TCP is required for seamless failover of traffic within the tunnel. SSL VPN sessions are not synchronized.

---

### **Module 3: HA Failover**

**Failover Protection Types:**
*   **Device Failover:** A secondary device stops receiving hello packets from the primary, assumes the primary has failed, and initiates a new election.
*   **Link Failover (Port Monitoring):** A monitored interface on the primary device goes down, causing its "monitored ports" count to decrease and triggering a failover.
*   **Remote Link Failover (Link Health Monitor):** Uses link health monitoring (ping servers) to detect the failure of a remote link. A penalty (`ha-priority`) is applied, and if the total penalty exceeds the threshold, a failover occurs.
*   **Memory-Based Failover:** Triggers a failover if memory usage on the primary exceeds a configured threshold for a specific duration.
*   **SSD Failover:** Triggers a failover if FortiOS detects critical filesystem errors on an SSD in the primary device.

**Failover Protection Configuration:**
*   **Device Failover Timers:**
    *   `hb-interval`: Time between heartbeat packets (default 2, representing 200ms or 20ms depending on `hb-interval-in-milliseconds`).
    *   `hb-lost-threshold`: Number of missed heartbeats before a device is considered dead (default 6).
    *   Total Failover Time = `hb-interval` * `hb-lost-threshold`.
*   **Link Failover:** Configured by specifying interfaces in `config system ha > set monitor <interface1> <interface2> ...`
*   **Remote Link Failover:** Requires configuring a `system link-monitor` entry and setting the `pingserver-failover-threshold` under `config system ha`.

**Virtual MAC Addresses and Failover:**
*   To ensure smooth traffic transition, the primary device uses virtual MAC addresses on its interfaces.
*   The virtual MAC is generated based on the HA group ID and the interface index.
*   Heartbeat interfaces do *not* use virtual MACs.
*   **Upon failover:**
    1.  The newly elected primary takes over the *exact same* virtual MAC addresses as the former primary.
    2.  The new primary sends **gratuitous ARP** packets to the connected switches, announcing that the virtual MAC addresses are now reachable through its physical ports.
    3.  This updates the switches' MAC address tables, redirecting traffic to the new primary device without clients needing to re-ARP.

**Full Mesh HA:**
*   An advanced topology designed to eliminate single points of failure, including switches.
*   Each FortiGate device connects to two or more redundant upstream and downstream switches.
*   Requires the use of redundant interfaces or Link Aggregation (LAG) interfaces on the FortiGate.
*   If using LAG, the switches must support a multi-chassis link aggregation (MCLAG) protocol. FortiSwitch supports this.

---

### **Module 4: Monitoring**

**Checking HA Status:**
*   **GUI:**
    *   `System > HA`: Shows a table with all cluster members, their role (Primary/Secondary), host name, serial number, status (in-sync/out-of-sync), and other stats. The **Checksum** column can be enabled to verify synchronization.
    *   `Dashboard > Status > HA Status` widget: Provides a quick summary of the cluster's state.
*   **CLI (`get system ha status`):**
    *   Provides the most comprehensive status information.
    *   Shows cluster status, member models/roles, uptime, primary election reason, configuration checksums for each member, and performance stats (CPU/memory/sessions).
*   **CLI (`diagnose sys ha checksum ...`):**
    *   `show`: Displays checksum for the local member.
    *   `cluster`: Polls all members and displays their checksums. Essential for verifying if the cluster is in sync.
    *   `recalculate`: Forces a recalculation of checksums if you suspect a false out-of-sync state.

**HA Management Interface:**
*   Provides a way to access each cluster member directly, regardless of its primary/secondary role.
*   **Reserved HA Management Interface (Out-of-band):**
    *   A dedicated physical interface is used for management.
    *   Has its own separate routing table.
    *   Each member has a unique IP on this interface.
*   **In-band HA Management Interface:**
    *   Assigns a unique management IP to an existing user-traffic interface.
    *   Shares the main routing table.

**Firmware Upgrade in HA:**
*   **Uninterruptible Upgrade (Default):** Provides a zero-downtime upgrade process.
    1.  Firmware is uploaded to the primary.
    2.  Primary pushes the firmware image to all secondaries.
    3.  Secondaries upgrade and reboot one by one.
    4.  The first secondary to finish its upgrade becomes the new primary.
    5.  The former primary becomes a secondary and begins its own upgrade.
*   **Simultaneous Upgrade:** Upgrades all members at the same time. Causes a service outage but is faster.
*   It is best practice to always perform upgrades via the primary device's GUI or CLI.

---
