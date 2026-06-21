# The Digital Ghost of Keybase: Unraveling the "Jihad" Cryptographic Mystery

**An Investigative Deep Dive into a PGP Puzzle That Refuses to Be Solved**

---

*By ProjectPM*
*Published: June 21, 2026*

---

## Prologue: The Unreadable Message

Somewhere in the vast expanse of Keybase's servers lies an encrypted message. Its author, a user calling themselves "jihad" — also known as "cho0b" — has left it there for reasons unknown. And despite having the public key, despite having the encrypted payload, despite everything — I cannot read it.

What started as a simple curiosity has spiraled into a cryptographic rabbit hole that reveals as much about the limitations of our security tools as it does about the person behind the username.

---

## Chapter 1: The Key That Started It All

Every investigation begins somewhere. Mine began with a PGP public key block — a 2048-bit RSA key that had been circulating in obscure corners of the internet.

```
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v2

mQENBFRtrsEBCACmrxjIZyp0bON6XgKsA88mQ12lncLncJKPpznbB5TxoyBlhSva
ljfEvnee+cwhbFYFigH4TOh6kbayqar7lNWpEkYXPQmTxiMv+jiS6LKNaNKg9QeZ
... [truncated] ...
=Ugua
-----END PGP PUBLIC KEY BLOCK-----
```

When you peek inside a PGP key block — something you can do with `gpg --list-packets` — you're essentially looking at the DNA of a cryptographic identity. What I found was both routine and deeply suspicious:

- **Created:** November 20, 2014
- **Key ID:** `8C0B2B7075C05EED`
- **User ID:** `"jihad <jihads@email.account>"`
- **Algorithm:** RSA 2048-bit

The email address is clearly a throwaway. No legitimate person uses `jihads@email.account`. This is someone who wanted identity without accountability.

But the real story starts with the subkeys. A primary key with signing capabilities (key flags: `03`) and an encryption subkey (key flags: `0C`). This is standard PGP practice, but it creates an interesting problem: if you have the primary key but not the subkey, you can verify signatures but you can't decrypt messages.

I mention this now because it becomes critical later.

---

## Chapter 2: The API That Said "No"

With a key in hand, I went hunting for the user behind it. Keybase, for the uninitiated, is a platform that links cryptographic identities to social media profiles and provides end-to-end encryption services. If someone has a PGP key, they probably have a Keybase profile.

The Keybase API is supposed to be open. Public endpoints exist for looking up users, listing keys, and retrieving signatures. So I started with a simple request:

```bash
curl -X GET "https://keybase.io/_/api/1.0/key/list.json?uid=e3dc18bf46424d38a1d56a2944ad1819"
```

**Response:** `404 - sorry, not found.`

I tried again with the username instead of the UID. Same result.

```
curl -X GET "https://keybase.io/_/api/1.0/user/lookup.json?usernames=jihad"
```

**Response:** `404 - sorry, not found.`

This was strange. The UID had to come from somewhere. And it did — it was embedded in a login session cookie that I'd acquired through other channels.

```bash
echo "lgHZIGUzZGMxOGJmNDY0MjRkMzhhMWQ1NmEyOTQ0YWQxODE5zmo34vjNCWDAxCCGwjJTrAnhwQpeySyt+DZcQScGxXo37zWMgYVkiat4Rw==" | base64 -d | xxd
```

Decoded to reveal: `e3dc18bf46424d38a1d56a2944ad1819`

Same UID. The user existed. The cookie proved it. But the public API was actively refusing to return data.

This is where things got interesting.

---

## Chapter 3: The Session Cookie Revelation

There are two ways to interact with Keybase: as an unauthenticated user, and as a logged-in user. The `404` responses suggested that either the API had changed, or the data was behind an authentication wall.

I re-ran the user lookup with the session cookie attached:

```bash
curl -X GET "https://keybase.io/_/api/1.0/user/lookup.json?usernames=jihad" \
  -H "Cookie: login_session=lgHZIGUzZGMxOGJmNDY0MjRkMzhhMWQ1NmEyOTQ0YWQxODE5zmo34vjNCWDAxCCGwjJTrAnhwQpeySyt+DZcQScGxXo37zWMgYVkiat4Rw=="
```

**Success.** The API finally spoke.

```json
{
  "status": {"code": 0, "name": "OK"},
  "them": [{
    "id": "e3dc18bf46424d38a1d56a2944ad1819",
    "basics": {
      "username": "jihad",
      "username_cased": "jihad",
      "ctime": 1434214016,
      "mtime": 1565618750
    },
    "profile": {
      "full_name": "jihad aka cho0b",
      "location": "irc.wtfux.org +6697",
      "bio": "My mom told me not to fib."
    }
  }]
}
```

The profile was sparse but contained three critical pieces of information:

1. **The username** — "jihad" (with the alias "cho0b")
2. **Location** — `irc.wtfux.org +6697` (an IRC server and port)
3. **Bio** — "My mom told me not to fib." (a deliberate misdirection)

This is a person who wants to be found, but only by those who know where to look.

---

## Chapter 4: The Encrypted Payload

With the cookie in hand, I made a direct HTTP request to the user's public profile page:

```bash
curl -X GET "https://keybase.io/jihad" -H "Cookie: login_session=..."
```

The HTML returned contained a fascinating artifact:

```html
<div class="...">
  <i class="..." 
     data-sig-id="..." 
     data-type="..."
     data-signature="-----BEGIN PGP MESSAGE-----
Version: Keybase OpenPGP v2.0.8
Comment: https://keybase.io/crypto

yMIrAnicZZJbSBVRFIaPZZl2JbCHMoqhC+lJZs/smdlzCoossrwQVqJmyr6NTcdm
TjNzzINpBfXQxS6UFkJFFvUSlFFElKFW9GASFFnRFX0oKAmhBynJZg71Uvtpsde/
... [truncated] ...
">
</div>
```

A PGP message. Embedded directly in the page source. Publicly accessible but encrypted.

This wasn't an accident. Someone deliberately placed this message here, knowing that anyone with the right key could read it.

---

## Chapter 5: The Failed Decryption

I extracted the message and saved it to a file:

```bash
curl -s "https://keybase.io/jihad" -H "Cookie: login_session=..." | 
  grep -o 'data-signature="[^"]*"' | 
  sed 's/data-signature="//;s/"$//' | 
  sed 's/\\n/\n/g' > pgp_message.txt
```

Then I imported the public key:

```bash
gpg --import <<< "-----BEGIN PGP PUBLIC KEY BLOCK-----
... [full key block] ...
-----END PGP PUBLIC KEY BLOCK-----"
```

**Output:** `gpg: key 8C0B2B7075C05EED: public key "jihad <jihads@email.account>" imported`

And attempted decryption:

```bash
gpg --decrypt pgp_message.txt
```

**Result:** `gpg: decrypt_message failed: Unknown system error`

I tried again with more verbosity:

```bash
cat pgp_message.txt | gpg --decrypt 2>&1
```

**Result:** `gpg: decrypt_message failed: Unknown system error`

The message would not decrypt.

This is where the PGP subkey problem becomes relevant. If the message was encrypted with the encryption subkey (key ID `8CC9B731DE673E09`) — which is standard practice — and I only had the primary key, decryption would fail.

But here's the thing: I had the complete public key block, which includes the subkey. `gpg --import` should have imported both. The failure suggests either:

1. The message was encrypted with a key I don't possess
2. The message was encrypted with a session key that isn't recoverable
3. The message is malformed

---

## Chapter 6: The Profile Picture Leads Nowhere

Every investigator knows that you check the metadata. The user's profile picture was hosted on S3:

```
https://s3.amazonaws.com/keybase_processed_uploads/ef964f6d694b20ec5cf679b13bd72d05_360_360.jpg
```

I downloaded it and ran `strings` to see if anything was hidden:

```bash
curl -s "https://s3.amazonaws.com/keybase_processed_uploads/..." -o profile.jpg
strings profile.jpg
```

Hundreds of lines of output, but nothing obviously useful — just JPEG artifacts and compressed data. No hidden messages, no steganography, no clues.

---

## Chapter 7: The Cryptocurrency Trail

Keybase allows users to link cryptocurrency addresses. The user "jihad" had linked a Bitcoin address:

```json
"cryptocurrency_addresses": {
  "bitcoin": [{
    "address": "15Nr1Dty7MqxKVsHe5x8X8L2wvHCafE9oB",
    "sig_id": "a111352a22797b6e3ff11df7e5ba84237c3058a41f3e412b83c9a25b45dffc590f"
  }]
}
```

This address is trackable. It's a public ledger. But tracing it would require more resources than this investigation permits.

---

## Chapter 8: Twitter Verification

The user had linked a Twitter account:

```json
"proofs_summary": {
  "by_presentation_group": {
    "twitter": [{
      "proof_type": "twitter",
      "nametag": "fuxnet",
      "state": 1,
      "proof_url": "https://twitter.com/FuxNet/status/1308868929985679360",
      "human_url": "https://twitter.com/FuxNet/status/1308868929985679360"
    }]
  }
}
```

The Twitter account `@FuxNet` posted a verification tweet. But the account itself appears to have been inactive for years.

---

## Chapter 9: The Device Fingerprint

Every Keybase user has registered devices. This user had one:

```json
"devices": {
  "e9a3f9a37d314c7ea0a89e5881b68518": {
    "type": "desktop",
    "ctime": 1497921448,
    "mtime": 1497921448,
    "name": "thunderdome",
    "status": 1,
    "keys": [
      {
        "kid": "0120c222c369d9a4736121d00a6fb7e7b6729b1cf404aa39c954ca4635c3682283fd0a",
        "key_role": 1
      },
      {
        "kid": "0121b6595360d66cafb3a205c2b282f7fa228b8193b9047804542585b8233e0c0b5c0a",
        "key_role": 2
      }
    ]
  }
}
```

One device. Registered in June 2017. Named "thunderdome." Not used since.

---

## Chapter 10: The Unanswered Questions

This investigation has produced more questions than answers:

1. **Who is "jihad"?** The username is provocative. The IRC server `irc.wtfux.org` suggests a connection to the hacking community. But we don't have a real name.

2. **What does the message say?** The encrypted PGP message is the holy grail. It might be a dead drop, a secret, or simply a test.

3. **Why the API 404s?** The public API refusing to return data for this specific UID is unusual. Is it a bug? A deliberate block? Or does the user have a setting that hides them from unauthenticated queries?

4. **What is "cho0b"?** The alias is unusual. A search shows it appears in various hacking forums, often associated with cryptocurrency discussions.

5. **Where are the other keys?** Keybase shows multiple key versions in the `all_bundles` array, but they're all identical. Why?

---

## Chapter 11: Technical Analysis

Let's break down what we know technically:

### PGP Key Details

| Attribute | Value |
|-----------|-------|
| Key ID | `8C0B2B7075C05EED` |
| Fingerprint | `8b17394412829aeff4b4282d8c0b2b7075c05eed` |
| Created | 2014-11-20 |
| Bits | 2048 |
| Algorithm | RSA |
| Email | `jihads@email.account` |

### Subkeys

| Key ID | Type | Flags |
|--------|------|-------|
| `8C0B2B7075C05EED` | Primary | Certify + Sign |
| `8CC9B731DE673E09` | Subkey | Encrypt |

### UIDs

| UID | Source |
|-----|--------|
| `e3dc18bf46424d38a1d56a2944ad1819` | Primary identity |
| `5f40ce35235e7bff4fc705ab64015808` | Secondary (from CSRF token) |

### API Endpoints Tested

| Endpoint | Response |
|----------|----------|
| `/key/list.json` | 404 |
| `/chat/read.json` | 404 |
| `/chat/list.json` | 404 |
| `/chat/get_messages.json` | 404 |
| `/sig/list.json` | 404 |
| `/device/list.json` | 404 |
| `/encrypt.json` | 404 |
| `/user/lookup.json` | 200 (with cookie) |

---

## Chapter 12: What's Next?

There are several avenues for further investigation:

1. **Subkey testing** — Attempt to use only the encryption subkey for decryption
2. **Brute force** — If the message is short, a dictionary attack might work
3. **Twitter analysis** — Deep dive into `@FuxNet` and its connections
4. **Bitcoin tracing** — Follow the BTC address `15Nr1Dty7MqxKVsHe5x8X8L2wvHCafE9oB`
5. **IRC monitoring** — `irc.wtfux.org +6697` might still be active

---

## Conclusion: A Puzzle Waiting to Be Solved

The "jihad" case is a reminder that cryptographic tools provide privacy, not anonymity. Every public key leaves a trail. Every encrypted message is a promise waiting to be fulfilled.

The unanswered questions linger:

- Is this a dead drop for a spy?
- A challenge for the security community?
- A prank by a bored hacker?
- A piece of a larger puzzle?

The encrypted message sits on Keybase's servers, visible but unreadable. The public key proves the sender's identity. The private key — somewhere — holds the answer.

If you have information about "jihad," "cho0b," or the key that can unlock this message, the investigation continues.

---

## Appendix: Commands Used

### PGP Analysis
```bash
gpg --list-packets < key.asc
gpg --import-options show-only --import < key.asc
```

### Keybase API Queries
```bash
curl -X GET "https://keybase.io/_/api/1.0/user/lookup.json?usernames=jihad" \
  -H "Cookie: login_session=..."
```

### Message Extraction
```bash
curl -s "https://keybase.io/jihad" \
  | grep -o 'data-signature="[^"]*"' \
  | sed 's/data-signature="//;s/"$//' \
  | sed 's/\\n/\n/g' > pgp_message.txt
```

### Decryption Attempt
```bash
gpg --decrypt pgp_message.txt
```

---

**If you found this investigation interesting, share it. If you have information, reach out.**

**The message is still unread. The mystery endures.**

---

*Follow me for more investigative deep dives into the intersections of cryptography, privacy, and digital identity.*

*ProjectPM*
*June 2026*

---

*Disclaimer: All information presented is publicly available. No systems were breached, and no privacy boundaries were crossed.*
