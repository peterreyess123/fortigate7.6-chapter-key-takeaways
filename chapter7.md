# Chapter 7: Antivirus

## 1. Chapter Summary

This chapter focuses on FortiGate's antivirus (AV) capabilities, detailing the various techniques and inspection modes used to detect and block malware. It begins by outlining the primary function of an AV scan: to stop viruses from harming systems or compromising connected devices, whether on individual endpoints (via FortiClient) or within network traffic inspected by a FortiGate NGFW. The core of FortiGate's AV engine relies on signature databases updated in real-time by the FortiGuard Antivirus service. The chapter explains the different scanning techniques employed, starting with traditional signature-based AV scans for exact matches, grayware scans for unsolicited programs, and more advanced AI/ML-based scans designed to detect zero-day malware in Windows PE files by identifying suspicious file features. The order of these scans, when all are enabled, is also noted.

The chapter then explores industry-standard techniques like Virus Outbreak Prevention Service (VOS), which subsidizes the local AV database with third-party malware hash signatures from FortiGuard's Global Threat Intelligence, and the use of external malware block lists hosted on web servers. The integration with FortiClient EMS for receiving malware feeds is also mentioned, along with the checking order when multiple threat intelligence sources are enabled. Content Disarm and Reconstruction (CDR) is introduced as a method to sanitize Microsoft Office and PDF files by removing active content, and behavior-based detection through FortiSandbox integration allows for submitting suspicious files for deeper analysis. CIFS scanning for inspecting files transferred over common internet file system traffic and general AI/ML behavioral analysis round out the detection capabilities.

A key distinction is made between the two main AV inspection modes: flow-based and proxy-based. Flow-based inspection, the default, scans files as they stream through the FortiGate, offering better performance and the ability to offload to SPUs on supported models. It uses a hybrid approach, with stream-based scanning for HTML/JavaScript and legacy scans for other file types or specific feature configurations. Proxy-based inspection, on the other hand, buffers the entire file before scanning, enabling more thorough inspection and supporting additional features like CDR and certain FortiNDR integrations, but at the cost of higher resource usage and potential latency. The chapter details the packet flow for each mode and how to configure AV profiles for both, emphasizing that proxy-based features are generally not supported on FortiGates with 2GB of RAM or less. Finally, it covers the configuration of protocol options, such as handling oversized or compressed files, the appearance and content of the antivirus block page, and methods for troubleshooting common AV issues, including license verification, update problems, and ensuring correct policy application.

## 2. Chapter Takeaways

### **Module 1: Antivirus Scanning Modes**

**Antivirus Techniques (General):**
*   **Primary Goal:** Detect and stop viruses/malware from compromising systems and networks.
*   **Engine:** FortiGate AV engine leverages signature databases from FortiGuard Antivirus service (real-time updates).
*   **Scanning Order (if all enabled):**
    1.  **Antivirus Scan (Signature-based):** Detects exact matches to known virus signatures in the database. Uses CPRL (Content Pattern Recognition Language) for efficiency.
    2.  **Grayware Scan:** Detects unsolicited programs installed without user consent (e.g., adware, spyware). Uses FortiGuard grayware signatures.
    3.  **AI Scan (Artificial Intelligence):** Integrates with regular AV to detect zero-day malware in Windows Portable Executable (PE) files by analyzing file features. Identifies files as `W32/AI.Pallas.Suspicious`.

**Industry-Standard AV Techniques (FortiGate):**
*   **Signature-based Detection:** As described above.
*   **Virus Outbreak Prevention (VOS):**
    *   Subsidizes AV database with third-party malware hash signatures from FortiGuard Global Threat Intelligence.
    *   Queries FortiGuard with file hash; a match deems the file malicious.
    *   Requires a valid FortiGuard Outbreak Prevention license.
*   **External Malware Block List:**
    *   Enhances AV database by linking to dynamic external block lists (MD5, SHA1, SHA256 hashes) hosted on web servers (HTTP/HTTPS).
    *   Requires creating a "Malware Hash" external connector in Security Fabric.
*   **EMS Threat Feed:**
    *   FortiGate receives malware feeds (hashes) from FortiClient EMS (which gathers them from FortiClients).
    *   **Checking Order (if all enabled):** Local AV DB > EMS Threat Feed > External Malware Blocklist > VOS.
*   **Content Disarm and Reconstruction (CDR):**
    *   Sanitizes MS Office documents and PDF files (including in ZIPs) by removing active content (hyperlinks, macros, JavaScript, etc.).
    *   Sends disarmed file to client.
    *   Supported for HTTP, SMTP, POP3, IMAP. **Requires proxy-based inspection.**
*   **Behavior-based Detection (FortiSandbox):**
    *   Submit suspected malicious files to FortiSandbox for dynamic analysis.
*   **CIFS Scanning:** File filtering and AV scanning on Common Internet File System (CIFS/SMB) traffic.
*   **AI/ML, Behavioral, and Human Analysis:** Broader terms for advanced threat detection capabilities, including the AI Scan for PE files. ML detection is enabled per-VDOM.

**Inspection Modes for Antivirus:**
*   **Flow-based Inspection:**
    *   **Default inspection mode** for new AV profiles.
    *   Scans files "on the fly" as they pass through, without buffering the entire file first.
    *   Optimized for performance; pattern matching can be offloaded to CP8/CP9 SPUsor accelerated by Nturbo on supported models.
    *   Uses a hybrid scan method: stream-based for HTML/JS, legacy scan for other types or when certain features are enabled.
    *   If a virus is found mid-stream, the connection is reset, and the latter part of the file is not delivered (file truncated).
    *   Fewer advanced AV features available (e.g., no CDR).
*   **Proxy-based Inspection:**
    *   FortiGate acts as a proxy, buffering the entire file before scanning.
    *   Allows for more thorough inspection and supports more advanced AV features (e.g., CDR, MAPI/SSH inspection, FortiNDR integration).
    *   Higher resource usage and potential for increased latency.
    *   Generally not supported on FortiGate models with 2GB RAM or less (feature set may be hidden or restricted).
*   Both modes use the full AV database (extended or extreme, depending on CLI settings/model).

**Flow-Based Inspection Mode Packet Flow:**
1.  Client requests a file.
2.  FortiGate receives packets and forwards them to the client while simultaneously caching them.
3.  IPS engine assembles the file from cached packets once the last packet is received (and held).
4.  File is sent to the AV engine for scanning.
5.  If clean, the last packet is released to the client.
6.  If infected, the connection is reset, last packet dropped, and the URL is cached for quicker blocking on subsequent attempts.

**Configuring Flow-Based Antivirus:**
1.  Create/Edit an Antivirus Profile (`Security Profiles > AntiVirus`).
2.  Ensure "Feature set" is set to "Flow-based".
3.  Select protocols to inspect (HTTP, SMTP, POP3, IMAP, FTP, CIFS).
4.  Define action for infected files (default "Block", or "Monitor").
5.  Apply the AV profile to a firewall policy.

**Stream-Based Antivirus Scanning in Flow-Based Inspection (Antivirus Engine 7.0+):**
*   Default scanning method for HTML and JavaScript files in flow-based AV.
*   AV engine determines necessary payload to buffer for scanning flow-based signatures, eliminating the need to cache the entire file for these types, improving memory usage.
*   **Not used if any of the following are enabled:** ML-based malware detection, extreme AV database, grayware scan, mobile malware DB, external block list, EMS threat feed, FortiGuard outbreak prevention, DLP, or file filter. (FortiGate then uses legacy flow-based scan for all file types).

**Proxy Inspection Mode Packet Flow:**
1.  Client sends a request.
2.  FortiGate (acting as proxy) intercepts. Connection is split: client-to-FortiGate and FortiGate-to-server.
3.  FortiGate buffers the entire file from the server.
4.  Once fully buffered, the AV engine scans the file.
5.  If clean, file is forwarded to the client. If infected, file is dropped, and a replacement message is sent.
*   Client comforting (HTTP/FTP) can be configured to slowly transmit data during buffering to prevent timeouts.

**Configuring Proxy-Based Antivirus:**
1.  Create/Edit an Antivirus Profile (`Security Profiles > AntiVirus`).
2.  Set "Feature set" to "Proxy-based".
3.  More options become available: MAPI, SSH, CDR, FortiNDR.
4.  Apply the AV profile to a firewall policy that has "Inspection Mode" set to "Proxy-based".

**Stream-Based Antivirus Scanning in Proxy-Based Inspection:**
*   Default scan mode within proxy inspection.
*   Decompresses and scans large archive files (ZIP, GZIP, BZIP2, TAR, ISO 9660) simultaneously to optimize memory.
*   Supports HTTP(S), FTP(S), SCP/SFTP. Does *not* support HTTP POST.
*   **Not supported if these features are enabled:** DLP, Quarantine, FortiGuard outbreak prevention, external block list, EMS threat feed, Content Disarm.

**Antivirus Block Page:**
*   Displayed when a virus is detected and blocked (primarily in proxy mode, or in flow mode on subsequent attempts if the URL was cached).
*   **Information shown:** File name, Virus name, Website host/URL, Username/group (if authenticated), Link to FortiGuard Encyclopedia for more virus details.

**Configuring Protocol Options:**
*   Located under `Policy & Objects > Protocol Options`. Applied to firewall policies.
*   Provide granular control over how various protocols are handled by security profiles (including AV).
*   **Oversize File Handling:**
    *   Default: Files larger than the oversize limit (default 10MB, hardware dependent max) are bypassed from scanning.
    *   Option: "Block Oversized File/Email" globally.
    *   CLI: `set oversize-log {enable|disable}` to log, `config <protocol_name> set options oversize set oversize-limit <integer>` to adjust per protocol.
*   **Compressed File Handling:**
    *   Archives are unpacked and scanned. Password-protected archives cannot be decompressed/scanned.
    *   `set uncompressed-oversize-limit`: Limit for individual files within an archive.
    *   `set uncompressed-nest-limit`: Max archive nesting level (default 12). Increasing impacts memory.
*   **Client Comforting:** (Proxy-mode HTTP/FTP) Sends small amounts of data to client during buffering to prevent timeouts.

---

### **Module 2: Troubleshooting**

**Monitoring Antivirus Events:**
*   **Antivirus Logs (`Log & Report > Security Events > AntiVirus`):**
    *   Generated when AV scan detects a virus.
    *   Details: Virus name, action, policy ID, AV profile, detection type, link to FortiGuard.
    *   "Size limit is exceeded" message if oversized file logging is enabled.
*   **Forward Traffic Logs (`Log & Report > Forward Traffic`):**
    *   If AV action occurs, security details will show AV information.
*   **Security Dashboard (`Dashboard > Security`):**
    *   Provides widgets to monitor threats, including AV detections.
    *   Advanced Threat Protection Statistics widget can be added.

**Troubleshooting Common Antivirus Issues:**
*   **FortiGuard License/Updates:**
    *   Verify license validity (`System > FortiGuard`).
    *   Force update: `execute update-av` (CLI).
    *   Check connectivity to `update.fortinet.net` (DNS, routing, port 443 TCP).
    *   Real-time debug: `diagnose debug application update -1`, `diagnose debug enable`, then `execute update-av`.
*   **Failure to Detect Viruses (with valid contract/DB):**
    *   **Policy Application:** Ensure correct AV profile is applied to the relevant firewall policy.
    *   **Protocol Port Mapping (Proxy-mode):** Ensure correct protocol options profile with correct port mappings is applied if non-standard ports are used.
    *   **SSL Inspection:** For encrypted traffic (HTTPS, SMTPS, etc.), deep SSL inspection must be enabled with an appropriate SSL inspection profile on the same firewall policy.
*   **Useful CLI Commands for AV Status:**
    *   `get system performance status`: Shows "Virus caught" statistics.
    *   `diagnose antivirus database-info`: Shows current AV DB version and details.
    *   `diagnose autoupdate versions`: Shows AV engine and definition versions, contract expiry, last update attempt/result.
    *   `diagnose antivirus test "get scantime"`: Shows distribution of scan times for infected files.

---
