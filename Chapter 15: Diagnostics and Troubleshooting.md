Certainly. Here is the comprehensive Chapter Takeaways document for Chapter 15: Diagnostics and Troubleshooting, in the requested markdown format.

# Chapter 15: Diagnostics and Troubleshooting

## 1. Chapter Summary

This chapter equips administrators with the essential tools and methodologies for diagnosing and troubleshooting common issues on a FortiGate device. It begins by establishing the foundational principle of troubleshooting: an administrator must first know what constitutes "normal" behavior for their network. This involves establishing a baseline for key performance indicators such as CPU and memory usage, traffic volume and patterns, and having up-to-date physical and logical network diagrams. The chapter highlights the importance of using FortiGate's built-in monitoring tools, like the Security Fabric views, dashboard widgets, and logging, to collect this baseline data before a problem occurs, making it significantly easier to identify abnormal behavior when it arises. It then covers initial diagnostic steps at the physical and network layers, using fundamental CLI commands like `get system status` for general device health and `execute ping` and `execute traceroute` to test basic connectivity.

The chapter then progresses to more advanced, packet-level diagnostics. It provides a detailed look at two of the most powerful troubleshooting tools: the packet sniffer and the debug flow. The packet sniffer (`diagnose sniffer packet`) is presented as a tool for capturing raw packets on an interface, allowing an administrator to see exactly what traffic is arriving at or leaving the FortiGate. The debug flow (`diagnose debug flow`), on the other hand, is explained as a step-by-step trace of how the FortiGate's CPU processes a specific packet, showing every major decision point, including routing lookups, firewall policy matching, NAT application, and session creation. The chapter provides clear examples of how to interpret the output for both a new session (SYN packet) and a reply packet (SYN/ACK), demonstrating its utility in pinpointing exactly where a packet is being dropped or why it's being handled in an unexpected way. The GUI-based Packet Capture and Debug Flow tools are also presented as more user-friendly alternatives to the CLI commands.

Finally, the chapter addresses resource-related problems, focusing on high CPU and memory usage. It introduces the process monitor (in the GUI) and the `diagnose sys top` CLI command as primary tools for identifying which specific processes are consuming the most resources. This allows an administrator to correlate high usage with specific features or traffic. The chapter concludes with a detailed explanation of FortiGate's memory conserve mode, a self-protection mechanism that activates when memory usage becomes critically high. It describes the three configurable thresholds (green, red, and extreme) that govern this mode and the specific actions FortiGate takes to preserve memory, such as blocking new sessions, skipping quarantine actions, and modifying the behavior of flow-based and proxy-based inspection engines through the `fail-open` and `av-failopen` settings.

## 2. Chapter Takeaways

### **Module 1: General Diagnosis**

**Before a Problem Occurs:**
*   **Know what normal is (Baseline):** The most critical step in troubleshooting is to understand the network's normal operational state.
*   **Baseline Components:**
    *   CPU and Memory usage patterns.
    *   Traffic volume, directions, and distribution.
    *   Protocols, port numbers, and application mix.
    *   Network topology (physical and logical).
*   **Why?:** Abnormal behavior (e.g., a traffic spike) is difficult to identify and diagnose without a baseline for comparison.
*   **Tools for Baselining:** Security Fabric, Dashboard widgets, SNMP, Logging (FortiAnalyzer/syslog), CLI commands.

**Network Diagrams:**
*   Essential for analyzing and explaining network behavior, especially to technical support.
*   **Physical Diagram:** Shows Layer 1/2 connectivity (cables, ports, physical devices).
*   **Logical Diagram:** Shows Layer 3 relationships (IP subnets, VLANs, routers, logical devices).
*   The Security Fabric topology views on FortiGate can automatically generate and display these diagrams for connected Fabric devices.

**Monitoring Traffic Flows and Resource Usage:**
*   Continuously collect data to understand performance ranges (typical minimum and maximum).
*   Use FortiGate's built-in tools to monitor interface bandwidth, session counts, and CPU/memory usage over time (hour, day, week).

**System Information:**
*   Use CLI commands to get a snapshot of the device's current state, even if the GUI is inaccessible.
*   **`get system status`:** Provides a wealth of general-purpose information, including:
    *   Model and serial number.
    *   Firmware version and build.
    *   FortiGuard license status.
    *   System time.
    *   Versions of AV, IPS, and other signature databases.

**Network Layer Troubleshooting:**
*   Used to diagnose connectivity issues beyond the physical/link layers.
*   **`execute ping <destination>`:** Tests basic Layer 3 reachability using ICMP echo requests.
    *   `execute ping-options` allows for customization (source interface, packet size, repeat count, etc.).
    *   IPv6 equivalent: `execute ping6`.
*   **`execute traceroute <destination>`:** Identifies the path (hops) a packet takes to reach its destination.
    *   IPv6 equivalent: `execute traceroute6`.
*   **Important Context:**
    *   Tests are most accurate when performed from the correct location (e.g., from an endpoint to test the path *through* the FortiGate).
    *   May need to specify a source IP if NAT is involved.

---

### **Module 2: Packet Sniffer and Debug Flow**

**Packet Sniffer:**
*   Captures raw network packets on a specified interface.
*   **CLI Command Syntax:**
    *   `#diagnose sniffer packet <interface> '<filter>' <verbose> <count> <tsformat>`
*   **Key Parameters:**
    *   **`<interface>`:** The interface to sniff on (e.g., `port1`, `wan1`, or `any` for all interfaces).
    *   **`<filter>`:** Berkeley Packet Filter (BPF) syntax to capture specific traffic (e.g., `'host 8.8.8.8 and icmp'`).
    *   **`<verbose>`:** Controls the level of detail in the output (1-6). Level 4+ shows the interface name.
    *   **`<count>`:** The number of packets to capture before stopping.
    *   **`<tsformat>`:** Timestamp format (`a` for absolute UTC time, `l` for local time).
*   If packets are dropped by the kernel during sniffing, the filter is too broad and needs to be made more specific.

**Packet Capture—GUI:**
*   A user-friendly alternative to the CLI sniffer, found under `Network > Diagnostics > Packet Capture`.
*   Allows for real-time viewing of captured packets.
*   Provides basic and advanced filtering options for host, port, and protocol.
*   The capture can be stopped and the output can be downloaded as a `.pcap` file for analysis in tools like Wireshark.

**Debug Flow:**
*   A powerful step-by-step trace showing how the FortiGate CPU processes a packet.
*   Shows the "life of a packet" inside the FortiGate. If a packet is dropped, the debug flow shows the reason.
*   **Multi-step CLI Process:**
    1.  `diagnose debug flow filter <filter>`: Define a filter to trace only specific packets. This is crucial to avoid performance impact.
    2.  `diagnose debug enable`: Turns on the debug output.
    3.  `diagnose debug flow trace start <count>`: Starts the trace for a specified number of packets.
    4.  `diagnose debug flow trace stop`: Stops the trace.
*   **Interpreting Output:** Look for key functions like `print_pkt_detail` (packet arrival), `init_ip_session_common` (new session allocation), `vf_ip_route_input_common` (routing lookup), `fw_forward_handler` (firewall policy match), and `__ip_session_run_tuple` (NAT application).

**Debug Flow—GUI:**
*   Found under `Network > Diagnostics > Debug Flow`.
*   Provides a real-time view of the debug flow trace.
*   Allows for basic and advanced filtering.
*   The final output can be viewed, filtered, and exported as a CSV file after stopping the trace.

**Life of a Packet (Review):**
*   The debug flow visualizes this process. Key steps include ingress, NP offloading checks, kernel routing, policy lookup, session creation, UTM inspection, NAT application, and egress. Understanding this path is key to interpreting the debug flow output.

---

### **Module 3: CPU and Memory**

**High CPU and Memory Troubleshooting—CLI:**
*   Used to diagnose performance issues like network slowness when connectivity is not the problem.
*   **`get system performance status`:** Provides a quick overview of current and historical CPU/memory usage.
*   **`diagnose sys top`:** The primary tool for identifying which processes are consuming the most resources.
    *   Provides a real-time, updating list of processes similar to `top` in Linux.
    *   **Common high-usage processes:** `ipsengine`, `scanunitd` (inspection), `fgfmd` (management), `newcli`, `httpsd` (admin access).
    *   **Interactive Commands:**
        *   `Shift + P`: Sort by CPU usage.
        *   `Shift + M`: Sort by Memory usage.

**High CPU and Memory Troubleshooting—GUI:**
*   **Dashboard Widgets:** CPU and Memory widgets show current usage.
*   **Process Monitor (`Dashboard > Status > Process Monitor`):**
    *   A GUI-based view of running processes and their resource consumption.
    *   Allows for filtering, sorting, and terminating processes.

**Memory Conserve Mode:**
*   A self-protection mechanism FortiOS enters when memory usage becomes critically high.
*   **Three configurable thresholds (as % of total RAM):**
    *   **Red (`memory-use-threshold-red` - default 88%):** FortiGate enters conserve mode.
    *   **Extreme (`memory-use-threshold-extreme` - default 95%):** FortiGate takes more drastic action (drops all new sessions).
    *   **Green (`memory-use-threshold-green` - default 82%):** FortiGate exits conserve mode.
*   **CLI to check status:** `diagnose hardware sysinfo conserve`.

**What Happens During Conserve Mode?:**
*   **System configuration changes are blocked.**
*   **Quarantine actions are skipped** (including sending files to FortiSandbox).
*   **Flow-based IPS engine behavior is altered:**
    *   `config ips global > set fail-open [enable|disable]`
    *   `enable`: Packets bypass IPS inspection.
    *   `disable` (default): New packets requiring IPS inspection are dropped.
*   **Proxy-based Antivirus behavior is altered:**
    *   `config system global > set av-failopen [off|pass|one-shot]`
    *   `pass` (default): New sessions pass through without AV scanning.
    *   `off`: New sessions are not passed.
*   If the extreme threshold is reached, all new sessions requiring inspection are blocked, regardless of fail-open settings.

---
