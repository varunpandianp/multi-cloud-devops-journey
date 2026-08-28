# DNS — Quick Notes

## What is DNS?

**DNS (Domain Name System)** translates human-readable domain names into information computers can use, commonly IP addresses.

```text
www.example.com
       ↓ DNS
203.0.113.10
```

> DNS finds the destination; it does not establish the application connection.

---

## DNS Main Components

### 1. Recursive DNS Resolver

A resolver receives a DNS query from a client and finds the answer on the client's behalf.

```text
Client
  ↓
Recursive Resolver
  ↓
Root
  ↓
TLD
  ↓
Authoritative DNS
  ↓
Answer
```

Examples:

- Google Public DNS → `8.8.8.8`
- Cloudflare DNS → `1.1.1.1`
- AWS VPC DNS Resolver

### Why do we need a recursive resolver?

Without it, every client would need to understand and query the complete DNS hierarchy.

The resolver:

- Finds the answer
- Follows DNS referrals
- Caches DNS responses
- Handles retries/timeouts
- Returns the final answer to the client

---

## 2. Root DNS

Root DNS is the top of the public DNS hierarchy.

It does **not normally provide the final IP address**.

It tells the resolver where to find the appropriate TLD.

Example:

```text
Resolver → Root
Root → "Ask the .com TLD servers"
```

There are **13 logical root server identities**, but many physical/anycast instances exist worldwide.

---

## 3. TLD DNS

TLD = **Top-Level Domain**

Examples:

```text
.com
.org
.net
.in
.uk
```

The TLD server tells the resolver which authoritative nameservers are responsible for a specific domain.

Example:

```text
Resolver → .com TLD
.com → "Ask the authoritative nameservers for example.com"
```

### Important

> TLD tells you **WHO is authoritative** for a domain.

---

## 4. Authoritative DNS

Authoritative DNS is the **source of truth for a domain's DNS records**.

Example:

```text
example.com

www     A       203.0.113.10
api     A       203.0.113.20
mail    MX      mail.example.com
```

The authoritative DNS server provides the actual DNS records.

Examples:

- AWS Route 53
- Cloudflare DNS
- Google Cloud DNS
- Azure DNS
- BIND

### Important

> Authoritative DNS tells you **WHAT the DNS records are**.

---

# DNS Hierarchy

```text
Root
 ↓
TLD (.com)
 ↓
example.com
 ↓
Authoritative DNS
 ↓
DNS Records
```

Full lookup:

```text
Client
  ↓
Recursive Resolver
  ↓
Root
  ↓
TLD
  ↓
Authoritative DNS
  ↓
DNS Record
  ↓
IP Address
```

---

# Route 53

**Amazon Route 53 is not the Root DNS.**

A Route 53 hosted zone can act as the **authoritative DNS** for your domain.

Example:

```text
Root
 ↓
.com
 ↓
mycompany.com
 ↓
Route 53
 ↓
DNS Records
```

Route 53 can manage records such as:

```text
www.mycompany.com
api.mycompany.com
mail.mycompany.com
```

AWS also provides a **VPC DNS Resolver** for DNS resolution from resources inside a VPC.

So don't confuse:

```text
Route 53 Hosted Zone
        ↓
Authoritative DNS
```

with:

```text
AWS VPC DNS Resolver
        ↓
Recursive/caching DNS resolution
```

---

# DNS Cache

A DNS cache stores previously resolved DNS answers temporarily.

Example:

```text
www.example.com
→ 203.0.113.10
→ TTL = 300 seconds
```

The recursive resolver can cache this answer for the TTL period.

Next request:

```text
Client
 ↓
Recursive Resolver
 ↓
Cache HIT
 ↓
203.0.113.10
```

This avoids querying the DNS hierarchy repeatedly.

### Cache ≠ Database

A cache is generally a **temporary copy kept for faster access**.

It may be stored in:

- RAM
- Local disk
- Application memory
- Dedicated caching systems such as Redis

---

# Multiple DNS Caches

DNS information can be cached at different layers:

```text
Browser
 ↓
OS DNS Cache
 ↓
Recursive Resolver Cache
 ↓
Authoritative DNS
```

Therefore, DNS changes may not appear immediately because an old answer may still be cached until its TTL expires.

---

# DNS vs TLS vs TCP vs HTTP

These technologies have different jobs:

```text
DNS
 ↓
Find the destination

TCP
 ↓
Establish network connection

TLS
 ↓
Encrypt/authenticate the connection

HTTP/HTTPS
 ↓
Exchange application data
```

Example:

```text
https://api.example.com
        ↓
       DNS
        ↓
      IP/ALB
        ↓
       TCP
        ↓
       TLS
        ↓
      HTTPS
        ↓
   Application
```

---

# Key Definitions

| Component | Main job |
|---|---|
| Recursive Resolver | Finds DNS answers for clients |
| Root DNS | Directs queries to the correct TLD |
| TLD DNS | Identifies authoritative nameservers |
| Authoritative DNS | Provides official DNS records |
| DNS Cache | Temporarily stores DNS answers |
| Route 53 Hosted Zone | Can provide authoritative DNS |
| AWS VPC DNS Resolver | Provides DNS resolution inside VPCs |
| TLS | Secures communication |
| TCP | Establishes reliable network connection |
| HTTP/HTTPS | Application communication |

## ⭐ Most Important Memory Trick

```text
Recursive Resolver
→ "I'll find the answer for you."

Root
→ "Go to this TLD."

TLD
→ "These servers are authoritative for this domain."

Authoritative DNS
→ "Here is the actual DNS record."

Cache
→ "I already know this answer, so I'll return it faster."
```

## DevOps Mental Model

```text
User
 ↓
Browser
 ↓
DNS Resolver
 ↓
Root
 ↓
TLD
 ↓
Authoritative DNS (e.g. Route 53)
 ↓
IP / ALB
 ↓
TCP
 ↓
TLS
 ↓
HTTPS
 ↓
Application
```