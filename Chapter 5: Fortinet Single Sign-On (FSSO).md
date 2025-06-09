# Chapter 5: Fortinet Single Sign-On (FSSO)

## 1. Chapter Summary

This chapter provides a comprehensive overview of Fortinet Single Sign-On (FSSO), a technology designed to transparently identify network users for security policy enforcement without requiring them to re-authenticate at the FortiGate. It begins by differentiating SSO as a general concept (allowing identified users access to multiple applications post-initial login) from FSSO, which is Fortinet's specific implementation primarily for integrating with directory services like Windows Active Directory (AD) and Novell eDirectory. FSSO works by monitoring user login events on these directory services and then relaying user ID, IP address, and group membership information to the FortiGate, which maintains a local database for policy matching.

The core of the chapter details FSSO deployment and configuration, with a strong focus on Windows AD integration. It explains the different FSSO modes: DC Agent mode, Collector Agent-based Polling mode, and Agentless Polling mode. DC Agent mode, the most scalable and recommended, involves installing a `dcagent.dll` on each Domain Controller (DC) to monitor login events and forward them to a Collector Agent. The Collector Agent, installed on a Windows server, then consolidates these events, performs group lookups, and communicates with the FortiGate. Collector Agent-based Polling mode also uses a Collector Agent but polls DCs for event logs (via WMI, WinSecLog, or NetAPI) instead of relying on DC agents, offering a less complex installation. Agentless Polling mode has the FortiGate itself poll the DCs (WinSecLog only), suitable for simpler networks but less scalable and feature-rich. The chapter also touches upon Terminal Server (TS) agents for Citrix/Terminal Server environments where multiple users share an IP.

Configuration aspects are thoroughly covered, including installing FSSO agents (Collector Agent, DC Agent), setting up communication ports (default TCP 8000 for FortiGate-CA, UDP 8002 for DC-CA), and configuring AD access modes (Standard vs. Advanced for naming conventions and nested group support). The critical role of group filters in optimizing performance by sending only relevant group information to FortiGate is emphasized. The chapter also explains various FSSO Collector Agent settings, such as NTLM authentication support, SSL for secure communication, and crucial timers like workstation verify interval, dead entry timeout, and IP address change verify interval. Advanced settings like Citrix/Terminal Server integration, RADIUS Accounting, and Syslog server forwarding are also mentioned. Finally, the chapter addresses troubleshooting common FSSO issues, including monitoring FSSO-related log messages on FortiGate, interpreting Collector Agent logs, and using CLI diagnostic commands to check user lists, server status, and polling details.

## 2. Chapter Takeaways

### **Module 1: FSSO Deployment**

**SSO and FSSO Concepts:**
*   **SSO (Single Sign-On):** A general process allowing users, once authenticated to a primary system (e.g., domain login), to access multiple applications/resources without re-entering credentials.
*   **FSSO (Fortinet Single Sign-On):** Fortinet's implementation of SSO, primarily for transparently identifying network users based on their directory service logins (e.g., Windows AD, Novell eDirectory).
    *   FSSO software (agents) identify user ID, IP address, and group membership.
    *   FortiGate uses this information to match users to FSSO user groups in firewall policies, allowing access without an active authentication prompt at the firewall.

**FSSO Deployment and Configuration Overview:**
*   **Microsoft Active Directory (AD) Integration:**
    *   **Domain Controller (DC) Agent Mode:** Most scalable and recommended.
    *   **Polling Mode (Collector Agent-based):** Collector agent polls DCs.
    *   **Polling Mode (Agentless):** FortiGate polls DCs directly.
    *   **Terminal Server (TS) Agent:** For environments where multiple users share an IP (e.g., Citrix, RDS). Requires a Collector Agent or FortiAuthenticator.
*   **Novell eDirectory Integration:**
    *   Uses an eDirectory agent mode.
    *   Leverages Novell API or LDAP settings.

**DC Agent Mode (for Windows AD):**
*   **Components:**
    *   **DC Agent (`dcagent.dll`):** Installed on each Windows DC in the `Windows\system32` directory.
        *   Monitors user login events directly from the DC's security event log or WMI.
        *   Forwards login events to Collector Agent(s).
        *   Handles DNS lookups by default to resolve workstation names to IP addresses.
    *   **Collector Agent (CA):** Installed on a Windows server (member of the domain).
        *   Consolidates login events from multiple DC Agents.
        *   Performs group verification (checks user's group memberships in AD).
        *   Conducts workstation checks (verifies user is still logged on).
        *   Updates FortiGate(s) with login records (user, IP, groups).
        *   Communicates with FortiGate over TCP (default port 8000). Listens for DC agent updates over UDP (default port 8002).
*   **Process Flow:**
    1.  User authenticates against Windows DC.
    2.  DC Agent detects login event, forwards to Collector Agent.
    3.  Collector Agent receives event, optionally performs DNS lookup/workstation check, verifies group membership, and forwards information (user, IP, groups) to FortiGate.
    4.  FortiGate updates its FSSO user list. When traffic from that user's IP arrives, FortiGate matches it to policies based on FSSO group membership.

**Collector Agent-Based Polling Mode (for Windows AD):**
*   **Collector Agent (CA):** Installed on a Windows server. No DC agent (`dcagent.dll`) is required on DCs.
*   **Operation:** CA periodically polls each DC for user login events.
    *   Less complex installation than DC Agent mode.
    *   Can generate more network traffic due to polling, especially if no new logins.
    *   Event logging must be enabled on DCs (except for NetAPI method).
*   **Polling Methods (Options within CA configuration):**
    *   **WMI (Windows Management Instrumentation):** Recommended. Reads selected event logs. Reduces network load compared to full log polling.
    *   **WinSecLog (Windows Security Event Log):** Polls all security events from DCs. Can have log latency.
    *   **NetAPI:** Polls temporary `NetSessionEnum` function on Windows. Faster but can miss logins under heavy DC load. Retrieves DC login events directly.
*   **Process Flow:** Similar to DC Agent mode from the CA to FortiGate, but CA actively pulls data from DCs.

**Agentless Polling Mode (for Windows AD):**
*   **No external agents (DC Agent or Collector Agent) required.**
*   **FortiGate acts as the collector:** Directly polls DCs for user login events using WinSecLog method (Event IDs 4768, 4769).
    *   FortiGate uses SMB (TCP 445) to read event logs.
*   **Limitations:**
    *   More CPU and RAM intensive on the FortiGate.
    *   Supports WinSecLog polling option only.
    *   Fewer available features compared to Collector Agent-based modes (e.g., no workstation verification by FortiGate itself).
*   Suitable for simple networks with a minimal number of users and DCs.

**Comparing FSSO Modes:**
| Feature             | DC Agent Mode                                  | Collector Agent Polling                | Agentless Polling             |
|---------------------|------------------------------------------------|----------------------------------------|-------------------------------|
| **Installation**    | Complex (agent on each DC, CA on server)       | Simpler (CA on server only)            | Simplest (FortiGate only)     |
| **DC Agent Required**| Yes                                            | No                                     | No                            |
| **Scalability**     | Higher                                         | Moderate                               | Lower                         |
| **Resources on DC** | Lower (agent pushes events)                    | Higher (polled by CA)                  | Higher (polled by FortiGate)  |
| **Resources on FG** | Lower                                          | Moderate                               | Higher                        |
| **Reliability**     | Captures all logins (event-driven)             | Can miss logins (NetAPI), delay (WinSecLog) | Can have delay (WinSecLog)    |
| **Redundancy**      | Multiple CAs can receive from DC Agents        | Multiple CAs can poll (more overhead)  | Relies on FortiGate HA        |
| **Level of Confidence** | Captures all logins                          | Might miss (NetAPI) or delay (WinSecLog) | Might have delay (WinSecLog)  |

**Additional FSSO AD Requirements:**
*   **DNS Resolution:** Microsoft login events contain workstation names, not IPs. The Collector Agent (or FortiGate in agentless mode) must use DNS to resolve workstation names to IP addresses. Accurate and timely DNS updates are crucial.
*   **Workstation Polling (for full functionality with CA):** To verify if a user is still logged in (since logout events may not always be reliably captured), the Collector Agent polls workstations.
    *   Uses WMI by default.
    *   Requires TCP ports 445 (SMB) and 139 (NetBIOS) to be open between CA/FortiGate and workstations.

---

### **Module 2: FSSO Settings**

**FSSO Configuration on FortiGate – Agentless Polling Mode:**
*   Navigate to `Security Fabric > External Connectors`.
*   Create a new connector, select "Poll Active Directory Server".
*   Provide AD administrator credentials and IP addresses for each DC to be polled.
*   FortiGate uses LDAP (configure an LDAP server object and link it here) to query AD for user group information.

**FSSO Configuration on FortiGate – Collector Agent-Based or DC Agent Mode:**
*   Navigate to `Security Fabric > External Connectors`.
*   Create a new connector, select "Fortinet Single-Sign-On Agent".
*   Provide IP address and password for each Collector Agent.
*   **User Group Source:**
    *   **Collector Agent:** FortiGate relies on group filters defined on the Collector Agent.
    *   **Local:** FortiGate acts as an LDAP client to query AD for group information; group filters are defined on FortiGate. Collector Agent must be in Advanced AD access mode for this to work effectively.

**FSSO Agent Installation (Collector Agent & DC Agent):**
*   Download agents from Fortinet Support website (`https://support.fortinet.com` under Firmware Download > FortiGate > FSSO).
*   **Collector Agent Installation Wizard:**
    1.  Run as Administrator.
    2.  Provide service account credentials (e.g., `DomainName\UserName`).
    3.  Configure monitoring options (event log sources, NTLM, directory access). Standard mode is default; Advanced mode offers more granular control.
    4.  Optionally launch DC Agent Install Wizard.
*   **DC Agent Installation Wizard:**
    1.  Specify Collector Agent IP and listening port (default UDP 8002).
    2.  Select domains to monitor.
    3.  Optionally specify users to ignore (e.g., service accounts).
    4.  Select DCs to install the agent on and choose the working mode (DC Agent Mode or Polling Mode - choosing Polling here means no DC agent will be installed, CA will poll).
    5.  Requires DC reboot for DC Agent mode.

**FSSO Collector Agent Configuration (GUI Console):**
*   **Key Settings:**
    *   **Listening port for DC Agents:** Default UDP 8002.
    *   **Listening port for FortiGates:** Default TCP 8000.
    *   **Require authenticated connection from FortiGate:** Password protection.
    *   **Enable SSL:** Secure communication with FortiGates.
    *   **NTLM Authentication Support:** Enable if NTLM logins need to be monitored.
    *   **Timers:**
        *   **Workstation verify interval:** Default 5 minutes. Checks if user is still logged on.
        *   **Dead entry timeout:** Default 480 minutes (8 hours). Removes unverified entries.
        *   **IP address change verify interval:** Default 60 seconds. Checks for IP changes (DHCP environments).
        *   **Cache user group lookup result:** Caches group membership.
*   **Set Directory Access Information (AD Access Mode):**
    *   **Standard Mode:** Windows convention (`Domain\group`). Simpler, no nested groups. Group filters on CA.
    *   **Advanced Mode:** LDAP convention (`CN=User,OU=Org,DC=domain,DC=com`). Supports nested/inherited groups, filtering on FortiGate (if Local group source) or CA.

**Group Filters:**
*   Control which AD user group information is sent to FortiGate. Essential for performance and relevance.
*   Configured on the FSSO Collector Agent.
*   Filters are tied to specific FortiGate serial numbers (or a default filter for all).
*   Can filter by Groups, OUs, or individual Users.

**Ignored User List:**
*   Configured on the Collector Agent.
*   Login events from users/OUs in this list are not reported to FortiGate.
*   Useful for service accounts, backup accounts, etc., that might overwrite legitimate user login events.

**Advanced Settings (Collector Agent):**
*   **Citrix/Terminal Server:** Configure TS Agent mode for monitoring user logins in multi-user shared IP environments.
*   **RADIUS Accounting:** Integrate with RADIUS servers to receive accounting messages for user logins/logouts.
*   **Syslog Servers:** Forward FSSO events to syslog servers.
*   **Event IDs to poll:** Customize which Windows Event IDs are monitored in WinSecLog polling mode.

**FortiGate FSSO Group Object Configuration:**
*   Create FSSO user groups on FortiGate (`User & Authentication > User Groups`).
*   Select "Fortinet Single Sign-On (FSSO)" as Type.
*   Add AD groups (members) obtained from the Collector Agent.
    *   In Standard AD access mode on CA, only AD groups can be selected.
    *   In Advanced AD access mode on CA, AD users, groups, and OUs can be selected.
*   `SSO_Guest_Users` group: Default group. If passive FSSO is used, users not part of any other FSSO group may fall into this (behavior varies).

---

### **Module 3: Troubleshooting**

**Troubleshooting Tips for FSSO:**
*   **Connectivity:** Ensure firewalls allow FSSO ports (TCP 8000, UDP 8002, LDAP/S 389/636, SMB 445, WMI ports).
*   **DNS:** Accurate and timely DNS is critical for resolving workstation names to IPs.
*   **Service Account Permissions:** Collector Agent service account needs appropriate permissions (e.g., read event logs, query AD).
*   **Workstation Polling:** Ensure CA can reach workstations for verification (WMI/RPC ports).
*   **Timers:** Adjust timers (workstation verify, dead entry) based on network environment to avoid premature logouts or stale entries.
*   **Group Filters:** Incorrect or missing group filters are a common issue.
*   **AD Access Mode:** Ensure consistency between CA and FortiGate expectations if using "Local" group source on FortiGate.

**FSSO Log Messages on FortiGate:**
*   View under `Log & Report > Events > User Events`.
*   Key log messages/IDs:
    *   `43012`: FSSO authentication successful.
    *   `43013`: FSSO authentication failed.
    *   `43014`: FSSO user logged on.
    *   `43015`: FSSO user logged off.
*   Set minimum log level to Notification or Information for comprehensive FSSO event logging.

**Log Messages on FSSO Collector Agent:**
*   Accessible via the FSSO Collector Agent GUI console.
*   **Log Level:** Debug, Information, Warning, Error.
*   **View Log:** Shows all FSSO agent logs.
*   **View Login Events:** If "Log login events in separate logs" is enabled, shows a summary of events sent/removed from FortiGate.

**FSSO CLI Diagnostics on FortiGate:**
*   **Currently Logged-In Users:**
    *   `diagnose debug authd fsso list`
    *   Shows User, Groups, IP, Workstation, MemberOf (FortiGate FSSO group).
    *   `execute fsso refresh`: Manually refresh user group info from CA.
*   **FSSO Collector Agent Server Status:**
    *   `diagnose debug authd fsso server-status`
    *   Shows connection status, version, and IP of connected CAs.
*   **General FSSO commands:** `diagnose debug authd fsso ?` for options like `clear-logons`, `refresh-groups`.
*   **Polling Mode Specifics (if FortiGate is polling directly):**
    *   `diagnose debug fsso-polling detail`: Status of polls to DCs.
    *   `diagnose debug fsso-polling refresh-user`: Flushes locally learned FSSO users.
    *   `diagnose sniffer packet any 'host <DC_IP> and tcp port 445'`: Sniff SMB traffic for WinSecLog polling.
    *   `diagnose debug application fssod -1`: Real-time debug for the FSSO polling daemon.

---
