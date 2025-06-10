# Chapter 8: Web Filtering

## 1. Chapter Summary

This chapter provides a detailed exploration of FortiGate's web filtering capabilities, a critical security feature for controlling user access to web content and protecting against web-based threats. It begins by explaining how web filtering activates, initially for unencrypted HTTP traffic by inspecting GET requests and sending real-time queries to FortiGuard to categorize websites. For encrypted HTTPS traffic, the chapter emphasizes the importance of SSL certificate inspection, where FortiGate uses Server Name Indication (SNI) or the certificate's Common Name (CN)/Subject Alternative Name (SAN) to identify and categorize sites without full decryption. The fundamental difference between flow-based and proxy-based inspection modes for web filtering is then detailed. Flow-based inspection, the default, offers faster scanning with fewer processing resources but limited features, while proxy-based inspection provides more thorough analysis, a wider range of features like advanced overrides and quotas, but consumes more resources and can introduce latency.

The core of the chapter focuses on configuring web filter profiles. This includes enabling FortiGuard Category Based Filtering, which uses a live, contract-dependent service to classify websites into numerous categories. For each category, administrators can define actions such as Allow, Monitor, Block, Warn, Authenticate, or apply Quotas (time or traffic-based, proxy-mode only). The chapter explains each action, including the user experience for "Warn" (user can proceed after a warning) and "Authenticate" (user must provide credentials to override). The ability to customize replacement messages for blocked or warned pages is also noted. Beyond FortiGuard categories, static URL Filtering is presented as a method to explicitly allow (exempt from further security checks), block, or monitor specific URLs or patterns (Simple, Regex, Wildcard), which are evaluated before FortiGuard categories. The chapter also covers Web Rating Overrides, allowing administrators to manually change the FortiGuard category for a specific URL if it's deemed miscategorized.

Finally, the chapter touches upon troubleshooting common web filtering issues. This includes verifying the FortiGuard connection, as category filtering relies on a live service and valid license. Techniques like disabling FortiGuard anycast to use alternative ports for FortiGuard communication and enabling web filter cache to reduce FortiGuard requests are discussed. Common misconfigurations, such as not applying the correct SSL inspection profile for HTTPS traffic or mismatched inspection modes between the firewall policy and the web filter profile, are highlighted as frequent causes of web filtering not working as expected. Monitoring web filter logs (`Log & Report > Security Events > Web Filter`) is crucial for confirming filter actions and troubleshooting.

## 2. Chapter Takeaways

### **Module 1: Inspection Modes**

**When Does Web Filtering Activate?:**
*   **Unencrypted HTTP Traffic:**
    *   FortiGate inspects the HTTP GET request.
    *   Sends a real-time query with URL/hostname to FortiGuard Distribution Network (FDN) for categorization.
    *   Applies policy action based on the returned category.
*   **Encrypted HTTPS Traffic (SSL Certificate Inspection):**
    *   FortiGate inspects the initial unencrypted SSL/TLS handshake.
    *   Primarily uses **Server Name Indication (SNI)** from the Client Hello message to get the FQDN.
    *   If SNI is not present, it uses the Common Name (CN) or Subject Alternative Name (SAN) from the server's certificate.
    *   The FQDN is sent to FortiGuard for categorization.
    *   Full payload decryption is **not** performed in SSL Certificate Inspection mode. Deep SSL inspection is needed for full content analysis but certificate inspection is sufficient for basic category-based web filtering.

**Web Filtering Inspection Modes:**
*   A firewall policy must include both an SSL inspection profile and a web filter profile for web filtering to be effective on HTTPS traffic.
*   **Flow-based Inspection:**
    *   Default inspection mode for web filtering.
    *   Inspects packets as they flow, without full buffering.
    *   Faster response time, fewer processing resources.
    *   Limited features (e.g., no quotas, fewer override options).
    *   Safe search and YouTube/Vimeo restrictions are supported (FortiOS 7.4.4+).
*   **Proxy-based Inspection:**
    *   FortiGate acts as a transparent proxy, buffering and examining web content more thoroughly.
    *   Supports more advanced features: quotas, more granular overrides, custom block pages for more protocols.
    *   More resource-intensive, can introduce latency.
    *   Generally not supported on FortiGate models with 2GB RAM or less.

---

### **Module 2: Web Filtering Basics**

**Configure SSL Certificate Inspection (for Web Filtering):**
*   To effectively filter HTTPS websites based on category, at least SSL Certificate Inspection is required.
*   Create/edit an SSL/SSH Inspection profile (`Security Profiles > SSL/SSH Inspection`).
*   Select "Multiple Clients Connecting to Multiple Servers".
*   Choose "SSL Certificate Inspection" as the inspection method.
*   Define action for "Server certificate SNI check" (Enable, Strict, Disable).
    *   **Enable:** Uses SNI. If SNI domain doesn't match CN/SAN, may use CN/SAN (behavior can vary).
    *   **Strict:** Uses SNI. If SNI domain doesn't match CN/SAN, connection is closed.
    *   **Disable:** Ignores SNI, relies on CN/SAN from certificate.
*   Apply this SSL/SSH Inspection profile to the firewall policy along with the Web Filter profile.

**HTTPS Inspection Order (Web Filtering Context):**
1.  **Static URL Filter:** Checked first. If a URL matches an "Exempt" action, all further web filtering and some other security inspections are bypassed.
2.  **FortiGuard Category Filter:** If no match in static URL filter, FortiGuard categorizes the site.
3.  **Advanced Filters (within Web Filter profile):** E.g., Safe Search, removing ActiveX. Applied if traffic is allowed by the category filter.

**Configure Web Filter Profiles—Flow-Based:**
*   Navigate to `Security Profiles > Web Filter`.
*   Set "Feature set" to "Flow-based".
*   **Key Options:**
    *   **FortiGuard Category Based Filter:** Enable to use FortiGuard categories.
    *   **Safe Search:** Enforce safe search on supported search engines.
    *   **Static URL Filter:** Define custom URL allow/block lists.
    *   **Rating Options:**
        *   `Allow websites when a rating error occurs`: If FortiGuard is unreachable or license issue.
        *   `Rate URLs by domain and IP address`: Sends both domain and IP to FortiGuard for rating; uses rating weight if a mismatch occurs.

**Configure Web Filter Profiles—Proxy-Based:**
*   Set "Feature set" to "Proxy-based".
*   **Additional Options (compared to flow-based):**
    *   More granular search engine keyword filtering.
    *   Stricter safe search enforcement.
    *   Ability to block/allow specific HTTP POST actions.
    *   Remove ActiveX, Java Applets, Cookies.
    *   Quotas for category access.
    *   More comprehensive replacement messages.

**FortiGuard Category Filter:**
*   Live service requiring an active FortiGuard Web Filtering license.
*   Websites are categorized by FortiGuard. FortiGate queries FDN for ratings.
*   **Actions per category:**
    *   **Allow:** Permits access, no log by default.
    *   **Monitor:** Permits access and logs the event.
    *   **Block:** Denies access, logs the event, displays a block page.
    *   **Warning:** Displays a warning page; user can choose to proceed. Logs both warning and passthrough if user proceeds. Customizable warning interval.
    *   **Authenticate:** Prompts user for credentials (from defined user groups) to override the block. Logs block attempt and passthrough if authenticated. Customizable authentication interval.
    *   **Quota:** (Proxy-mode only) Allows access for a specific duration or traffic volume per day. Applies to Monitor, Warning, and Authenticate actions.
*   FortiManager can act as a local FortiGuard server for ratings.

**Web Rating Override:**
*   Located under `Security Profiles > Web Rating Overrides`.
*   Allows an administrator to manually change the FortiGuard category for a specific URL.
*   This changes the *category*, not the *action* for that category (the action is still defined in the Web Filter profile).
*   Useful if a site is believed to be miscategorized by FortiGuard.

**Configure a URL Filter (Static URL Filter):**
*   Part of the Web Filter profile.
*   Entries are checked top-to-bottom before FortiGuard category lookup.
*   **Pattern Types:**
    *   **Simple:** Exact match or leading part (e.g., `example.com/path`).
    *   **Regular Expression (Regex):** Powerful pattern matching.
    *   **Wildcard:** Uses `*` as a wildcard (e.g., `*.example.com`, `example.com/*`).
*   **Actions:**
    *   **Exempt:** Bypasses ALL further web filtering and many other security inspections (AV, DLP, etc.). Use with extreme caution.
    *   **Block:** Denies access.
    *   **Allow:** Permits access, but traffic is still subject to FortiGuard category filtering and other security profiles.
    *   **Monitor:** Permits access and logs it. Traffic is still subject to other filters.

---

### **Module 3: Troubleshooting**

**Troubleshooting the FortiGuard Connection:**
*   Web filtering (category-based) requires a live connection to FortiGuard and a valid license.
*   **Verify Connection:**
    *   CLI: `diagnose debug rating` - shows server list, RTT, flags, TZ, request counts, current/total lost packets. Weight decreases with successful packets.
*   **Common Connection Issues & Fixes:**
    *   **HTTPS Port 443 Blocked:** FortiGuard often uses HTTPS/443. Ensure it's allowed outbound.
    *   **FortiGuard Anycast Issues:** By default, `fortiguard-anycast` is enabled, resolving to a single IP.
        *   CLI: `config system fortiguard set fortiguard-anycast disable end` - allows FortiGate to use other FDN servers and potentially different ports (UDP 443/53/8888, HTTPS 53/8888) based on `set protocol` and `set port`.
    *   **High Request Volume:**
        *   Enable Web Filter Cache: `config webfilter fortiguard set cache-mode ttl set cache-ttl <seconds> end` (GUI: `System > FortiGuard`). Default TTL 60 minutes. Reduces queries for frequently accessed sites.

**Troubleshooting Web Filtering Not Working:**
*   **SSL Inspection:** For HTTPS sites, ensure an appropriate SSL Inspection profile (at least 'certificate-inspection') is applied to the same firewall policy as the Web Filter profile.
*   **Policy & Profile Application:**
    *   Verify the correct Web Filter profile is applied to the correct firewall policy.
    *   Ensure the firewall policy is actually matching the traffic in question.
*   **Inspection Mode Mismatch:** If the firewall policy is in flow-mode but the applied Web Filter profile is configured for proxy-only features, those features won't work. Ensure consistency or use flow-compatible settings.
*   **URL Filter Order:** Remember static URL filters are processed first. An "Allow" or "Exempt" here might bypass desired category blocks.
*   **Browser Cache/DNS Cache:** Clear browser cache and flush DNS cache on the client machine to ensure fresh lookups.

**Web Filter Logs:**
*   Located under `Log & Report > Security Events > Web Filter`.
*   **Key Information:** Timestamp, source IP, destination URL, action taken (allowed, blocked, warned, etc.), category, Web Filter profile used, Policy ID.
*   Raw log data can be downloaded for detailed analysis.
*   Crucial for verifying web filter behavior and identifying misconfigurations or why a site was blocked/allowed.

---
