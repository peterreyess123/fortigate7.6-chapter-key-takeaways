# Chapter 3: Routing

## 1. Chapter Summary

This chapter focuses on the IP routing capabilities of FortiGate, a fundamental aspect of how it directs network traffic. It begins by defining IP routing as the process FortiGate uses in NAT mode to forward packets between different IP networks, supporting both IPv4 and IPv6. The chapter clarifies that routing decisions are made for both firewall traffic (user traffic transiting the FortiGate) and local-out traffic (traffic originating from the FortiGate itself, like DNS queries or FortiGuard updates). A key concept introduced is that routing generally precedes most security actions, meaning security policies are configured based on routing decisions, not the other way around. The distinction between the Routing Information Base (RIB), which is the standard routing table visible in the GUI and CLI containing active routes, and the Forwarding Information Base (FIB), the kernel's routing table used for actual lookups, is explained.

The chapter then details the route lookup process within FortiOS, outlining the order of precedence: Policy Routes, ISDB Routes, SD-WAN Rules, and finally the FIB (standard routing table). If no match is found in any of these, the packet is typically dropped. Static routes, configured manually by an administrator, are presented as a primary method for directing traffic, including the essential default route (0.0.0.0/0) for internet-bound traffic. The ability to use named address objects (Subnet or FQDN type) and Internet Service Database (ISDB) objects as destinations in static routes is also covered, enhancing flexibility. The chapter provides guidance on interpreting the routing table, focusing on key attributes like network, gateway IP, interface, administrative distance, metric, and priority, and explains how these attributes are used in the best route selection process. Administrative distance is highlighted as the primary tie-breaker for duplicate routes from different routing protocols or static configurations, while metric serves as a tie-breaker for routes from the same dynamic routing protocol, and priority for static routes with the same distance.

Finally, the chapter discusses Equal Cost Multi-Path (ECMP) routing, where FortiGate installs multiple same-protocol routes with identical destination, distance, metric, and priority into the RIB and load balances traffic across them. Different ECMP load balancing algorithms are introduced, such as source IP (default), source-destination IP, weighted (for static routes), and usage/spillover. Configuration options for these algorithms, including setting weights on interfaces or static routes and configuring spillover thresholds via the CLI, are outlined. The chapter concludes by explaining the Reverse Path Forwarding (RPF) check, an anti-spoofing mechanism that verifies if a valid return path exists for the source IP on the packet's ingress interface, with options for strict or feasible path validation.

## 2. Chapter Takeaways

### **Module 1: Routing on FortiGate**

**What Is IP Routing?:**
*   FortiGate acts as an IP router in Network Address Translation (NAT) mode (default).
*   Forwards packets between different IP networks, supporting IPv4 and IPv6.
*   **Performed for:**
    *   **Firewall traffic:** User traffic passing through the FortiGate.
    *   **Local-out traffic:** Traffic originating from the FortiGate itself (e.g., DNS queries, FortiGuard updates, pings from FortiGate).
*   Determines the next hop (outgoing interface and gateway IP) for a packet based on its destination IP address.

**Routing Table & Best Route Selection:**
*   **Routing Table (RIB):** Contains active routes (connected, static, dynamic) with next-hop information.
*   **Best Route Selection Process:**
    1.  **Longest Prefix Match:** The most specific route to the destination is preferred.
    2.  **Administrative Distance (AD):** If multiple routes to the same destination exist from different sources (e.g., static vs. OSPF), the route with the lower AD is chosen.
    3.  **Metric:** For routes from the *same* dynamic routing protocol with the same AD, the one with the lower metric is preferred.
    4.  **Priority:** For static routes with the same AD, the one with the lower priority value is preferred. (Default priority is 0 for static routes in CLI, 1 in some GUI contexts, and it is actually used for tie-breaking static ECMP routes or influencing ADVPN shortcut selection).
*   **Duplicate Routes:** Multiple routes to the same destination prefix.
    *   If different AD, the lowest AD wins (others may become standby routes, not in RIB but in routing database).
    *   If same AD and protocol, metric decides.
    *   If same AD and static, priority decides.
    *   If all are equal (AD, metric, priority, protocol), ECMP may occur.
*   **Routing Precedes Security:** Configure security policies based on routing decisions, not vice-versa.

**Route Lookup Process in FortiOS:**
*   FortiGate performs route lookups for the first packet of a session and the first reply packet. Routing information is then cached in the session table.
*   **Order of Lookup:**
    1.  **Policy Routes:** Manually configured routes that match based on more granular criteria (source IP, protocol, port, etc.).
    2.  **ISDB Routes:** Routes using Internet Service Database objects as destinations.
    3.  **SD-WAN Rules:** Rules for steering traffic across SD-WAN members (covered in Chapter 12).
    4.  **FIB (Forwarding Information Base / Standard Routing Table):** Contains best connected, static, and dynamic routes.
*   If a session's route changes in the routing table, the session information is flushed, and new lookups are performed.

**RIB (Routing Information Base) and FIB (Forwarding Information Base):**
*   **RIB:** Standard routing table containing active (best) connected, static, and dynamic routes. Visible in GUI (`Dashboard > Network > Static & Dynamic Routing`) and CLI (`get router info routing-table all`).
*   **FIB:** Kernel's routing table, used for actual packet forwarding lookups. Composed mostly of RIB entries plus system-specific entries. Visible via CLI only (`get router info kernel`).
    *   This chapter primarily focuses on the RIB.

**Static Routes:**
*   Manually configured by an administrator.
*   Define a destination network, a gateway IP address (next hop), and an outgoing interface.
*   **Default Route:** A common static route with destination `0.0.0.0/0.0.0.0` (or `::/0` for IPv6) used to forward traffic for which no more specific route exists (typically internet traffic).
*   **Named Addresses for Destination:** Firewall address objects of type Subnet or FQDN (with "Routing configuration" enabled in the address object) can be used as the destination for static routes.
*   **Internet Services Routing:** ISDB objects can be used as the destination in static routes, effectively creating policy-based routing for those services. These are treated as policy routes.

**Routing Monitor (GUI):**
*   Located under `Dashboard > Network > Static & Dynamic Routing`.
*   Displays the RIB (best active routes: Connected, Static, Dynamic).
*   **Does NOT show:** Inactive routes, standby routes, or policy routes (ISDB routes, SD-WAN rules, regular policy routes are in a separate policy route table).
*   **Route Attributes Displayed:** Network, Gateway IP, Interface, Distance, Metric (disabled by default, enable via column settings), Priority (disabled by default), Route Type.

**Route Attributes Detailed:**
*   **Network:** Destination IP address and subnet mask.
*   **Gateway IP:** Next-hop router's IP address. (0.0.0.0 for connected routes or if interface is point-to-point).
*   **Interface:** Outgoing FortiGate interface.
*   **Administrative Distance (AD):** Trustworthiness of the route source. Lower is better.
    *   Connected: 0
    *   Static: 10 (default, configurable)
    *   eBGP: 20
    *   OSPF: 110
    *   RIP: 120
    *   iBGP: 200
*   **Metric:** Used by dynamic routing protocols to determine the best path when multiple routes from the same protocol exist for the same destination. Lower is better. Calculation varies by protocol (e.g., hop count for RIP, cost for OSPF).
*   **Priority:** Used for static routes as a tie-breaker when multiple static routes to the same destination have the same AD. Lower is better. Default is 0 if set via CLI, configurable.
    *   Note: The slide shows a default weight of 1 for static routes; this is actually **priority** which is used for static route selection or ADVPN. Weight is used for ECMP load balancing. Clarification on the updated slide: "Priority set to 10 (default value is 1)" is likely a typo and refers to administrative distance or priority, not weight. The weight is a different parameter for load balancing. The change log slide for Chapter 3 mentions "Clarification on weight description and image updated".
*   **Route Type (Source):** Indicates how the route was learned (Static, Connected, BGP, OSPF, RIP, etc.).

**GUI Route Lookup Tool:**
*   Available in the Routing Monitor widget.
*   **Required input:** Destination IP address.
*   **Optional input:** Destination port, source IP/port, protocol, source interface.
*   **Behavior:**
    *   If only destination IP (and optionally protocol/port) is provided, it checks the RIB and highlights the matching route.
    *   If all criteria (including source info) are provided, it checks both policy routes and the RIB. If a policy route matches, GUI redirects to the policy route page.

**Reverse Path Forwarding (RPF) Check:**
*   An anti-spoofing protection mechanism.
*   Verifies if the routing table has a return path to the source IP address through the same interface the packet arrived on.
*   **Modes (CLI configurable):**
    *   `config system settings set strict-src-check [disable|enable]`
        *   **Feasible Path (Default, formerly `loose`, `strict-src-check disable`):** A route to the source exists via the ingress interface (not necessarily the best route).
        *   **Strict Path (`strict-src-check enable`):** The *best* route to the source must be via the ingress interface.
    *   Can also be disabled on a per-interface basis (`config system interface edit <name> set src-check disable end`).
*   RPF check is performed only on the first packet of a new session.
*   If RPF check fails, debug flow shows "reverse path check fail, drop".

---

### **Module 2: ECMP Routing**

**Equal Cost Multi-Path (ECMP):**
*   Occurs when two or more routes of the **same protocol type** have:
    *   Equal Destination subnet
    *   Equal Administrative Distance
    *   Equal Metric (for dynamic routes)
    *   Equal Priority (for static routes)
*   All qualifying ECMP routes are installed in the RIB (and thus FIB).
*   Traffic is load-balanced across these ECMP routes.

**ECMP Load Balancing Algorithms:**
*   Determines how sessions are distributed across ECMP routes.
*   **Configurable via CLI (`config system settings set v4-ecmp-mode <mode> end` if SD-WAN is disabled):**
    *   **Source IP (Default):** Sessions from the same source IP use the same route.
    *   **Source-Destination IP:** Sessions with the same source and destination IP pair use the same route.
    *   **Weighted:** (Applies to static ECMP routes only). Sessions are distributed based on configured route weights or interface weights. Higher weight gets more traffic. Interface weight takes precedence.
    *   **Usage (Spillover):** One route is used until its bandwidth threshold (configured on the interface) is reached, then the next route is used.
*   If SD-WAN is enabled, ECMP behavior is controlled by `config system sdwan set load-balance-mode <mode>`.

**Configuring ECMP:**
*   **Algorithm Selection:** As above, via CLI.
*   **Weighted Algorithm Configuration:**
    *   Interface level: `config system interface edit <name> set weight <0-255> end`
    *   Static route level: `config router static edit <id> set weight <0-255> end`
    *   Interface weight has precedence over static route weight if both are configured for a path.
*   **Spillover Thresholds (for Usage algorithm):**
    *   Configured on interfaces: `config system interface edit <name> set spillover-threshold <kbps> set ingress-spillover-threshold <kbps> end`
    *   Thresholds are 0 by default (disables spillover check).

**Default ECMP Algorithm vs. SD-WAN ECMP Algorithm:**
*   When SD-WAN is disabled, `v4-ecmp-mode` controls ECMP.
*   When SD-WAN is enabled, `load-balance-mode` under `config system sdwan` takes over.
*   **Key difference:** SD-WAN's `load-balance-mode` supports a **Volume** algorithm.
    *   **Volume Algorithm:** FortiGate tracks cumulative bytes per member and distributes sessions based on member weight. Higher weight = higher target volume = more traffic.
    *   Weight and spillover thresholds for SD-WAN are defined in the SD-WAN member configuration, not on static routes/interfaces directly for this purpose.

---
