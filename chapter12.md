Of course. Here is the comprehensive Chapter Takeaways document for Chapter 12: SD-WAN Configuration and Monitoring, following your specified structure and format.

# Chapter 12: SD-WAN Configuration and Monitoring

## 1. Chapter Summary

This chapter provides a detailed exploration of FortiGate's Secure SD-WAN capabilities, covering its fundamental components, configuration, routing behavior, and monitoring tools. It begins by defining Secure SD-WAN as a software-defined approach that provides dynamic, policy-based application path selection across multiple WAN connections while integrating FortiGate's native security features. The chapter outlines the primary use cases for SD-WAN, including Direct Internet Access (DIA) to steer traffic across various physical links, site-to-site traffic management using overlay tunnels (typically IPsec), and Remote Internet Access (RIA) where spoke traffic is backhauled to a central hub for inspection before breaking out to the internet. These use cases highlight SD-WAN's core benefits: effective WAN use, improved application performance, and cost reduction.

The chapter then breaks down the fundamental components of an SD-WAN configuration. It starts with **Members**, which are the physical or logical interfaces (underlays like internet/MPLS or overlays like IPsec/GRE) used to steer traffic. These members are grouped into logical **Zones** for simplified policy and rule management. A critical component is the **Performance SLA**, which monitors member health by measuring latency, jitter, and packet loss, either actively via probes or passively by observing live traffic. This health information is then used by **SD-WAN Rules** to make intelligent steering decisions. The rules themselves combine traffic matching criteria (such as source/destination, application, or Internet Service) with a steering strategy, like selecting the member with the best quality, lowest cost, or simply following a manual preference order. The chapter emphasizes that SD-WAN rules are evaluated top-down, and an implicit rule at the end handles any unmatched traffic by using standard routing.

Finally, the chapter covers the crucial aspects of routing and monitoring in an SD-WAN context. It explains that while SD-WAN rules steer traffic to a specific member, a valid route must still exist in the routing table for that member to be considered a viable path. This leads to a discussion of how static and dynamic routes interact with SD-WAN zones and members. The chapter concludes by showcasing the various monitoring tools available in the FortiGate GUI. These include the SD-WAN widget on the Network dashboard, the detailed Performance SLA page for analyzing link health over time, and the SD-WAN Zones page for a graphical overview of traffic distribution (by bandwidth, volume, or sessions). It also explains how to verify SD-WAN behavior by examining logs, specifically the Forward Traffic log, which can be customized to show which SD-WAN rule and member were used for a session, and the System Events log, which records important state changes like a member going down.

## 2. Chapter Takeaways

### **Module 1: SD-WAN Basics**

**What Is SD-WAN?:**
*   A software-defined approach to dynamically steer WAN traffic based on flexible, user-defined rules.
*   **Fortinet's implementation is Secure SD-WAN**, integrating built-in security features (IPsec, UTM, etc.) with WAN path control.
*   **Core functions:**
    *   Application-aware traffic matching.
    *   Dynamic link selection based on performance.
    *   Control of egress traffic.
*   **Benefits:**
    *   **Effective WAN use:** Leverages multiple link types (MPLS, Broadband, LTE) in a hybrid WAN model.
    *   **Improved application performance:** Steers critical applications over the best-performing path.
    *   **Cost reduction:** Prioritizes low-cost internet links over more expensive private links like MPLS.

**SD-WAN Use Cases:**
*   **Direct Internet Access (DIA) / Local Breakout:**
    *   Steering internet-bound traffic directly from a site across multiple physical internet links (underlays).
    *   Most common use case.
    *   Typically uses static default routes for routing.
*   **Site-to-Site Traffic:**
    *   Steering corporate traffic between sites (e.g., branch to HQ) using overlay tunnels (IPsec).
    *   Often in a hub-and-spoke topology.
    *   Typically uses dynamic routing (e.g., BGP) over the tunnels for scalability.
*   **Remote Internet Access (RIA) / Remote Breakout:**
    *   Internet traffic from spoke sites is backhauled through overlay tunnels to a central hub.
    *   The hub performs centralized security inspection before traffic breaks out to the internet.
    *   Useful for centralizing security policy and leveraging a high-end hub firewall.

---

### **Module 2: SD-WAN Fundamentals**

**SD-WAN Components:**
*   **Members:** The interfaces used to steer traffic.
    *   Can be physical interfaces (port1, port2), VLANs, LAGs.
    *   Can be logical interfaces like IPsec tunnels (overlays).
*   **Zones:** Logical groupings of members to optimize and simplify configuration.
    *   A policy can reference a zone instead of multiple individual members.
    *   A default zone named `virtual-wan-link` exists for any member not explicitly assigned to a custom zone.
*   **Performance SLAs:** Monitor the health and performance of members.
    *   Measures **Latency, Jitter, and Packet Loss**.
    *   Determines if a member is `alive` or `dead`.
    *   **Monitoring Methods:**
        *   **Active:** Sends periodic probes (e.g., ping, http) to configured servers.
        *   **Passive:** Measures performance based on live user traffic.
*   **SD-WAN Rules:** The core logic that combines traffic matching with a steering strategy.
    *   Defines where to steer traffic (member preference) and the criteria for a member to be considered acceptable (member performance).

**SD-WAN Rules - Overview:**
*   Evaluated in descending order (top-down), with the first match being applied.
*   If no user-defined rule matches, the **implicit rule** at the bottom is used.
*   The implicit rule follows the standard routing table (FIB) and load balances traffic (default: per source IP) across all available SD-WAN members.
*   **Key Function:** SD-WAN rules *steer* traffic; a separate firewall policy is still required to *allow* the traffic.

**SD-WAN Members and Zones:**
*   **Members (Links):**
    *   **Underlay:** Physical links provided by an ISP (e.g., cable, DSL, MPLS). Traffic is subject to ISP routing policies.
    *   **Overlay:** Virtual links built on top of underlays (e.g., IPsec, GRE). Provide flexible and secure routing.
*   **Zones:**
    *   Group members with similar characteristics (e.g., an "underlay" zone for all internet links, an "overlay" zone for all IPsec tunnels).
    *   Firewall policies must reference SD-WAN zones, not individual members.

**Performance SLAs - Configuration:**
*   **Link Health Monitor:** Defines how to monitor the link.
    *   **Detection Mode:** Active, Passive, or Prefer Passive.
    *   **Server(s):** Up to two servers (beacons) can be configured for active monitoring to ensure the server itself isn't the point of failure.
*   **SLA Targets (Optional):** Define the maximum acceptable Latency, Jitter, and Packet Loss for a member.
    *   Used by certain strategies (like Lowest Cost SLA) to determine if a link is eligible for steering traffic.
*   **Link Status (Probes):** Configure the check interval and the number of failed probes before a link is declared dead.

**SD-WAN Rules - Strategies:**
*   Defines how to select the best member(s) from the list of outgoing interfaces.
*   **Member Selection Strategies:**
    *   **Manual:** Strictly follows the configured interface preference order. Does not consider performance metrics.
    *   **Best Quality:** Chooses the member with the best performance based on a specified quality criterion (latency, jitter, packet loss, etc.).
    *   **Lowest Cost (SLA):** Chooses the member with the lowest cost (a manually assigned value) that is currently meeting the defined SLA targets. Cost and then configuration order are tie-breakers.
*   **Load Balancing:** Can be combined with strategies to distribute traffic across all preferred members (e.g., all members that meet the SLA target).

**SD-WAN Rules - Traffic Matching Criteria:**
*   Rules can match traffic based on a combination of:
    *   **Source:** IP address, user/group, interface (CLI only).
    *   **Destination:** IP address, protocol/port.
    *   **Internet Service:** ISDB objects or groups.
    *   **Application:** Application Control signatures or categories.
    *   **Type of Service (ToS):** DSCP values in the IP header.

**Rule Lookup Process and Routing:**
*   **Key Routing Principle #1:** An SD-WAN rule is skipped if the standard routing table's best route to the destination is NOT through an SD-WAN member. The lookup process starts with a FIB lookup.
*   **Key Routing Principle #2:** An SD-WAN rule is skipped if none of its configured members have a valid route to the destination.
*   **Process Flow:**
    1.  Packet arrives.
    2.  FIB lookup for destination. If best route is not via an SD-WAN member, skip to next rule.
    3.  If it is, check the rule's outgoing interface list (`oif list`).
    4.  Look for an acceptable member (alive and has a route to destination).
    5.  If found, steer traffic to that member and send for firewall policy lookup.
    6.  If not found, move to the next SD-WAN rule.
*   **Firewall Policies for SD-WAN:** Must be configured to allow traffic.
    *   The `Source` and/or `Destination` interface in the policy will be an SD-WAN zone (e.g., `underlay`, `overlay`).
    *   You cannot reference an individual SD-WAN member directly in a firewall policy.

**Policy Routes and SD-WAN:**
*   Regular Policy Routes have precedence over SD-WAN rules.
*   SD-WAN rules are essentially a more advanced, software-defined form of policy routing.
*   A static route must be configured to direct traffic towards the SD-WAN members.
    *   You can reference an SD-WAN zone in a static route, which automatically creates individual ECMP routes for each member in that zone.

---

### **Module 3: SD-WAN Basic Monitoring**

**Verifying SD-WAN Traffic Routing:**
*   **Forward Traffic Logs (`Log & Report > Forward Traffic`):**
    *   Enable the "SD-WAN Rule Name" and "SD-WAN Quality" columns.
    *   These columns show which rule was matched and which member was selected (and why, e.g., "SLA," "lowest cost").
    *   An empty "SD-WAN Rule Name" field indicates the implicit rule was used.
*   **Packet Sniffer (`diagnose sniffer packet ...`):** Use verbosity level 4 or higher to see the egress interface for captured packets.

**Policy Route Lookup (CLI):**
*   The command `diagnose firewall proute list` shows the system's policy routes, including those generated by SD-WAN rules.
*   **Key fields in output:**
    *   `vwl_service`: The ID and name of the matching SD-WAN rule.
    *   `vwl_mbr_seq`: The sequence ID of the member chosen to steer traffic.
    *   `oif`: The outgoing interface list, sorted by preference for that rule.

**SD-WAN Fields in Session List (CLI):**
*   The command `diagnose sys session list` shows details for active sessions.
*   **Key fields in output:**
    *   `policy_id`: The firewall policy that allowed the session.
    *   `sdwan_mbr_seq` and `sdwan_service_id`: The ID of the SD-WAN member and rule used for the session.
    *   These fields will not appear if the implicit rule was used (standard FIB routing).

**Monitoring Tools in GUI:**
*   **Dashboard (`Dashboard > Network`):** The SD-WAN widget provides a high-level overview of member status and performance. Can be expanded for more detail.
*   **SD-WAN Zones Page (`Network > SD-WAN > SD-WAN Zones`):**
    *   Shows a synthetic view of all zones and their members.
    *   Provides graphical charts for traffic distribution by Bandwidth, Volume, or Sessions.
*   **SD-WAN Rules Page (`Network > SD-WAN > SD-WAN Rules`):**
    *   Summary view of all rules, their configuration, and which members are currently in use.
*   **Performance SLA Page (`Network > SD-WAN > Performance SLAs`):**
    *   The primary tool for monitoring link health.
    *   Displays graphs for Packet Loss, Latency, and Jitter over the past 10 minutes.
    *   Shows member status (alive/dead) and highlights any SLA target violations.
*   **System Event Logs (`Log & Report > System Events`):**
    *   The SD-WAN Events log summary shows important events like SLA status changes and member preference order changes.

---
