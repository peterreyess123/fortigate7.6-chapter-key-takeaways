# Chapter 10: SSL VPN

## 1. Chapter Summary

This chapter covers Secure Sockets Layer (SSL) Virtual Private Networks (VPNs), a method for providing secure remote access to private network resources. It begins by outlining the two primary SSL VPN deployment modes on FortiGate: tunnel mode and web mode. Tunnel mode, the focus of this chapter, provides full network layer access for all IP-based applications and requires the FortiClient VPN client (or FortiClient ZTNA agent) to establish a virtual network adapter (`fortissl`) on the client host. Web mode, in contrast, requires only a web browser and provides access to a limited set of predefined services (like HTTP/S, RDP, SMB) through a web portal. The chapter clarifies that for FortiGate models with 2GB RAM or less, SSL VPN web and tunnel mode features might be removed or restricted.

The configuration of SSL VPN in tunnel mode is detailed step-by-step. This includes setting up user accounts and groups for authentication (supporting local users, LDAP, RADIUS, TACACS+, and 2FA), configuring SSL VPN portals to define user access profiles and tunnel mode settings (like split tunneling and source IP pools), and adjusting global SSL VPN settings. Global settings involve specifying the listening interface and port (default 443/HTTPS), the server certificate used by the portal, idle timeouts, and DNS/WINS server settings for VPN clients. A crucial part of the setup is creating authentication/portal mappings, which assign specific user groups to particular SSL VPN portals, allowing for granular access control. The chapter then explains the necessity of firewall policies to allow traffic from the SSL VPN virtual interface (e.g., `ssl.root`) to internal network segments and, optionally, to the internet if split tunneling is disabled or if all client traffic needs to be inspected.

Further, the chapter discusses advanced SSL VPN topics. Split tunneling is explained as a mechanism to route only traffic destined for the private network through the VPN tunnel, while other traffic (like general internet browsing) uses the client's local gateway, conserving bandwidth. The configuration of FortiGate itself as an SSL VPN client (for site-to-site SSL VPNs using a tunnel interface) is also covered, outlining its advantages for bypassing intermediate device issues and its disadvantages, such as requiring a CA certificate on the server. Finally, monitoring and troubleshooting aspects are addressed, including viewing connected SSL VPN users via the GUI dashboard widget, interpreting SSL VPN logs (VPN events and user events), managing SSL VPN idle timeouts versus firewall authentication session timeouts, and using various CLI diagnostic commands for troubleshooting connectivity and session preservation in multi-WAN setups.

## 2. Chapter Takeaways

### **Module 1: Tunnel Mode Deployment**

**SSL VPN Deployment Modes:**
*   Provides secure remote access to private network resources.
*   **Two main modes:**
    *   **Tunnel Mode:**
        *   Accessed via FortiClient (or FortiClient ZTNA agent).
        *   FortiClient creates a virtual network adapter (typically named `fortissl`) on the client host.
        *   Assigns a virtual IP address to the client from a pool.
        *   All IP-based application traffic from the client can be routed through the tunnel (SSL/TLS encapsulated).
        *   Provides full network layer access.
    *   **Web Mode:**
        *   Accessed via a web browser through an SSL VPN portal.
        *   Supports a limited number of protocols/services (HTTP/S, RDP, SMB/CIFS, SSH, Telnet, VNC, Ping) via bookmarks or quick connections in the portal.
        *   No VPN client software installation required on the endpoint (generally).
*   **FortiGate Model Limitation:** Models with 2GB RAM or less may have SSL VPN web and tunnel mode features removed or restricted from GUI/CLI.

**Tunnel Mode Operation:**
1.  Remote user connects to the FortiGate SSL VPN gateway using FortiClient.
2.  User authenticates (credentials, certificate, 2FA).
3.  FortiGate establishes the SSL/TLS tunnel.
4.  FortiGate assigns a virtual IP address to the FortiClient's `fortissl` adapter from a predefined IP pool. This IP becomes the source IP for traffic from the client through the tunnel.
5.  User can now access resources on the private network as permitted by firewall policies.
*   All traffic through the tunnel is encapsulated in SSL/TLS.

**Tunnel Mode—Split Tunneling:**
*   Determines which traffic is routed through the SSL VPN tunnel.
*   **Split Tunneling Disabled:**
    *   All traffic from the client (including internet-bound traffic) is routed through the SSL VPN tunnel to the FortiGate.
    *   FortiGate becomes the default gateway for the client.
    *   Allows for inspection and security enforcement on all client traffic by the FortiGate.
    *   Requires an egress firewall policy on FortiGate for clients to access the internet.
    *   Increases bandwidth usage and latency for internet traffic.
*   **Split Tunneling Enabled:**
    *   Only traffic destined for specific private networks (defined in the SSL VPN portal settings or by routing) is routed through the tunnel.
    *   Internet traffic uses the client's local gateway directly (unencrypted, not inspected by FortiGate).
    *   Conserves bandwidth on the FortiGate and reduces latency for internet access.
    *   No traffic inspection by FortiGate for non-tunneled traffic.

---

### **Module 2: Configuring SSL VPNs**

**Configuration Steps Overview (User as Client - Tunnel Mode):**
1.  **User Accounts/Groups:** Set up local users/groups or configure remote authentication (LDAP, RADIUS, SAML) for SSL VPN users.
2.  **SSL VPN Portals:** Define access profiles, including tunnel mode settings (split tunneling, IP pools, allowed subnets).
3.  **SSL VPN Settings (Global):** Configure listening interface, port, server certificate, DNS, and authentication/portal mapping.
4.  **Firewall Policies:** Create policies to allow traffic from the SSL VPN interface (e.g., `ssl.root`) to internal networks and optionally to the internet.
5.  **(Optional) Egress Policy:** If split tunneling is disabled, a policy from `ssl.root` to WAN is needed for internet access.

**Configure the SSL VPN Portal (`VPN > SSL-VPN Portals`):**
*   Defines the resources and features available to users connecting via this portal.
*   **Tunnel Mode Settings within a Portal:**
    *   **Enable Tunnel Mode:** Activates tunnel mode access for this portal.
    *   **IP Pools:** Assign a pre-configured IP Pool object from which client virtual IPs will be assigned. Default pool `SSLVPN_TUNNEL_ADDR1`.
    *   **Split Tunneling:**
        *   `Disabled`: All traffic through tunnel.
        *   `Enabled Based on Policy Destination`: Client routes traffic based on destinations specified in the SSL VPN firewall policy.
        *   `Enabled for Trusted Destination`: Client routes traffic based on "Routing Address Overrides" configured in the portal.
    *   **Routing Address Overrides (for "Enabled for Trusted Destination"):** Specify destination subnets that should be routed through the tunnel.
*   Multiple portals can be created for different user groups with varying access levels.

**Configure SSL VPN Settings (Global - `VPN > SSL-VPN Settings`):**
*   **Listen on Interface(s):** Select the FortiGate interface(s) that will accept SSL VPN connections.
*   **Listen on Port:** Default is `443` (HTTPS). Can be changed to avoid conflicts with HTTPS admin access on the same interface.
*   **Redirect HTTP to SSL-VPN:** If enabled, HTTP requests to the SSL VPN port are redirected to HTTPS.
*   **Server Certificate:** Certificate presented by FortiGate to SSL VPN clients (default `Fortinet_Factory`). Best practice: use a publicly trusted or enterprise CA-signed certificate.
*   **Restrict Access:** Limit connections to specific hosts/subnets (source IPs).
*   **Idle Logout:** Timeout for inactive SSL VPN sessions (default 300 seconds).
*   **DNS Server / WINS Server:** Provide DNS/WINS servers to SSL VPN clients.
*   **Authentication/Portal Mapping:**
    *   Crucial for assigning users/groups to specific SSL VPN portals.
    *   Define mappings for "All Other Users/Groups" as a catch-all.
    *   Rules are evaluated top-down.

**Firewall Policies to and From SSL VPN Interface:**
*   SSL VPN traffic uses a virtual interface, typically `ssl.<vdom_name>` (e.g., `ssl.root` if no VDOMs).
*   **Inbound Policy (Client to Internal Network):**
    *   **Incoming Interface:** `ssl.root` (or relevant `ssl.<vdom>`).
    *   **Outgoing Interface:** Internal network interface(s) (e.g., `port1`, `LAN_zone`).
    *   **Source:** SSL VPN user group(s) and/or the SSL VPN IP Pool address object(s).
    *   **Destination:** Internal network resources/subnets.
    *   **Service:** `ALL` or specific services.
    *   **Action:** `ACCEPT`.
    *   Apply security profiles as needed.
*   **Outbound Policy (Client to Internet - if split tunneling disabled):**
    *   **Incoming Interface:** `ssl.root`.
    *   **Outgoing Interface:** WAN interface(s).
    *   **Source:** SSL VPN user group(s) and/or SSL VPN IP Pool.
    *   **Destination:** `all`.
    *   **Service:** `ALL`.
    *   **Action:** `ACCEPT`.
    *   **NAT:** Must be enabled.
    *   Apply security profiles.

**Tunnel Mode—FortiGate as Client (Site-to-Site SSL VPN):**
*   One FortiGate acts as an SSL VPN client, another as the SSL VPN server.
*   Useful for connecting sites, especially if IPsec is problematic due to intermediate devices.
*   **Client FortiGate Configuration:**
    1.  **PKI User:** Create a PKI user with a CA certificate that can verify the server's certificate. Subject CN might need to match server cert.
    2.  **SSL-VPN Tunnel Interface:** Create a new interface of type "SSL-VPN".
        *   Specify remote server IP/port, source IP (optional), and the PKI user for authentication.
    3.  **Static Route:** Route traffic destined for remote subnets via the SSL-VPN tunnel interface.
    4.  **Firewall Policies:** Allow traffic from internal interface to the SSL-VPN tunnel interface.
*   **Server FortiGate Configuration:** Similar to standard SSL VPN server setup, but the "user" will be the client FortiGate's PKI identity.

---

### **Module 3: Monitoring**

**Monitoring SSL VPN Sessions (GUI):**
*   **Dashboard Widget:** `Dashboard > Network > SSL-VPN` (or add widget).
    *   Shows currently connected SSL VPN users.
    *   Displays username, connection time, source IP (public IP of client), virtual IP (from IP Pool for tunnel mode), connection mode (tunnel/web).
*   **Actions:**
    *   Right-click a user to `End Session` (disconnect).
    *   View connection details.

**SSL VPN Logs:**
*   Located under `Log & Report > Events`.
*   **VPN Events:**
    *   Show SSL VPN tunnel establishment (tunnel-up) and termination (tunnel-down) events.
    *   Includes information like tunnel name, remote IP, tunnel IP, duration.
*   **User Events:**
    *   Show authentication actions related to SSL VPN users (auth-login, auth-logout, auth-failed).
    *   Includes username, action, message.

**SSL VPN Idle Timeout vs. Authentication Session Timeout:**
*   **SSL VPN Idle Timeout:** (`VPN > SSL-VPN Settings > Idle Logout`)
    *   Specific to the SSL VPN session itself.
    *   If no traffic from the client over the SSL VPN tunnel for this duration, the SSL VPN connection is terminated. Default 300 seconds.
*   **Firewall Authentication Session Timeout:** (`config user setting set auth-timeout ...`)
    *   General timeout for any firewall-authenticated user.
    *   SSL VPN authentication is distinct and **not** subject to this firewall authentication timeout setting. Its idle state is managed by the SSL VPN `idle-timeout`.
*   When an SSL VPN session ends (by user logout or idle timeout), associated firewall sessions are typically flushed to prevent reuse by a different user.

**SSL VPN Timers (CLI - `config vpn ssl settings`):**
*   Helpful for high latency connections to prevent premature logouts during negotiation.
*   `set login-timeout <10-180>`: Max time for login process (default 30s).
*   `set dtls-hello-timeout <10-60>`: Max DTLS hello timeout (default 10s). DTLS is UDP-based SSL, often used for tunnel mode if available.
*   `set http-request-header-timeout <1-60>` / `set http-request-body-timeout <1-60>`: Mitigate DoS attacks like Slowloris by setting timeouts for receiving HTTP request components.

**SSL VPN—Session Preservation (Multi-WAN):**
*   **CLI Command:** `config system interface edit <interface_name> set preserve-session-route enable end`
*   In multi-WAN setups, if a route changes causing traffic to egress a different WAN interface, existing SSL VPN sessions might drop if not re-evaluated through the original path.
*   `preserve-session-route` attempts to keep existing sessions on their original interface even if routing tables change.
*   Useful if one WAN link is dedicated or preferred for SSL VPNs.

**Best Practices for Common SSL VPN Issues:**
*   **FortiClient Version:** Ensure compatibility with FortiOS firmware (check Release Notes).
*   **Split Tunneling:**
    *   If enabled, ensure routing on client directs appropriate traffic to the tunnel.
    *   If disabled, ensure FortiGate has an egress policy for internet access with NAT.
*   **Port Conflicts:** Ensure SSL VPN listening port (e.g., 443) doesn't conflict with HTTPS admin access on the same interface if both are needed simultaneously (change one of the ports).
*   **Firewall Policies:** Correctly define source (SSL VPN users/IPs), destination, and allow SSL VPN interface (`ssl.root`).
*   **Timeouts:** Adjust `idle-timeout` and other CLI timers appropriately for user experience and security.
*   **DTLS:** Ensure UDP port (often same as TCP SSL VPN port, e.g., UDP/443) is allowed if DTLS is used by FortiClient, as it can improve performance.

**Useful Troubleshooting Commands (CLI):**
*   `diagnose vpn ssl <sub-command>`:
    *   `list`: Show current connections, users, IPs.
    *   `info`: General SSL VPN information.
    *   `statistics`: Memory usage, connection stats.
    *   `tunnel-test`: Enable/disable old tunnel mode IP allocation method (less common now).
*   `diagnose debug application sslvpn -1`: Enable detailed debug messages for SSL VPN daemon.
*   `diagnose debug application fnbamd -1`: Debug for authentication daemon (if auth issues).
*   `diagnose debug console timestamp enable`: Prepend timestamps to debug output.
*   `diagnose debug enable`
*   Check FortiClient logs for client-side issues.

---
