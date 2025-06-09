# Chapter 1: System and Network Settings

## 1. Chapter Summary

This chapter lays the groundwork for FortiGate administration, beginning with an understanding of its role in modern, complex network security environments beyond traditional perimeter defense. It details the FortiGate platform's architecture, emphasizing its internal modularity and the use of specialized ASICs for enhanced performance. The initial setup process is covered, starting from factory default settings, including the default IP address and administrative credentials, and progresses to configuring essential network connectivity through interface IP addressing (manual, DHCP, or PPPoE), understanding interface roles versus aliases, and optionally setting up the FortiGate as a DHCP server. The chapter then introduces logical network segmentation using VLANs, the critical need for a static gateway or default route for outbound communication, especially for FortiGuard services, and the concept of VDOMs for partitioning the device into multiple virtual firewalls.

Building on the initial setup, the chapter transitions into basic administration practices. It outlines the various methods for managing a FortiGate, including GUI, CLI, API, and FortiManager. A significant focus is placed on securing administrative access by creating unique administrator accounts with granular permissions defined by administrator profiles (such as `super_admin` and `prof_admin`), implementing trusted hosts to restrict login sources, and customizing administrative ports alongside robust password policies. The importance of enabling specific administrative protocols like HTTPS and SSH only on necessary interfaces is also highlighted, along with understanding other interface-level protocols related to the Security Fabric.

Finally, the chapter covers fundamental maintenance tasks crucial for ongoing operation and security. This includes the procedures for backing up and restoring system configuration files, emphasizing the differences between encrypted and unencrypted backups and their respective restore requirements. The process of upgrading FortiGate firmware is discussed, stressing the importance of consulting Release Notes. A detailed overview of FortiGuard Subscription Services explains their vital role in providing real-time threat intelligence, the different types of updates (packages vs. live queries), and how to verify licenses. The chapter concludes by presenting the initial stages of how a packet traverses the FortiGate, offering a foundational understanding of its processing pipeline from network processor handling to kernel-level routing and inspection. Mastering these system and network settings is paramount for establishing a secure and functional FortiGate deployment.

## 2. Chapter Takeaways

### **Module 1: Initial Setup**

**Modern Context of Network Security:**
*   Firewalls are more than perimeter gatekeepers; they address multifaceted threats in complex, borderless environments (mobile workforce, IoT, cloud).
*   FortiGate can assume various deployment roles: NGFW, ISFW, Data-center firewall, and provide services like DNS, DHCP, web filtering, IPS.
*   Today’s networks require robust, multifunctional devices due to the erosion of the traditional perimeter and increased attack vectors.

**Platform Design:**
*   FortiGate platforms are internally modular, offering strength without sacrificing flexibility.
*   Hardware utilizes Application-Specific Integrated Circuits (ASICs) like Content Processors (CPs) and Network Processors (NPs) for accelerated performance in tasks like cryptography and packet forwarding.
*   Designed for flexibility (enable/disable UTM/NGFW features per policy) and cooperation (open standards, Security Fabric integration).

**Factory Default Settings:**
*   **Default IP Address:** `192.168.1.99/24` on `port1` or `internal` interface.
*   **Default Admin Credentials:** Username: `admin`, Password: (blank). Must be changed immediately upon first login.
*   **Enabled Admin Protocols:** PING, HTTPS, SSH on the default management interface.
    *   Entry-level models often have a DHCP server enabled on `port1` or the `internal` interface.
*   **Access Methods:** GUI via web browser (HTTPS), CLI via console port (no network needed), CLI console widget, or terminal emulator (PuTTY, Tera Term).

**Interface IPs:**
*   In NAT mode (default), interfaces handling traffic must have an IP address.
*   **Assignment Methods:**
    *   **Manual:** Static IP address configuration.
    *   **DHCP:** Interface obtains IP address from a DHCP server.
    *   **PPPoE:** Interface obtains IP address via Point-to-Point Protocol over Ethernet (often used for DSL).
*   IP addresses are used for sourcing traffic from FortiGate and as a destination for devices routing through or to the FortiGate.

**Interface Role Compared to Alias:**
*   **Role:** Defines a typical set of interface settings to simplify GUI configuration and prevent misconfiguration.
    *   **Types:** LAN, WAN, DMZ, Undefined (shows all settings).
    *   Hidden settings based on role can be revealed by selecting "Undefined".
*   **Alias:** A user-friendly, descriptive name for an interface (e.g., `internal_network`).
    *   Used in firewall policy lists to improve readability and understanding of interface purpose.

**FortiGate as a DHCP Server:**
*   Can be configured on any interface (typically LAN-facing).
*   Requires the interface to have a manually assigned static IP address first.
*   **Configuration Options:** IP address range, DNS server, default gateway, lease time, advanced DHCP options (e.g., NTP), and IP address assignment rules (MAC-to-IP mapping).
*   DHCP relay can also be configured to forward requests to an external DHCP server.

**Virtual LANs (VLANs):**
*   Logically subdivide a physical Layer 2 network into multiple broadcast domains.
*   **IEEE Standards:** `802.1Q` (basic VLAN tagging, up to 4094 VLANs), `802.1ad` (QinQ, VLAN stacking for scalability).
*   Each VLAN is a separate logical interface bound to a physical interface, identified by a VLAN ID (tag).
    *   Frames on the physical interface segment (native VLAN, VLAN ID 0) are untagged.
*   In a multi-VDOM environment, a physical interface and its VLAN subinterface can be in separate VDOMs.

**Static Gateway (Default Route):**
*   Essential for routing traffic to networks not directly connected, especially the internet.
*   **Configuration:** Manually configured as a static route with destination `0.0.0.0/0.0.0.0` (or `::/0` for IPv6).
*   The gateway IP address is the next-hop router, typically the ISP's router.
*   Required for FortiGuard updates and resolving external services if FortiGate is the edge device.
    *   If an interface uses DHCP/PPPoE, it may dynamically receive a default gateway.

**Virtual Domains (VDOMs):**
*   Split a single FortiGate into multiple independent virtual firewall devices.
*   Each VDOM has its own security policies, routing tables, interfaces, and administrators.
*   Traffic is confined to its VDOM by default; inter-VDOM routing can be configured if needed.
*   Most FortiGate models support up to 10 VDOMs by default; high-end/virtual models may allow more (licensed).

---

### **Module 2: Basic Administration**

**Administration Methods:**
*   **GUI (Graphical User Interface):** Web-browser based, most common for general configuration.
*   **CLI (Command Line Interface):** Accessible via console port, SSH, or GUI widget. Essential for advanced diagnostics, scripting, and some specific configurations not in GUI.
*   **API (Application Programming Interface):** For programmatic access and automation (e.g., Fortinet Developer Network, Ansible, Terraform).
*   **FortiManager:** Centralized management for multiple FortiGate devices. Uses FGFM protocol.
*   **Read-only Protocols:** SNMP for monitoring.

**Create an Administrative User:**
*   Best practice: Create separate accounts for each administrator for accountability and security.
*   **Types:**
    *   **Administrator:** Standard admin user.
    *   **REST API Admin:** For API access.
*   **Authentication:** Local password, remote server (LDAP, RADIUS), or digital certificates.
*   **Password Strength:** Enforce strong, complex passwords. Audit with tools like L0phtcrack or John the Ripper.

**Administrator Profiles:**
*   Define granular access permissions (Read-Write, Read-Only, None) for different FortiGate configuration areas.
*   **Default Profiles:**
    *   `super_admin`: Full global access, cannot be changed. Associated with the `admin` account.
    *   `prof_admin`: Full access to VDOM settings, not global settings. Can be modified.
*   **Custom Profiles:** Create profiles like `auditor_access` (read-only) to adhere to the principle of least privilege.
*   **Override Idle Timeout:** Per-profile setting to override the global admin idle timeout.
*   Administrators with limited scope cannot view or create accounts with higher permissions.

**Administrative Access—Trusted Hosts:**
*   Restrict administrative logins to specific source IP addresses or subnets for enhanced security.
*   Configured per administrator account.
*   Default is `0.0.0.0/0` (any IPv4 address), which should be changed for better security.
*   Consider NAT between the admin station and FortiGate, as the source IP checked is the one FortiGate sees.
*   Option to restrict an admin account to only provision guest accounts (requires a guest user group).

**Administrative Access—Ports and Password:**
*   **Admin Ports:** Customizable ports for HTTPS (default 443), HTTP, SSH (default 22), Telnet. Best practice: use only secure protocols (HTTPS, SSH) and change default ports.
*   **Idle Timeout:** Default is 5 minutes. System-wide setting for inactive admin sessions. Can be overridden by Admin Profile setting.
*   **Password Policy:** Enforce complexity, expiry, reuse limitations.
    *   `Minimum length`, `Character requirements` (uppercase, lowercase, number, special), `Password expiration`.
*   **Concurrent Sessions:** Option to allow or disallow multiple simultaneous admin logins. Disallowing prevents accidental overwrites.

**Administrative Access—Protocols (Interface-level):**
*   Enable or disable administrative protocols (HTTPS, PING, SSH, HTTP, etc.) on a per-interface basis.
*   Separate configuration for IPv4 and IPv6.
*   Disable unnecessary admin protocols on internet-facing interfaces to reduce attack surface.
*   **Other Destination Protocols for FortiGate:**
    *   **Security Fabric Connection:** CAPWAP (for FortiAP/FortiSwitch), FortiTelemetry (for FortiClient/Fabric).
    *   **FMG-Access:** For FortiManager communication.
    *   **FTM:** FortiToken Mobile push.
    *   **RADIUS Accounting:** For SSO.
*   **LLDP (Link Layer Discovery Protocol):** Supported for detecting upstream Security Fabric FortiGates.

---

### **Module 3: Fundamental Maintenance**

**Configuration File—Backup and Restore:**
*   **Backup:** Save system configuration to an external device (local PC, USB on some models).
    *   **Options:** Encrypt with a password, Mask passwords and secrets (for sharing with support).
    *   YAML format export available.
*   **Restore:** Upload a configuration file. FortiGate reboots after restore.
    *   VDOM-specific restore only reboots that VDOM's processes.
*   Essential for disaster recovery and quick rollout of new devices.

**Configuration File Format:**
*   Contains a cleartext header with model, firmware version, build number, and VDOM information.
*   **Encrypted File:** After the header, content is unreadable without the password. Requires same model and firmware build for restore.
*   **Unencrypted File:** Readable text. Requires same model for restore; firmware should match as best practice (FortiGate may attempt to upgrade config if versions differ).
*   Typically includes only non-default settings to minimize file size.

**Upgrade Firmware:**
*   View current firmware on Dashboard (System widget) or System > Firmware & Registration page.
*   **Notification:** GUI indicates if an updated firmware version is available.
*   **Upgrade Methods:**
    *   **All Upgrades / File Upload:** From GUI using downloaded firmware image.
    *   **Fabric Upgrade:** Upgrades root FortiGate and managed Fabric devices.
*   **Crucial Prerequisite:** ALWAYS read the FortiOS Release Notes for the target version to understand supported upgrade paths, known issues, and specific instructions.
*   CLI command: `get system status` shows firmware version.

**FortiGuard Subscription Services:**
*   Provide up-to-date threat intelligence (AV, IPS, web/DNS filtering, antispam, etc.). Requires active licenses and internet connectivity.
*   **FDN (FortiGuard Distribution Network):** Global network of data centers providing updates and real-time query responses.
    *   FortiGate selects server based on load/location, or can be set to USA-only.
*   **Update Types:**
    *   **Packages:** AV/IPS engines and signatures. Downloaded (typically daily) via TCP port 443 (SSL).
    *   **Live Queries:** Web filtering, DNS filtering, antispam. Real-time queries via UDP port 53/8888 or HTTPS port 443/53/8888.
*   **DNS for FortiGuard:** FortiOS uses DNS over TLS (DoT) by default to secure DNS traffic to FortiGuard servers.
*   **Access Mode:** `anycast` is default, optimizing routing. Forces HTTPS on port 443 for rating.
*   **SSL Certificate Verification & OCSP Stapling:** Ensures secure and validated communication with FortiGuard servers.

**FortiGuard Licenses:**
*   Verify license status and database versions via GUI (System > FortiGuard) or CLI (`diagnose autoupdate versions`).
*   **Key Information from CLI:** AV Engine version, Virus Definitions version, contract expiry dates, last update times, update results.
*   Ensures features like AV, IPS, web filtering are effective with the latest protection.

**Life of a Packet—Initial Session Packets:**
*   A foundational understanding of how FortiGate processes the first packets of a new session:
    1.  **Ingress Network Interface:** Packet arrives.
    2.  **Network Processor (NP) - Early Checks:** Access Control List (ACL), Host Protection Engine (HPE), IP integrity header checking, IPsec VPN decryption (if applicable).
    3.  **Kernel - Routing & Session Setup:** Destination NAT (DNAT), Routing lookup (RPF check, SD-WAN decision), Stateful inspection/policy lookup, Session creation.
    4.  **Kernel - User & Device ID:** Session helpers, User authentication, Device identification.
    5.  **Kernel - SSL VPN Processing** (if applicable).
    6.  **Kernel - Local Management Traffic Handling** (if destined for FortiGate itself).
    7.  **UTM/NGFW Inspection (CPU/CP):**
        *   **Flow-based inspection:** Antivirus, IPS, Web Filter, App Control.
        *   **Proxy-based inspection:** (If policy uses proxy mode) Antivirus, Web Filter, DLP, etc.
        *   Explicit web proxy, Botnet check.
    8.  **Kernel - Forwarding & Egress:** Source NAT (SNAT), IPsec VPN encryption (if applicable).
    9.  **Kernel - Traffic Shaping.**
    10. **Kernel - WAN Optimization.**
    11. **Egress Network Interface:** Packet departs.
*   Offloading to NP/CPs significantly improves performance. Virtual FortiGates use CPU for all processing.

---
