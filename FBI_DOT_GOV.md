**Subject:** Security Vulnerability Report – Information Disclosure via Publicly Accessible Portal Configuration Endpoint  

**Date:** March 22, 2026  
**Affected URL:** `https://atlasbeta.fbi.gov/portal/sharing/portals/self`  
**Affected System:** ArcGIS Enterprise (version 11.3.0) – Beta Environment  

---

### 1. Description  
The endpoint `/portal/sharing/portals/self` on the `atlasbeta.fbi.gov` domain returns detailed portal configuration information. This data includes internal service URLs, software version details, group queries, helper service endpoints, and other sensitive operational settings. The endpoint is accessible without any authentication, exposing information that should be restricted to authorized personnel only.

---

### 2. Vulnerability Details  
When accessed, the endpoint returns a JSON‑formatted response containing:

- **Software version:** `enterpriseVersion: "11.3.0"` and `currentVersion: "2024.1"`.  
- **Internal service URLs:** Full paths to internal ArcGIS Server services, such as:
  - Geocoding: `https://atlasbeta.fbi.gov/wwgc/rest/services/World/GeocodeServer`
  - Routing: `https://atlasbeta.fbi.gov/gptools/rest/services/Routing/...`
  - Printing: `https://atlasbeta.fbi.gov/hosted2/rest/services/Utilities/PrintingTools/GPServer/...`
  - Raster/imagery services: `https://atlasbeta.fbi.gov/himage/...`
- **Helper services configurations:** details of all external and internal helper services used by the portal.  
- **Group queries:** specific queries used to populate galleries and content (e.g., `"title:"Esri 2D Styles" AND owner:esri_webstyles"`).  
- **Authentication settings:** SAML enabled, OAuth supported, and the `access` property set to `private` (though the endpoint itself is public).  
- **Custom CSS/HTML snippets:** content from the `rotatorPanels` field containing embedded style definitions referencing internal file paths (`https://atlas.fbi.gov/files/...`).

The presence of this information provides an attacker with a detailed map of the internal architecture, software versions, and available service endpoints. No authentication or special privileges are required to obtain this data.

---

### 3. Steps to Reproduce  
1. Open a web browser or use a tool like `curl`.  
2. Navigate to `https://atlasbeta.fbi.gov/portal/sharing/portals/self`.  
3. Observe the returned configuration data, which is rendered as plain text or JSON.

**Example request (using curl):**  
```bash
curl https://atlasbeta.fbi.gov/portal/sharing/portals/self
```

**Snippet of the response (sanitized):**  
```json
{
  "enterpriseVersion": "11.3.0",
  "helperServices": {
    "geocode": [
      {
        "url": "https://atlasbeta.fbi.gov/wwgc/rest/services/World/GeocodeServer",
        "name": "World"
      }
    ],
    "printTask": {
      "url": "https://atlasbeta.fbi.gov/hosted2/rest/services/Utilities/PrintingTools/GPServer/Export%20Web%20Map%20Task"
    }
  },
  "portalMode": "singletenant",
  "access": "private",
  ...
}
```

---

### 4. Impact  
- **Information Disclosure:** The exposed configuration details can be used by an adversary to:
  - Identify the exact software version, which may have known vulnerabilities (CVE’s) that can be exploited.
  - Discover internal service endpoints, facilitating further reconnaissance and targeted attacks.
  - Understand the portal’s integration with third‑party services (e.g., geocoding, routing) and potentially craft abuse scenarios.
  - Gain insight into the underlying infrastructure (e.g., server naming conventions, paths, hosted services).
- **Lack of Access Control:** The endpoint should be restricted to authenticated users or trusted IP ranges, but it is currently accessible to anyone on the internet.

While the affected environment appears to be a beta site, the information exposed is sensitive and could be leveraged in a more sophisticated attack against the production environment if patterns or dependencies are shared.

---

### 5. Severity Assessment  
**CVSS 3.1 Base Score:** 5.3 (Medium)  
- **Attack Vector:** Network (AV:N)  
- **Attack Complexity:** Low (AC:L)  
- **Privileges Required:** None (PR:N)  
- **User Interaction:** None (UI:N)  
- **Scope:** Unchanged (S:U)  
- **Confidentiality:** Low (C:L) – configuration details are disclosed, but not user data  
- **Integrity:** None (I:N)  
- **Availability:** None (A:N)  

Given the potential to facilitate further attacks, the risk is considered **Medium**. If the information could lead directly to a compromise of the system or user accounts, the severity would be higher.

---

### 6. Recommendations  
1. **Restrict Access:**  
   Configure the portal to require authentication for the `/sharing/portals/self` endpoint. This can be done through ArcGIS Enterprise’s role‑based access control or by implementing IP‑based restrictions.

2. **Review Other Configuration Endpoints:**  
   Audit similar endpoints (e.g., `/portal/sharing/rest/info`, `/portal/sharing/rest/portals/self`) to ensure they are not publicly exposed.

3. **Apply Principle of Least Privilege:**  
   Ensure that only authenticated administrators or trusted internal services can retrieve portal configuration details.

4. **Version Obscurity:**  
   While not a substitute for proper access control, consider whether version information can be suppressed from public endpoints to reduce the attack surface.

5. **Monitor Logs:**  
   Check access logs for any suspicious requests to this endpoint to determine if it has already been exploited.

---

### 7. Conclusion  
The endpoint `https://atlasbeta.fbi.gov/portal/sharing/portals/self` is currently exposing sensitive configuration details of the ArcGIS Enterprise portal without authentication. This information could be used by an attacker to gather intelligence for a targeted attack. Immediate action should be taken to restrict access to this endpoint.

Thank you for your attention to this matter. I am available to provide further details or assist in validating the fix.

---  
*Please contact me if you require any additional information or have questions.*
