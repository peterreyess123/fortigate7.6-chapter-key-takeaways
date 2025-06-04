# Chapter 6: Certificate Operations

## 1. Chapter Summary

This chapter focuses on digital certificates and their critical role in FortiGate operations, particularly for authentication, ensuring data privacy, and enabling traffic inspection. It begins by defining a digital certificate as a digital identity produced and signed by a Certificate Authority (CA), analogous to a passport, and outlines its primary purposes: authentication, encryption/decryption, and integrity. The chapter explains how FortiGate utilizes the Subject and Subject Alternative Name (SAN) fields within an X.509v3 certificate to identify devices and people, and how it checks for revocation, CA possession, validity dates, and digital signature integrity before trusting a certificate. The process of digital signature verification, involving hash functions and asymmetric cryptography (public/private keys), is detailed to illustrate how FortiGate ensures both authenticity and integrity. The use of SSL/TLS for privacy is also discussed, covering how symmetric and asymmetric cryptography work together in an SSL handshake to establish secure sessions.

The chapter then delves into practical applications and configurations related to inspecting encrypted data. It contrasts scenarios with no SSL inspection, where malware can pass undetected, with SSL certificate inspection (superficial, checking SNI/CN for web filtering without decryption) and full SSL/deep inspection (man-in-the-middle, where FortiGate decrypts, inspects, and re-encrypts traffic). The necessity of full SSL inspection for protecting against threats in HTTPS, SMTPS, POP3S, IMAPS, and FTPS is emphasized. Configuration of SSL/SSH inspection profiles is covered, including predefined profiles (certificate-inspection, deep-inspection) and custom profiles for more granular control over inbound or outbound traffic, CA certificate selection (defaulting to `Fortinet_CA_SSL`), and actions for various certificate statuses (e.g., untrusted, expired, revoked). The chapter addresses how to handle Encrypted Client Hello (ECH) TLS connections, which FortiGate strips during deep inspection, and how to exempt specific sites from SSL inspection due to technical issues (like HSTS or certificate pinning) or legal/privacy concerns, using web categories or specific addresses.

Finally, the chapter covers managing certificates on FortiGate and client endpoints. It explains how FortiGate handles invalid certificates (Keep Untrusted & Allow, Block, Trust & Allow) and the implications of different settings for untrusted SSL certificates. The requirement for the CA certificate used in full SSL inspection (e.g., `Fortinet_CA_SSL`) to be trusted by client devices to avoid browser warnings is a key point, leading to instructions on installing this CA certificate on endpoints. The chapter details the specific requirements for a CA certificate used by FortiGate for full SSL inspection (cA=True, keyUsage=keyCertSign). It also discusses the impact of SSL inspection on applications beyond browsers (like Outlook 365, Dropbox) and how to handle issues arising from HPKP (HTTP Public Key Pinning) or certificate transparency requirements by exempting sites or using SSL certificate inspection. Procedures for downloading private CA certificates from FortiGate, importing them into client browsers (e.g., Firefox), and importing company-owned CAs or local certificates onto FortiGate are provided. Managing Certificate Revocation Lists (CRLs) and understanding the FortiGate certificate store (local CAs, local certificates, remote CAs, CRLs) concludes the chapter, providing a comprehensive view of certificate lifecycle management on FortiGate.

## 2. Chapter Takeaways

### **Module 1: Authenticate and Secure Data Using Certificates**

**Digital Certificate Basics:**
*   **Definition:** A digital identity (X.509v3 standard) produced and signed by a Certificate Authority (CA).
*   **Analogy:** Passport or driver's license for the digital world.
*   **Primary Purposes:**
    *   **Authentication:** Verifies the identity of a device, server, or user.
    *   **Encryption/Decryption:** Secures data in transit using public/private key pairs.
    *   **Integrity:** Ensures data has not been tampered with, often via digital signatures.
*   **Identification Fields:**
    *   **Subject:** Identifies the entity the certificate belongs to (e.g., `CN=www.example.com`).
    *   **Subject Alternative Name (SAN):** Allows multiple identifiers (DNS names, IP addresses, email addresses).
    *   FortiGate uses Subject Key Identifier and Authority Key Identifier to relate issuer and certificate.

**How FortiGate Trusts Certificates:**
*   FortiGate performs several checks before trusting a certificate:
    1.  **Revocation Check:** Verifies against Certificate Revocation Lists (CRLs) or Online Certificate Status Protocol (OCSP) responders to ensure the certificate hasn't been revoked.
    2.  **CA Certificate Possession:** Uses the `Issuer` field to determine if FortiGate possesses the corresponding CA certificate in its store. Without the issuing CA's certificate, the presented certificate cannot be trusted.
    3.  **Validity Dates:** Checks if the current date is within the certificate's `Valid From` and `Valid To` dates.
    4.  **Digital Signature Validation:** Verifies the integrity and authenticity of the certificate by checking its digital signature using the issuer CA's public key.

**FortiGate Verifies a Digital Signature (Process):**
1.  CA signs the original hash of the certificate content with its private key (this is the digital signature).
2.  FortiGate receives the certificate and the digital signature.
3.  FortiGate calculates a fresh hash of the received certificate content using the same hashing algorithm specified in the certificate.
4.  FortiGate uses the CA's public key (from its trusted CA store) to verify the digital signature, thereby decrypting/revealing the original hash.
5.  FortiGate compares the fresh hash with the original hash. If they match, integrity is confirmed, and the signature is valid.
    *   Terms "sign" and "verify" are more accurate than "encrypt" and "decrypt" for digital signatures, as the purpose is identity and integrity, not just confidentiality.

**FortiGate Uses SSL/TLS for Privacy:**
*   SSL (Secure Sockets Layer) / TLS (Transport Layer Security) establishes secure, encrypted sessions.
*   **Key SSL/TLS Features used by FortiGate:**
    *   **Privacy of data:** Encrypts data in transit.
    *   **Identifies one or both parties:** Uses certificates for authentication.
    *   **Uses symmetric and asymmetric cryptography:**
        *   **Symmetric Cryptography:** Same key for encryption and decryption. Faster. Key exchange must be secure. Used for bulk data encryption.
        *   **Asymmetric Cryptography (Public Key Cryptography):** Uses a public key (shared) and a private key (kept secret). Slower, more resource-intensive. Used for key exchange (e.g., exchanging symmetric session keys) and digital signatures.

---

### **Module 2: Inspect Encrypted Data**

**Encrypted Traffic With No SSL Inspection:**
*   Malware and threats cloaked by encryption can bypass network defenses if SSL inspection is not enabled.
*   Even if a site has a legitimate CA-signed certificate, the site itself could be compromised and serve malicious content.

**SSL Inspection Modes:**
*   **SSL Certificate Inspection (Superficial Inspection):**
    *   FortiGate inspects the initial SSL/TLS handshake (Client Hello for SNI, Server Hello for certificate).
    *   Relies on Server Name Indication (SNI) extension in TLS or the Common Name (CN)/SAN fields in the server's certificate to identify the FQDN.
    *   Does **not** decrypt the traffic payload.
    *   Primarily used for Web Filtering and Application Control based on domain/URL.
*   **Full SSL Inspection (Deep Inspection / Man-in-the-Middle):**
    *   FortiGate acts as a proxy, terminating the SSL session from the client and initiating a new SSL session to the server.
    *   Maintains two separate SSL sessions: client-to-FortiGate and FortiGate-to-server.
    *   FortiGate decrypts, inspects the full payload for threats (AV, IPS, DLP etc.), and then re-encrypts the traffic using its own certificate (typically `Fortinet_CA_SSL` or a custom CA cert).
    *   Protects against threats in HTTPS and other SSL-encrypted protocols (SMTPS, POP3S, IMAPS, FTPS).

**Inbound vs. Outbound SSL/SSH Inspection:**
*   **Outbound:** Protecting internal users accessing external SSL/SSH services (e.g., users browsing HTTPS websites). FortiGate impersonates the *server* to the client.
*   **Inbound:** Protecting internal company servers from external SSL/SSH connections (e.g., protecting a company web server). FortiGate acts as a reverse proxy, using the server's actual certificate (or one issued for it) to impersonate the *client* to the server (less common for full inspection, more for WAF or load balancing with SSL offloading). Typically, for protecting servers, FortiGate presents the server's certificate to external users.

**SSL Inspection Profile Configuration:**
*   Located under `Security Profiles > SSL/SSH Inspection`.
*   **Predefined Profiles:**
    *   `no-inspection` (default for new firewall policies).
    *   `certificate-inspection` (for outbound certificate inspection).
    *   `deep-inspection` (for outbound full SSL inspection).
*   **Customizable Profile (`custom-deep-inspection` or new user-defined):**
    *   Allows configuring inspection for "Multiple Clients Connecting to Multiple Servers" (outbound) or "Protecting SSL Server" (inbound).
    *   **CA Certificate:** Selects the CA certificate FortiGate uses to re-encrypt traffic in deep inspection (default `Fortinet_CA_SSL`).
    *   **Action on Invalid Certificates:** Define actions (Allow, Block, Ignore) for various certificate issues (untrusted, expired, revoked, etc.).
*   Applied to a firewall policy to enable SSL inspection for matching traffic.

**Controlling ECH (Encrypted Client Hello) TLS Connections:**
*   ECH encrypts the initial "Client Hello" to enhance privacy by hiding the SNI.
*   When FortiGate performs deep inspection, it currently strips the ECH extension, forcing a non-ECH TLS connection to inspect the SNI.
*   DNS filters can also strip ECH information from DNS over HTTPS (DoH) responses.
*   FortiGate can be configured to block or allow connections that attempt to use ECH.

**Exempting Sites From SSL Inspection:**
*   Necessary for sites with:
    *   **HSTS (HTTP Strict Transport Security):** Browsers may refuse to proceed if the certificate changes (as it does with FortiGate deep inspection).
    *   **Certificate Pinning (HPKP - HTTP Public Key Pinning):** Applications expect a specific server certificate or CA.
    *   **Legal/Privacy Reasons:** E.g., financial or healthcare sites.
*   **Exemption Methods (in SSL/SSH Inspection profile):**
    *   **Reputable websites:** Allowlist maintained by FortiGuard.
    *   **Web Categories:** Exempt specific categories (e.g., Finance and Banking).
    *   **Addresses:** Exempt specific FQDNs, IP addresses, or address ranges (wildcards like `*.example.com` are supported).

**Invalid Certificates Handling:**
*   FortiGate can detect various invalid certificate issues (expired, revoked, validation timeout, validation failed).
*   **Actions for Invalid Certificates (in SSL/SSH Inspection profile):**
    *   **Keep Untrusted & Allow:** Allows connection, presents certificate as untrusted to the browser (browser warning).
    *   **Block:** Blocks the connection.
    *   **Trust & Allow:** Allows connection, presents certificate as trusted (re-signed by FortiGate's trusted CA).
    *   **Custom:** Allows per-reason action configuration.
*   **Untrusted SSL Certificates Setting (for deep inspection):**
    *   **Allow:** Sends a temporary certificate signed by `Fortinet_CA_Untrusted` (user sees warning).
    *   **Block:** Blocks connection to sites with untrusted certs.
    *   **Ignore:** Re-signs with `Fortinet_CA_SSL` regardless of original server cert trust status (user sees no warning if `Fortinet_CA_SSL` is trusted).

**FortiGate Self-Signed CA Certificates for Deep Inspection:**
*   By default, FortiGate uses `Fortinet_CA_SSL` (a self-signed CA certificate) for re-encrypting traffic during deep inspection.
*   This certificate is **not** trusted by client browsers/OS by default, leading to certificate warnings.
*   **To avoid warnings:**
    1.  **Install `Fortinet_CA_SSL` on all client devices/browsers as a trusted root CA.** This is the most common method.
    2.  Install a company CA certificate (signed by your internal enterprise CA, which clients already trust) on FortiGate and use it for SSL full inspection.

**Full SSL Inspection—Certificate Requirements for FortiGate's CA:**
*   The CA certificate used by FortiGate to re-sign server certificates must have specific extensions:
    *   `cA=True` (Basic Constraints: Is CA)
    *   `keyUsage=keyCertSign` (Key Usage: Certificate Signing)
*   The preloaded `Fortinet_CA_SSL` meets these requirements.
*   The root CA that signed FortiGate's inspection CA must be imported into client machines.

**Applications and SSL Inspection:**
*   SSL inspection impacts not just browsers but any application using SSL/TLS (e.g., Outlook 365, Dropbox, FTPS clients).
*   Solutions depend on application security design:
    *   Import FortiGate's CA into the OS/application certificate store.
    *   Exempt application domains/servers from SSL inspection.
*   Certificate pinning by applications will often break with deep inspection, requiring exemption.

**HPKP (HTTP Public Key Pinning) and Certificate Transparency:**
*   **HPKP:** Security feature where a site tells browsers to only trust specific public keys for a period, can conflict with deep inspection. (Largely deprecated in favor of Certificate Transparency).
*   **Certificate Transparency (CT):** Requires CAs to log issued certificates to public logs.
*   Workarounds for sites with these requirements: Exempt from full SSL inspection or use SSL certificate inspection only.

**Applying an SSL Inspection Profile to a Firewall Policy:**
*   SSL inspection profiles are applied within a firewall policy.
*   The policy must also have other security profiles (e.g., Antivirus, Web Filter) for inspection to occur on the decrypted content.
*   `no-inspection` profile means no SSL/SSH traffic inspection.
*   **Decrypted Traffic Mirror:** Option to send a copy of decrypted SSL traffic to a specified interface for external analysis (e.g., by a DLP system).

**Certificate Warnings During Full SSL Inspection:**
*   Occur because browsers don't trust the FortiGate's re-signing CA (`Fortinet_CA_SSL`) by default.
*   **Solutions:**
    1.  Distribute and install `Fortinet_CA_SSL` as a trusted root CA on all client endpoints.
    2.  Use an SSL certificate issued by your own private CA (that clients already trust) on the FortiGate for re-signing.

**FortiGate GUI HTTPS Server Certificate:**
*   By default, FortiGate uses a `self-sign` certificate for its HTTPS admin GUI, triggering browser warnings.
*   **Options to avoid warnings:**
    1.  Accept the warning on first connection (browser usually saves exception).
    2.  Change GUI server certificate to `Fortinet_GUI_Server` (signed by `Fortinet_CA_SSL`) and import `Fortinet_CA_SSL` into admin browsers.
    3.  Use a certificate signed by a recognized public CA or your internal enterprise CA.

**Certificate Management on FortiGate:**
*   **Downloading Private CA Certificates:** `Fortinet_CA_SSL` can be downloaded from `System > Certificates` as a `.cer` file for distribution to clients.
*   **Importing Private CA Certificates into Endpoints:** Process varies by OS and browser (e.g., Firefox has its own store). Import as a trusted root CA.
*   **Importing CA Certificates onto FortiGate:** (`System > Certificates > Create/Import > CA Certificate`). For trusting external CAs or establishing CA chains. Can import from file or SCEP.
*   **Importing Local Certificates onto FortiGate:** (`System > Certificates > Create/Import > Certificate`). For GUI, SSL VPN, etc.
    *   **Options:** After CSR request, Certificate and Key file, PKCS#12 file (`.pfx`, `.p12`).
*   **Certificate Revocation Lists (CRLs):**
    *   Lists of revoked certificates. FortiGate can import CRLs via file or online (HTTP, LDAP, SCEP).
    *   CRL section in `System > Certificates` visible after at least one CRL is loaded.
*   **FortiGate Certificate Store (`System > Certificates`):**
    *   Central location for all certificates and CRLs.
    *   **Sections:** Loaded CRLs, Local CA Certificates (for signing), Local Certificates (server/device certs), Remote CA Certificates (trusted third-party CAs).
    *   **Source Column:** Indicates if factory default or user-imported.

---
