# Chapter 4: Firewall Authentication

## 1. Chapter Summary

This chapter explores the critical role of firewall authentication in enhancing network security beyond simple IP address or device-type verification. It begins by establishing the need for authenticating users and user groups to reliably control access to network resources and track individual activity across multiple devices. The chapter then outlines the primary methods of firewall authentication supported by FortiGate: local password authentication (credentials stored on FortiGate), server-based or remote password authentication (credentials verified by external servers like POP3, RADIUS, LDAP, or TACACS+), and two-factor authentication (requiring something you know, like a password, and something you have, like a token or certificate).

A significant portion of the chapter details the configuration and operation of remote authentication servers, focusing primarily on LDAP and RADIUS. For LDAP, it explains the hierarchical structure (DC, OU, CN, UID), the concept of Distinguished Names (DN) for binding, and the importance of secure connections using LDAPS. It covers configuring an LDAP server on FortiGate, including specifying the server IP, port, Common Name Identifier (e.g., `sAMAccountName`), Distinguished Name, and bind type (Simple, Anonymous, Regular). For RADIUS, the chapter describes it as a standard AAA protocol, detailing the client-server interaction (Access-Request, Access-Accept/Reject/Challenge) and the configuration steps on FortiGate, such as primary server IP/name, shared secret, and authentication method (PAP, CHAP, MSCHAP). Testing connectivity and user credentials for both LDAP and RADIUS using CLI diagnostic commands is also highlighted.

The chapter then shifts to different methods and behaviors of authentication. It elaborates on two-factor authentication, particularly using FortiTokens (hardware or mobile) which generate One-Time Passwords (OTPs). The algorithm behind OTP generation (seed + time) and the process of assigning FortiTokens to users are explained. A crucial distinction is made between active authentication, where users are prompted to enter credentials (used by local, server-based, and 2FA methods), and passive authentication, where user identity is determined transparently (primarily discussed in the FSSO chapter but introduced here). The integration of authentication into firewall policies is detailed, explaining how user and user group objects are added to the source field of a policy to trigger authentication. The chapter also specifies the required protocols (HTTP, HTTPS, FTP, Telnet) that must be allowed in the policy for active authentication prompts to appear and the role of DNS pre-authentication. Finally, it covers advanced topics like mixing authenticated and unauthenticated policies, the `auth-on-demand` CLI setting for controlling authentication behavior with fall-through policies, using captive portals for interface-level authentication, managing authentication timeouts, and monitoring authenticated firewall users via the GUI.

## 2. Chapter Takeaways

### **Module 1: Remote Authentication**

**Firewall Authentication (General Concept):**
*   Authenticates users and user groups, providing more reliable access control than IP address or device-type alone.
*   Users must typically enter valid credentials (username/password, token).
*   After validation, FortiGate applies firewall policies and security profiles based on user identity.

**FortiGate Methods of Firewall Authentication:**
*   **Local Password Authentication:**
    *   Usernames and passwords stored directly on the FortiGate.
    *   Suitable for small-scale deployments or standalone FortiGates.
*   **Server-Based Password Authentication (Remote Authentication):**
    *   Credentials stored on external servers.
    *   Protocols: POP3, RADIUS, LDAP, TACACS+.
*   **Two-Factor Authentication (2FA):**
    *   Enabled on top of an existing method (local or remote).
    *   Requires "something you know" (password) AND "something you have" (token like FortiToken, or certificate).

**Local Password Authentication Configuration:**
*   Users created under `User & Authentication > User Definition`.
*   Select "Local User" type.
*   Define username, password, optionally email/SMS for 2FA, and assign to user groups.

**Server-Based Password Authentication (Overview):**
*   Remote server authenticates users; FortiGate forwards credentials for verification.
*   Desirable for multiple FortiGates authenticating the same users or integrating into existing authentication infrastructure.
*   FortiGate typically does not store user information locally (some configurations might cache group info).
*   **Two main ways to integrate remote users:**
    1.  **Create user accounts on FortiGate:** Select remote server type (RADIUS, LDAP) and point to the preconfigured server. Username on FortiGate must match the remote server. Often used to add 2FA to remote users.
    2.  **Add remote authentication server to a User Group:** All users on that server become potential members of the FortiGate group (access controlled by group membership on the remote server, especially for LDAP).

**LDAP (Lightweight Directory Access Protocol):**
*   **Overview:** Application protocol for accessing and maintaining distributed directory information services (e.g., Microsoft Active Directory).
*   **Structure:** Hierarchical tree structure.
    *   `dc`: Domain Component (e.g., `dc=example,dc=com`)
    *   `ou`: Organizational Unit (e.g., `ou=users`)
    *   `cn`: Common Name (e.g., `cn=John Doe` or group name)
    *   `uid`: User ID (e.g., `uid=jdoe`)
*   **Binding:** The process where the LDAP server authenticates the client (FortiGate or user).
*   **Security:** LDAP on port 389 is clear text. **LDAPS** (LDAP over SSL/TLS) on port 636 is secure and highly recommended.
*   **Configuring LDAP Server on FortiGate (`User & Authentication > LDAP Servers`):**
    *   **Server IP/Name & Port:** Address of the LDAP server.
    *   **Common Name Identifier (CNI):** Attribute used to identify the user (e.g., `sAMAccountName`, `uid`, `cn`).
    *   **Distinguished Name (DN):** Base location in the LDAP tree where user objects are searched (e.g., `ou=users,dc=example,dc=com`).
    *   **Bind Type:**
        *   **Anonymous:** No credentials needed to search (rarely used).
        *   **Simple:** Username and password for an account that can search the directory (may or may not be an admin account, depending on LDAP server config).
        *   **Regular:** Credentials of an authorized user (often an admin) are used by FortiGate to perform queries, especially for searching across multiple domains.
    *   **Secure Connection:** Enable LDAPS or STARTTLS and provide CA certificate for server validation.
    *   **Test Connectivity:** Checks if FortiGate can reach the LDAP server.
    *   **Test User Credentials:** Verifies if a specific user can authenticate against the configured LDAP server via FortiGate.

**RADIUS (Remote Authentication Dial-In User Service):**
*   **Overview:** Standard protocol for centralized Authentication, Authorization, and Accounting (AAA).
*   **Interaction:**
    1.  Client (FortiGate) sends `Access-Request` to RADIUS server.
    2.  Server replies with `Access-Accept`, `Access-Reject`, or `Access-Challenge` (for 2FA).
*   **Configuring RADIUS Server on FortiGate (`User & Authentication > RADIUS Servers`):**
    *   **Primary Server IP/Name & Secret:** Address of RADIUS server and the shared secret (must match server configuration).
    *   **Authentication Method:** Default (tries PAP, MSCHAPv2, CHAP), PAP, CHAP, MSCHAP, MSCHAPv2.
    *   **Test Connectivity & Test User Credentials:** Similar to LDAP.
    *   **Include in every User Group:** Option to automatically add all RADIUS-authenticated users to all FortiGate user groups (use with caution).
*   **Vendor-Specific Attributes (VSAs):** Can be used by RADIUS server to return group membership or other Fortinet-specific attributes.

**Testing LDAP and RADIUS Query on the CLI:**
*   `diagnose test authserver ldap <server_name> <username> <password>`
*   `diagnose test authserver radius <server_name> <scheme> <user> <password>`
    *   `<scheme>` can be `pap`, `chap`, `mschap`, `mschap2`.
*   Output shows success/failure and group membership details.

---

### **Module 2: Methods of Authentication**

**Two-Factor Authentication (2FA):**
*   Improves security over static passwords by requiring two independent methods of identification.
*   **Factors:** Something you know (password/PIN) + Something you have (token/certificate).
*   Available for both regular user and administrator accounts on FortiGate.
*   Cannot be used with explicit proxy authentication.

**FortiTokens and OTPs (One-Time Passwords):**
*   **OTP:** Used once, more secure than static passwords.
*   **Delivery Methods:**
    *   **FortiToken Hardware (e.g., FTK200) or FortiToken Mobile (software app):** Generate a 6-digit code (typically every 60 seconds) based on a unique seed and GMT time.
    *   **Email or SMS:** OTP sent to user's configured contact info.
    *   **FortiToken Mobile Push:** User receives a push notification on their mobile app to approve/deny login, no code entry needed.
*   **NTP Server Recommended:** Crucial for time-based OTPs (FortiToken, Email, SMS) to remain synchronized between token/service and FortiGate.
*   **Algorithm:** Seed (unique, static per token) + Time (from accurate clock) -> OTP.

**Assigning a FortiToken to a User:**
1.  Register FortiTokens (hardware serials or mobile activation codes) on FortiGate (`User & Authentication > FortiTokens`).
    *   FortiGate VMs and physical devices usually come with two free FortiToken Mobile licenses.
2.  Edit or create a user account (`User & Authentication > User Definition`).
3.  Enable "Two-factor Authentication".
4.  Select the desired registered FortiToken from the "Token" field.
    *   Optionally, provide email/SMS details if using those delivery methods.

**Active vs. Passive Authentication:**
*   **Active Authentication:**
    *   User receives a login prompt.
    *   User manually enters credentials.
    *   Examples: Local, RADIUS, LDAP, TACACS+, 2FA methods.
*   **Passive Authentication:**
    *   User does not receive a login prompt from FortiGate.
    *   Credentials determined automatically/transparently.
    *   Examples: FSSO (Fortinet Single Sign-On), RSSO (RADIUS Single Sign-On), NTLM. (FSSO is covered in Chapter 5).

**Firewall Policy—Source (for Authentication):**
*   To trigger authentication, a firewall policy must include User or User Group objects in its **Source** field.
*   The source IP address must still match.
*   Any user belonging to the specified group who provides correct credentials (and matches other policy criteria) will be authenticated.
*   **User/Group Object Types:** Local firewall accounts, external server accounts (LDAP/RADIUS users/groups), PKI users, FSSO users.

**Protocols for Active Authentication:**
*   The firewall policy must allow at least one protocol that can display an authentication dialog for active authentication to work.
*   **Supported Protocols for Prompting:** HTTP, HTTPS, FTP, Telnet.
*   **DNS:** Must be allowed in the policy *before* authentication, as hostname resolution is often required for the user to reach a service that triggers the auth prompt. The DNS service itself does not trigger the prompt.
*   Other services are typically blocked until successful authentication via one of the supported prompting protocols.
*   Passive authentication does not require specific service protocols to be allowed for identity determination.

**Mixing Policies (Authenticated and Unauthenticated):**
*   Enabling authentication in policies doesn't always mean users *must* actively authenticate.
*   If an unauthenticated (open) policy matches the traffic *before* an authenticated policy, the user will not be prompted.
*   **To enforce active authentication:**
    1.  Ensure all policies that could match the traffic have authentication enabled.
    2.  Use `auth-on-demand always` (CLI only): Forces authentication prompt even if a fall-through policy exists.
        *   `config user setting set auth-on-demand [always|implicitly]` (`implicitly` is default).
    3.  Enable a captive portal on the ingress interface.
*   If passive authentication (e.g., FSSO) identifies a user, active authentication prompts are generally skipped. Active auth often serves as a backup if passive fails.

**Captive Portal (Interface-Level Authentication):**
*   Enables authentication at the interface level.
*   All users connecting through that interface are prompted for authentication before accessing any resource, unless an exemption matches.
*   Configured under `Network > Interfaces > Edit Interface > Security Mode`.

**Authentication Timeout:**
*   Specifies how long a user remains authenticated or how long they can be idle before re-authentication is required.
*   CLI command: `config user setting set auth-timeout-type [idle-timeout|hard-timeout|new-session] set auth-timeout <seconds> end`.
*   **Types:**
    *   **Idle (Default):** Logs out user if no traffic from their IP for the timeout period (default 5 minutes).
    *   **Hard:** Authentication expires after the timeout period, regardless of activity.
    *   **New Session:** Authentication expires if no *new* sessions are created from the host IP within the timeout period, even if existing sessions are active.

**Monitoring Firewall Users:**
*   GUI: `Dashboard > Assets & Identities > Firewall Users` (or similar path depending on FortiOS version/layout).
*   Displays: Authenticated user, user group, duration, IP address, traffic volume, authentication method.
*   Allows administrators to de-authenticate (disconnect) users.
*   Does not show FortiGate administrators who are logged in directly for management.

---
