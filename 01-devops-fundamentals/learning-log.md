#25-08-2026
📅 25-08-2026 — DevOps Fundamentals
**📌 Topics Covered**
1. DevOps Fundamentals — overview of DevOps and its purpose
2. SDLC and Agile methodology
3. Development lifecycle vs DevOps lifecycle
4. Infrastructure as Code (IaC)
5. SaaS and PaaS
6. Web Server and Application Server
7. Forward Proxy and Reverse Proxy
8. Overview of DevOps tools
9. Introduction to Multi-Cloud and cloud platforms


📅 26-08-2026 — Networking / OSI / Proxies
- Learned the 7 layers of the OSI model, including MAC, IP, ports and application protocols.
- Understood encapsulation/decapsulation and the flow of data from Application → Physical layer.
- Learned Public vs Private IP addresses and how private devices use NAT to communicate with the internet.
- Learned Forward Proxy vs Reverse Proxy — forward proxy represents/controls clients, while reverse proxy represents/protects backend servers.
- Learned VPN basics — a VPN creates an encrypted tunnel using the existing internet connection and can hide the client's public IP.
- Connected networking concepts with DevOps/AWS, including NAT Gateway, Internet Gateway, Load Balancer and Nginx reverse proxy.
  🔑 Quick Memory: L2 = MAC → L3 = IP → L4 = Port → L7 = HTTP/HTTPS | Forward Proxy = Client | Reverse Proxy = Server


Learning Log — DNS Basics

**Date: 27-Aug-2026**
Topic: DNS, Recursive Resolver, Authoritative DNS, Cache & TLS

What I Learned

Today I understood the basic architecture of DNS and the difference between the major DNS components.

Key Understanding

DNS translates domain names into IP addresses/information required to reach services.

Recursive DNS Resolver receives a DNS request from the client and finds the answer on behalf of the client.

The resolver can query:

Root → TLD → Authoritative DNS

Root DNS tells the resolver where to find the required TLD.

TLD DNS (.com, .org, .in, etc.) tells the resolver which nameservers are authoritative for a domain.

Authoritative DNS is the source of truth for a domain's DNS records.

Route 53 Hosted Zone can provide authoritative DNS for my domain.

Route 53 is NOT the Root DNS.

Google Public DNS (8.8.8.8) is primarily a recursive DNS resolver, not the authoritative DNS for every domain.

DNS cache temporarily stores DNS answers so future queries can be answered faster.

Cache can exist at multiple levels:

Browser → OS → Recursive Resolver

TTL determines how long a DNS answer can be cached.

28-08-2029
Learning Log — 28-08-2026

Topics Covered

Operating System (OS) Basics

Linux File System

Linux Directories

Basic Linux Commands