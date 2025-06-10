# Chapter 9: Intrusion Prevention and Application Control

## 1. Chapter Summary

This chapter covers two critical FortiGate security features: Intrusion Prevention System (IPS) and Application Control, both of which leverage the FortiGate's IPS engine for advanced traffic analysis and threat mitigation. It begins by explaining that IPS detects and prevents network attacks by utilizing signatures, protocol decoders, and heuristics. IPS components include signature databases for known exploits and rate-based signatures for anomalies, protocol decoders to identify non-compliant traffic, and the core IPS engine responsible for inspection. The chapter details how to configure IPS sensors, which are collections of signatures and filters, by adding individual signatures or groups of signatures using filters based on attributes like target, severity, protocol, and OS. Rate-based signatures, which block traffic exceeding a threshold over time, and the ability to exempt specific IPs from signature actions for false positives are also discussed. The importance of policy logging for IPS events and the process of applying IPS sensors to firewall policies, often in conjunction with deep SSL inspection for encrypted traffic, is emphasized. Troubleshooting IPS, including high CPU usage by IPS engines and managing IPS fail-open behavior, is also covered.

The second part of the chapter focuses on Application Control, which also uses the flow-based IPS engine to detect and act on network application traffic, even if it uses non-standard ports or protocols. This makes it particularly effective for P2P applications. A key aspect is the hierarchical organization of application control signatures, where parent signatures (e.g., "Social.Media") take precedence over child signatures (e.g., "Facebook.Chat"). The chapter explains how to configure application control sensors (profiles) by selecting categories or individual application signatures and assigning actions like Allow, Monitor, Block, or Quarantine. Additional options, such as Network Protocol Enforcement (restricting known services to their default ports) and blocking applications on non-default ports, are detailed. The HTTP block page for application control, similar to web filtering, provides user feedback. The order of scan within an application control profile (application/filter overrides first, then categories) and its interaction with other security features like web filtering are explained.

Throughout the chapter, the necessity of deep SSL inspection for both IPS and Application Control to effectively analyze encrypted traffic is a recurring theme. The chapter concludes by guiding administrators on how to apply these security profiles to firewall policies and monitor their activity through logs, enabling effective management and troubleshooting of both intrusion prevention and application usage within the network.

## 2. Chapter Takeaways

### **Module 1: IPS Configuration**

**IPS (Intrusion Prevention System) Overview:**
*   Detects and prevents network attacks (exploits, anomalies) from compromising the network.
*   Utilizes:
    *   **IPS Signature Databases:** For known attacks and exploits (from FortiGuard). Includes rate-based signatures for anomaly detection (e.g., DoS-like behavior).
    *   **Protocol Decoders:** Parse traffic according to protocol specifications to detect malformed packets or non-compliant behavior.
    *   **IPS Engine:** Responsible for IPS inspection, protocol decoding, and also powers Application Control and flow-based AV.
*   IPS performs deep packet inspection, including scanning encrypted payloads if deep SSL inspection is enabled.
*   Primarily uses flow-based detection for anomalies and exploits.

**List of IPS Signatures & Sensors:**
*   **IPS Sensor:** A collection of IPS signatures and/or filters that define what the IPS engine scans for when the sensor is applied.
*   FortiOS includes predefined sensors (e.g., `default`, `all_default`, `protect_client`).
*   New signatures are downloaded from FortiGuard IPS package updates.
*   **Default Action:** Each signature has a default action; this can be overridden within a sensor. Change default actions cautiously (e.g., if a patch is released or for false positives).
*   The active signature database is displayed in the IPS sensor configuration.

**Configuring IPS Sensors:**
*   Located under `Security Profiles > Intrusion Prevention`.
*   **Methods to add signatures/rules to a sensor:**
    1.  **Add Individual Signatures:** Select specific signatures from the list.
    2.  **Add Filters:** Create rules based on signature attributes (Target, Severity, Protocol, OS, Application). All signatures matching the filter criteria are included.
*   **Rate-Based Signatures:**
    *   Trigger an action when a threshold of events is exceeded within a defined duration.
    *   Tracked by Source IP, Destination IP, or Session.
    *   Useful for mitigating brute-force attacks or scanning activity.
*   **Inspection Sequence:** Within an IPS sensor, filters and individual signatures are evaluated top-down. The first match applies. New entries are added to the bottom.
*   **IP Exemptions:** Specific source or destination IP addresses can be exempted from an *individual signature's* action. Useful for handling false positives without disabling the signature entirely.

**IPS Actions:**
*   When a signature in a sensor matches traffic:
    *   **Default:** Uses the signature's predefined default action.
    *   **Allow:** Permits traffic, no log.
    *   **Monitor:** Permits traffic and logs the event.
    *   **Block:** Drops the matching packet/session and logs the event.
    *   **Reset:** (TCP only) Sends a TCP RST packet to both client and server to terminate the connection and logs.
    *   **Quarantine:** Blocks traffic from the source IP for a configurable duration and logs.
*   **Packet Logging:** If enabled for a signature/filter, FortiGate stores a copy of the packet that triggered the IPS event. Useful for analysis but consumes resources.
*   `override-signature-hold-by-id` (CLI): Can temporarily set new FortiGuard signatures to "Monitor" to avoid false positives.

**Enabling Botnet Protection:**
*   Botnet C&C (Command and Control) protection is integrated within the IPS sensor.
*   Utilizes FortiGuard's botnet database (requires valid IPS license).
*   **Actions for Botnet C&C connections:**
    *   **Disable:** No scanning for botnet connections.
    *   **Block:** Blocks connections to known botnet servers.
    *   **Monitor:** Logs connections to known botnet servers.

**Applying IPS Inspection:**
*   Apply an IPS sensor to a firewall policy in the "IPS" field under Security Profiles.
*   **SSL Inspection:** For effective IPS on encrypted traffic (HTTPS, SSL/TLS), **deep SSL inspection** must be enabled on the same firewall policy.
*   **Logging:** By default, FortiGate logs IPS security events. For troubleshooting, ensure "Log Allowed Traffic" is set to "All Sessions" on the policy if trying to identify why an attack isn't being caught.

---

### **Module 2: IPS Monitoring**

**IPS Logging:**
*   View IPS events under `Log & Report > Security Events > Intrusion Prevention`.
*   Logs provide details like attack name, severity, action taken, source/destination IP, policy ID, etc.
*   Attack references (links to FortiGuard or CVE details) are often included.

**Troubleshoot IPS High-CPU Usage:**
*   Short CPU spikes during policy/profile changes are normal. Continuous high CPU by IPS processes (e.g., `ipsengine`, `ipshelper`) is problematic.
*   **CLI Command:** `diagnose test application ipsmonitor <option>`
    *   `1`: Display IPS engine information.
    *   `2`: Toggle IPS engine enable/disable status. (Shuts down IPS engine completely).
    *   `5`: Toggle IPS engine bypass status. (IPS engine active, but bypasses inspection).
    *   `99`: Restart all IPS engines and monitor.
*   If CPU drops when IPS is in bypass mode (option 5), traffic volume/complexity might be too high for the model.
*   If CPU remains high in bypass mode, or drops only when engine is disabled (option 2), suspect an issue within the IPS engine itself (contact Fortinet support).

**IPS Fail Open:**
*   Triggered when the IPS socket buffer is full, and new packets cannot be processed for inspection.
*   **CLI Configuration:** `config ips global set fail-open [enable|disable] end`
    *   **`enable`:** New packets bypass IPS inspection. Recommended where network availability is paramount over security (e.g., some datacenter scenarios).
    *   **`disable` (Default):** New packets are dropped.
*   **Troubleshooting Fail-Open Events:**
    *   Indicates IPS cannot keep up with traffic demands.
    *   Check logs for `action="drop"` msg="IPS session scan, enter fail open mode".
    *   Identify patterns: traffic volume increase, specific times of day.
    *   Optimize IPS profiles: Remove unneeded signatures/filters, especially for specific traffic types (e.g., Linux signatures on a Windows server protection profile). Disable IPS on policies for trusted internal traffic if not strictly necessary.

---

### **Module 3: Application Control Basics**

**Application Control Overview:**
*   Uses the IPS engine and operates in **flow-based scan mode** only (even if the firewall policy is proxy-based for other features).
*   Detects and acts on network application traffic, often beyond standard port/protocol identification.
*   Effective for detecting peer-to-peer (P2P) applications that use port randomization or encryption.

**Peer-to-Peer (P2P) Architecture:**
*   Clients (peers) download files from multiple other peers simultaneously.
*   Many servers, dynamic port numbers, optional dynamic encryption.
*   Hard to block with traditional firewalls due to evasive nature; requires sophisticated scanning like Application Control.

**Application Control—Hierarchical Structure:**
*   Application control signatures are organized hierarchically (e.g., Category > Application > Sub-Application).
    *   Example: Social.Media (Category) -> Facebook (Application) -> Facebook.Chat (Sub-Application).
*   The parent signature's action takes precedence over a child signature's action if conflicting.
*   Deep SSL inspection is often required for accurate identification of sub-applications within encrypted traffic.
    *   Certificate inspection may be sufficient for identifying some parent applications like "YouTube".

**List of Application Signatures:**
*   Viewable under `Security Profiles > Application Control`.
*   Signatures are updated via FortiGuard Application Control Service.
*   Can be filtered by name, category, technology, risk level, etc.
*   "Cloud Applications" section shows applications requiring deep inspection.

---

### **Module 4: Application Control Configuration**

**Application Control Filters Actions:**
*   Configured within an Application Control sensor/profile.
*   **Actions for categories or individual applications:**
    *   **Monitor (Default for many categories):** Allows traffic and logs.
    *   **Allow:** Allows traffic, no log by default.
    *   **Block:** Drops traffic and logs.
    *   **Quarantine:** Blocks traffic from the source and logs. User/IP can be quarantined for a specified time.
*   **View Signatures:** Displays signatures within a selected category (not a configurable action).

**Configuring Additional Options (in Application Control Profile):**
*   **Network Protocol Enforcement:**
    *   Allows known services (HTTP, FTP, etc.) only on their standard ports.
    *   Blocks these services if detected on non-standard ports.
    *   Configurable list of known services and their allowed ports.
*   **Block applications detected on non-default ports:** If an application (identified by its signature) is using a port not typically associated with it (as per FortiGuard definitions), it can be blocked.
*   **Replacement Messages for HTTP-based Applications:** Display a block page for HTTP/HTTPS applications. Non-HTTP apps are just dropped/reset.

**HTTP Block Page (Application Control):**
*   Provides feedback to users when an HTTP/HTTPS application is blocked.
*   **Information displayed:** Application name, Category, URL, Policy ID.
*   Customizable in `System > Replacement Messages`.

**Application Control Scanning Order:**
*   Within an Application Control profile, matches are evaluated in this order:
    1.  **Application and Filter Overrides:** Manually defined overrides for specific applications or custom filters. Checked top-down.
    2.  **Categories:** Action defined for the matched application's category is applied.
*   Application Control occurs *before* Web Filtering in the FortiGate's order of security profile inspection. An "Allow" in Application Control does not bypass Web Filtering.

**Applying an Application Control Profile:**
*   Apply the configured Application Control sensor/profile to a firewall policy in the "Application Control" field under Security Profiles.
*   **SSL Inspection:** For accurate detection and control of applications within encrypted traffic (especially HTTPS), **deep SSL inspection** is usually required on the same firewall policy.
*   **Logging:** Enable security event logging on the firewall policy to see Application Control events.

**Monitoring Application Control Logging:**
*   View logs under `Log & Report > Security Events > Application Control`.
*   **Details:** Application name, category, action, source/destination, policy ID, etc.
*   Forward Traffic logs will also show application control actions in the security details.

**Troubleshoot Traffic Matching Application Control Profile:**
*   If Application Control is not working as expected:
    1.  **Verify SSL Inspection:** Ensure deep SSL inspection is active for encrypted application traffic.
    2.  **Check Logs:** Review Application Control logs and Forward Traffic logs to see if traffic is matching the intended policy and what action is being taken.
    3.  **Signature Accuracy:** Ensure the application is correctly identified. Some generic traffic (e.g., general SSL) might be allowed by a broad category if more specific signatures aren't being matched.
    4.  **Overrides:** Check if an application or filter override is taking precedence.
    5.  **Policy Order:** Ensure the traffic is hitting the correct firewall policy.
*   Use `Dashboard > FortiView Applications` to see traffic matching specific applications and drill down into sessions.

---
