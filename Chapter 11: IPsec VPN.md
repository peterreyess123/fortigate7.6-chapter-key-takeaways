# Chapter 11: IPsec VPN

## 1. Chapter Summary

This chapter provides an in-depth exploration of IPsec VPNs, a standard-based suite of protocols used to create secure, encrypted tunnels between remote hosts or networks over an untrusted network like the internet. It begins by defining IPsec's core benefits: authentication (verifying peer identity), data integrity (ensuring data isn't tampered with via HMAC), and data confidentiality (encrypting data using protocols like AES or 3DES). The chapter breaks down the IPsec protocol suite, explaining the roles of Internet Key Exchange (IKE) for negotiating keys and Security Associations (SAs), and Encapsulating Security Payload (ESP) for encapsulating and encrypting the actual data. Authentication Header (AH), which provides integrity but not encryption, is mentioned as part of the IPsec standard but noted as not used by FortiGate. The differences in port usage with and without Network Address Translation (NAT) are highlighted, particularly the role of UDP port 4500 for NAT Traversal (NAT-T).

The chapter then contrasts the two main encapsulation modes: tunnel mode, which encapsulates the entire original IP packet and adds a new IP header (true tunneling, common for site-to-site VPNs), and transport mode, which only encrypts the payload and is typically used for end-to-end security between hosts. A detailed comparison between IKEv1 and IKEv2 is provided, with IKEv2 presented as simpler and more robust due to its single exchange mode and native NAT-T support, though IKEv1 remains widely adopted. The negotiation process, centered around Security Associations (SAs), is explained through its two distinct phases. Phase 1 establishes a secure IKE SA (control channel) using either Main mode (more secure, ID protection) or Aggressive mode (faster, IDs exposed earlier), with authentication via Pre-Shared Keys (PSK) or digital signatures (certificates), and optionally XAuth for extended user authentication. Phase 2 negotiates IPsec SAs (data channels) over the secure IKE SA, defining the "interesting traffic" via selectors (Proxy IDs) and using Perfect Forward Secrecy (PFS) for enhanced key security. Different VPN topologies are discussed: remote access (dial-up server for FortiClient or other clients), site-to-site (simple point-to-point), and hub-and-spoke, along with their respective advantages and disadvantages.

Configuration of IPsec VPNs on FortiGate is a central theme, covering both the IPsec wizard for simplified setup (especially for FortiClient VPNs) and manual configuration. Key Phase 1 settings (network details like remote gateway type, IKE version, authentication method, DH groups, lifetime) and Phase 2 settings (selectors, encryption/authentication algorithms, PFS, key lifetimes, replay detection, auto-negotiate, autokey keep alive) are detailed. The chapter also explains specific network settings like IKE Mode Config (for dynamically assigning IPs to dial-up clients, similar to DHCP), NAT Traversal, and Dead Peer Detection (DPD) for monitoring tunnel liveness. The role of IPsec hardware offloading on supported FortiGate models for accelerating encryption/decryption is mentioned. Finally, the chapter addresses routing and firewall policies for route-based IPsec VPNs (the recommended type), explaining how static routes or dynamic routing protocols are used with the virtual tunnel interface and how firewall policies control traffic entering and exiting the tunnel. Configuring redundant VPNs for failover using multiple tunnels with different route distances/priorities concludes the practical application aspects. Monitoring IPsec VPN status, routes, and logs, along with common troubleshooting commands and scenarios, are also covered.

## 2. Chapter Takeaways

### **Module 1: IPsec Basics**

**What Is IPsec?:**
*   A suite of protocols to provide secure communication over IP networks.
*   **Core Benefits/Services:**
    *   **Authentication:** Verifies the identity of VPN peers.
    *   **Data Integrity (HMAC):** Ensures data has not been tampered with during transit.
    *   **Data Confidentiality (Encryption):** Ensures only intended recipients can read the data.
*   Joins remote hosts and networks into a private network over an untrusted medium (e.g., Internet).

**What Is the IPsec Protocol Suite?:**
*   **Multiple protocols work together:**
    *   **IKE (Internet Key Exchange):** Used to negotiate Security Associations (SAs), authenticate peers, and generate/exchange keys. This is the *control channel*.
        *   Default ports: UDP 500. Uses UDP 4500 if NAT-T is active.
    *   **ESP (Encapsulating Security Payload):** Provides confidentiality (encryption), data origin authentication, integrity, and anti-replay protection for the IP payload. This is the *data channel*.
        *   IP protocol 50. Encapsulated in UDP 4500 if NAT-T is active.
    *   **AH (Authentication Header):** Provides integrity and authentication for the IP header and payload but **no encryption**. IP protocol 51. **FortiGate does not use AH.**
*   **Custom IKE Port:** FortiGate allows configuring a custom port for IKE (initiator and responder) via `config system settings set ike-port <port>`. Responder always listens on UDP 4500 as well for NAT-T.

**How Does IPsec Work?:**
*   **Encapsulation:** Wraps original IP packets with IPsec headers.
    *   **Transport Mode:** Encrypts only the IP payload (e.g., TCP/UDP segment). Original IP header remains. Used for end-to-end security between hosts.
    *   **Tunnel Mode:** Encapsulates the entire original IP packet (header and payload) and adds a new outer IP header. Used for network-to-network or host-to-network VPNs.
*   **Negotiation (via IKE):**
    *   Peers authenticate each other.
    *   Peers negotiate encryption/authentication algorithms and exchange cryptographic keys.

**What Is IKE? (Internet Key Exchange):**
*   Negotiates a tunnel's private keys, authentication methods, and encryption algorithms.
*   **Phases:**
    *   **Phase 1:** Establishes a secure, authenticated channel (IKE SA) between peers.
    *   **Phase 2:** Uses the IKE SA to negotiate IPsec SAs for protecting actual data traffic.
*   **Versions:**
    *   **IKEv1 (Legacy, Wider Adoption):** More complex, distinct Main and Aggressive modes.
    *   **IKEv2 (Newer, Simpler Operation):** Single exchange mode, more robust, native NAT-T.

**IKEv1 vs. IKEv2 Comparison:**
| Feature                   | IKEv1                                                                 | IKEv2                                                                      |
|---------------------------|-----------------------------------------------------------------------|----------------------------------------------------------------------------|
| **Exchange Modes**        | Main (6 messages), Aggressive (3 messages)                            | Single exchange procedure (4 messages for one child SA)                      |
| **Authentication Methods**| Symmetric: PSK, Certificates. Extended Auth (XAuth).                  | Asymmetric support: PSK, Certificates. EAP (pass-through on FortiOS).        |
| **NAT-T**                 | Supported as an extension.                                            | Native support.                                                            |
| **Reliability**           | Unreliable (messages not acknowledged).                               | Reliable (messages acknowledged).                                          |
| **Dial-up Peer ID**       | Peer ID + Aggressive mode for PSK; Peer ID + Main mode for Certificates. | Peer ID or Fortinet Network ID attribute.                                  |
| **Traffic Selector Narrowing**| Not supported.                                                        | Supported (responder can choose a subset of initiator's proposed traffic). |

**Negotiation—Security Association (SA):**
*   A bundle of algorithms and parameters used to secure communication.
*   **IKE SA (Phase 1 SA):** Bidirectional. Secures the control channel for IKE negotiations.
*   **IPsec SA (Phase 2 SA):** Unidirectional. Two SAs are created for each Phase 2 negotiation (one inbound, one outbound) to secure the data channel (ESP).
*   SAs have lifetimes and must be renegotiated upon expiry.

**VPN Topologies:**
*   **Remote Access (Dial-up VPN):**
    *   Remote users (e.g., FortiClient) connect to a central FortiGate (dial-up server).
    *   FortiGate acts as a dial-up server; only clients can initiate the VPN.
    *   Client IP address is usually dynamic.
*   **Site-to-Site:**
    *   Connects two or more networks (e.g., branch office to HQ).
    *   **Simple:** Point-to-point connection between two FortiGates.
    *   **Hub-and-Spoke:** Multiple spoke sites connect to a central hub FortiGate. Spoke-to-spoke traffic usually traverses the hub.
    *   **Full Mesh:** Every site connects directly to every other site. High redundancy, low latency, but complex to configure and manage.
    *   **Partial Mesh:** Some sites connect directly, others via a hub.

---

### **Module 2: IPsec Configuration**

**IPsec Wizard (`VPN > IPsec Wizard`):**
*   Simplifies IPsec tunnel creation by using pre-configured templates.
*   **Templates:** Site to Site, Hub-and-Spoke, Remote Access (Dialup - User, Dialup - Device).
*   **Custom Template:** Bypasses wizard steps and goes directly to manual phase1/phase2 configuration.
*   Wizard asks for key information (remote gateway, auth method, interfaces, subnets) and creates phase1, phase2, address objects, routes, and firewall policies.
*   Shows a network diagram based on input and a summary of created objects.
*   Good for initial setup, can be manually adjusted later.

**Phase 1—Overview:**
*   Establishes a secure IKE SA (control channel).
*   **Key Steps:**
    1.  Authenticate peers (PSK, Digital Signatures, XAuth).
    2.  Negotiate IKE SA parameters (encryption, hashing, DH group, lifetime).
    3.  Perform Diffie-Hellman (DH) exchange to generate shared secret keys.

**Phase 1—Network Configuration:**
*   **IP Version:** IPv4 or IPv6 for the outer tunnel headers.
*   **Remote Gateway:**
    *   **Static IP Address:** Peer has a fixed IP.
    *   **Dialup User:** Local FortiGate is a server; remote peer (client) has a dynamic IP.
    *   **Dynamic DNS:** Peer has a dynamic IP but uses a DDNS service.
*   **Interface:** Local FortiGate interface for the tunnel endpoint (usually WAN).
*   **Local Gateway:** (Optional) Specify source IP for IKE packets if the local interface has multiple IPs.
*   **Mode Config:** (Dial-up server only) Enable to assign virtual IPs/DNS to clients (like DHCP over IKE).
*   **NAT Traversal:** `Enable` (detect NAT), `Forced` (always use UDP 4500), `Disable`.
*   **Keepalive Frequency:** For NAT-T, interval for sending keepalive probes.
*   **Dead Peer Detection (DPD):** `On Demand` (probes if traffic sent but no reply), `On Idle` (probes if no traffic), `Disable`.
*   **Advanced Network Options:** Add Route, Auto Discovery Sender/Receiver (ADVPN), Exchange Interface IP, Device Creation, Aggregate Member.

**Phase 1—Authentication Configuration:**
*   **Method:**
    *   **Pre-shared Key (PSK):** Both peers use the same secret key.
    *   **Signature (Digital Certificates):** Each peer uses a local certificate and validates the peer's certificate against a trusted CA.
*   **IKE Version:** 1 or 2.
*   **Mode (IKEv1 only):**
    *   **Main (ID protection):** More secure, 6 packets. ID payload is encrypted.
    *   **Aggressive:** Faster, 3 packets. ID payload is not encrypted. Required for dial-up with PSK and peer ID.
*   **Peer ID Options (for Aggressive mode or certificate authentication):** Specify how the remote peer identifies itself (e.g., FQDN, email, IP address).

**Phase 1—Phase 1 Proposal Configuration:**
*   Defines acceptable encryption/authentication algorithms, DH groups, and key lifetime for the IKE SA.
*   **Encryption:** AES (128, 192, 256), DES, 3DES, ChaCha20Poly1305.
*   **Authentication (Hashing):** MD5, SHA1, SHA256, SHA384, SHA512.
*   **Diffie-Hellman (DH) Group:** Determines strength of key exchange (e.g., 1, 2, 5, 14-32). Higher is more secure but slower. At least one must be selected.
*   **Key Lifetime (seconds):** How long the IKE SA is valid (default 86400s / 24 hours).
*   **Local ID:** (Optional) Local peer identifier sent to the remote peer.
*   Multiple proposals can be configured; peers negotiate a common set.

**Phase 1—Extended Authentication (XAuth):**
*   Adds an additional layer of username/password authentication on top of PSK or certificates (IKEv1 only).
*   **FortiGate as XAuth Server (for Dialup User remote gateway):**
    *   **Type:** PAP, CHAP, Auto Server (FortiGate detects client's method).
    *   **User Group:** `Choose` (select a specific FortiGate user group) or `Inherit from policy`.
*   **FortiGate as XAuth Client (for Static IP/Dynamic DNS remote gateway):**
    *   **Type:** Client.
    *   Provide username and password to authenticate to the remote XAuth server.

**Phase 2—How it Works:**
*   Negotiates two unidirectional IPsec SAs (one inbound, one outbound) for ESP.
*   Protected by the Phase 1 IKE SA.
*   Renegotiated when SAs are about to expire.
*   **Perfect Forward Secrecy (PFS):** If enabled, uses DH to generate new, independent keys for each Phase 2 SA renegotiation, so a compromised Phase 1 key doesn't compromise past Phase 2 keys.
*   One Phase 1 SA can have multiple Phase 2 SAs (e.g., for different subnets with different security requirements).

**Phase 2—Phase 2 Selectors (Proxy IDs / Quick Mode Selectors):**
*   Define the "interesting traffic" to be encrypted by this IPsec SA.
*   **Parameters:**
    *   **Local Address / Remote Address:** IP subnets, ranges, or single IPs. `0.0.0.0/0` for all.
    *   **Protocol (Advanced):** Specific protocol number (e.g., TCP=6, UDP=17, ICMP=1) or All.
    *   **Local Port / Remote Port (Advanced):** Specific ports or All.
*   In point-to-point VPNs, selectors must be mirrored (local on one side is remote on the other).
*   If traffic doesn't match any Phase 2 selector after passing firewall policy, it's dropped.

**Phase 2—Phase 2 Proposal Configuration:**
*   Defines encryption/authentication algorithms and key lifetime for the IPsec SAs.
*   **Encryption:** AES, DES, 3DES, NULL (no encryption), ChaCha20Poly1305.
*   **Authentication (Hashing):** MD5, SHA1, SHA256, SHA384, SHA512, NULL (no integrity).
*   **Enable Replay Detection:** (Default enabled) Protects against ESP replay attacks. Local setting.
*   **Perfect Forward Secrecy (PFS):** Enable and select DH group(s).
*   **Key Lifetime:**
    *   **Seconds:** Time-based (default 1800s for wizard, 43200s for manual).
    *   **Kilobytes:** Volume-based.
    *   **Both:** Whichever expires first.
    *   Thresholds do not have to match between peers; lowest common value is used.
*   **Auto-negotiate:** (Default disabled) If enabled, FortiGate proactively brings up Phase 2 SAs before current ones expire and even if no interesting traffic.
*   **Autokey Keep Alive:** (Default disabled, implicitly enabled if Auto-negotiate is on) Sends keepalives to prevent inactive tunnels from being torn down by intermediate devices.

**IPsec Hardware Offloading:**
*   Many FortiGate models have NP (Network Processor) units that can offload IPsec encryption/decryption, improving performance.
*   Supported algorithms vary by NP model (check `docs.fortinet.com`).
*   Enabled by default for supported algorithms.
*   Can be disabled per phase1-interface if needed: `config vpn ipsec phase1-interface edit <name> set npu-offload disable end`.

---

### **Module 3: Routing and Firewall Policies**

**Route-Based IPsec VPNs:**
*   **Preferred method on FortiGate.**
*   A virtual tunnel interface is automatically created for each IPsec VPN (named after the phase1-interface).
*   VPN matching and traffic steering are based on standard routing.
*   **Benefits:** Simpler operation/configuration, supports redundancy, L2TP-over-IPsec, GRE-over-IPsec, dynamic routing protocols over IPsec.
*   **Policy-Based VPNs:** Legacy, VPN matching based on policy, not recommended for new deployments.

**Routes for IPsec VPNs:**
*   **Dial-up Server:**
    *   `set add-route enable` (default in phase1): FortiGate automatically adds a static route for the network(s) presented by the dial-up client during Phase 2 negotiation. Default distance 15. Route removed if Phase 2 goes down.
    *   `set add-route disable`: No automatic routes. Used when dynamic routing protocols (e.g., BGP, OSPF) run over the VPN to exchange routes.
*   **Static IP Address / Dynamic DNS Remote Gateway:**
    *   Static routes must be manually configured.
    *   The virtual IPsec tunnel interface is selected as the outgoing interface for the static route.
*   A valid route through the IPsec tunnel interface is required for traffic to be steered into the tunnel.

**Firewall Policies for IPsec VPNs:**
*   At least one firewall policy accepting traffic on the IPsec tunnel is needed for the tunnel to come up and pass traffic (even if Phase 1 and Phase 2 are up).
*   **Typically two policies per tunnel for bidirectional traffic initiation:**
    1.  **Outgoing:** `Incoming interface: internal_LAN`, `Outgoing interface: ipsec_tunnel_interface`, Source: local_subnet, Destination: remote_subnet.
    2.  **Incoming:** `Incoming interface: ipsec_tunnel_interface`, `Outgoing interface: internal_LAN`, Source: remote_subnet, Destination: local_subnet.
*   Security profiles can be applied to these policies to inspect VPN traffic.

**Redundant VPNs:**
*   Provide failover if a primary VPN tunnel fails.
*   **Configuration:**
    1.  Two (or more) IPsec tunnels configured to the same remote peer, typically over different WAN links or to different peer IPs.
    2.  Enable DPD on both ends for faster failure detection.
    3.  Configure static routes for each VPN path:
        *   The route for the primary VPN path must have a lower administrative distance or priority than the route for the backup VPN path.
        *   Alternatively, use dynamic routing protocols to manage path selection and failover.
    4.  Configure firewall policies for traffic over both primary and backup IPsec interfaces.
*   **Types:**
    *   **Partially Redundant:** One peer (e.g., hub) has multiple WAN links/tunnels; other peer (spoke) has one.
    *   **Fully Redundant:** Both peers have multiple WAN links/tunnels to each other.

---

### **Module 4: Monitoring**

**IPsec Monitor Widget (Dashboard > Network > IPsec):**
*   Displays status of IPsec VPN tunnels (Phase 1 and Phase 2 selectors).
*   Shows name, Phase 1 status, Phase 2 selector status (up/down), data sent/received.
*   **Actions:** Right-click to bring up/down entire tunnel or specific Phase 2 selectors, view details.
*   VPN is "Up" if at least one Phase 2 selector is up.

**Monitor IPsec Routes:**
*   Routes for IPsec tunnels appear in the routing table (`Dashboard > Network > Static & Dynamic Routing` or `get router info routing-table all`).
*   **Static IP / Dynamic DNS Gateways:** Route appears after Phase 1 comes up (if auto-negotiate for phase 2 is enabled and successful, or if interesting traffic brings up phase 2).
*   **Dial-up Server (`add-route enable`):** Route appears after Phase 2 comes up.

**IPsec Logs (`Log & Report > System Events > VPN Events`):**
*   Track progress of Phase 1 and Phase 2 negotiations.
*   Report tunnel up/down events, DPD failures, negotiation errors.
*   Double-click a log entry for more details (e.g., specific error messages, peer IPs, proposal details).

**CLI Monitoring and Management:**
*   **SA Management (`diagnose vpn tunnel ?`):**
    *   `list`: List all active IPsec SAs (summary).
    *   `list name <tunnel_name>`: Detailed SA info for a specific tunnel (SPIs, encryption/auth keys, lifetimes, DPD status, NATT status, proxy IDs).
    *   `down <tunnel_name>`: Shut down a tunnel.
    *   `up <tunnel_name>`: Activate a tunnel.
    *   `flush`: Flush all SAs or SAs for a specific tunnel.
*   **IKE Gateway List (`diagnose vpn ike gateway list name <phase1_name>`):**
    *   Shows IKE SA details: interface, peer addresses, created time, IKE SA/IPsec SA counts, direction (initiator/responder), proposal, DPD status.
    *   `diagnose vpn ike gateway clear <name>`: Clears a specific IKE SA (Phase 1). Global effect if no name.
*   **IPsec Tunnel Details (`get vpn ipsec tunnel details name <phase1_name>`):**
    *   Shows traffic counters, negotiated selectors, encryption/auth algorithms, keys, MTU, replay status, NPU acceleration status.

**Common IPsec Problems and Solutions:**
*   **Tunnel Not Coming Up:**
    *   **Output:** Error: negotiation failure.
    *   **Cause:** IPsec configuration mismatch (Phase 1 or Phase 2 proposals, PSK, peer IDs, etc.).
    *   **Solution:** Verify P1/P2 settings match on both peers. Use IKE debug (`diag debug app ike -1`).
    *   **Output:** Error: no SA proposal chosen.
    *   **Cause:** Mismatched P1/P2 proposals.
    *   **Solution:** Ensure at least one common proposal set.
*   **Tunnel Unstable:**
    *   **Output:** DPD packet lost.
    *   **Cause:** ISP issue, network instability between peers.
    *   **Solution:** Check internet connection, enable NAT-T if NAT devices are present.
*   **Tunnel Up, No Traffic:**
    *   **Output (Debug Flow):** No matching IPsec selector, drop.
    *   **Cause:** Traffic not matching Phase 2 selectors, or NAT is enabled on the VPN firewall policy when it shouldn't be (for policy-based VPNs, or if traffic should not be NATted).
    *   **Solution:** Verify selectors, disable NAT on policy if applicable.
    *   **Output (Debug Flow):** Routing issue (e.g., no route, wrong next-hop).
    *   **Cause:** Incorrect static routes, dynamic routing problem.
    *   **Solution:** Verify routes, enable NAT-T if needed.

---
