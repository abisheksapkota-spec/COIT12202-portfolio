# COIT12202 – Network Security Concepts

This repository is my personal learning log for **COIT12202: Network Security Concepts**, Term 2 2026. I'm using it to document the weekly hands-on activities and labs as I work through the unit — building and hardening network services in GNS3, and picking up practical skills in areas like PKI, password security, SSH hardening, and firewalls.

The goal isn't just to submit work, but to keep a running record of what I've built, what I learned, and evidence (screenshots, configs, packet captures) that I can look back on.

---

## What this unit covers

Broadly, the topics I'm working through include:

- **Network fundamentals in GNS3** — virtual topologies, interfaces, loopback addressing, connecting virtual and physical nodes
- **PKI & HTTPS** — building a certificate authority, issuing certificates, setting up and verifying HTTPS, analysing TLS handshakes
- **Password security** — comparing hashing algorithms (MD5, SHA-512, yescrypt), configuring password-quality and account-lockout policies, understanding cracking resistance
- **SSH hardening** — key-based authentication, hardening `sshd_config`, blocking brute-force attempts with fail2ban, tunnelling through a bastion host
- **Kerberos authentication** — ticket-based authentication, KDC/AS/TGS roles, mutual authentication over an untrusted network
- **Firewalls & further topics** — to be added as the unit progresses

---

## Repository structure

```
.
├── Week1.md                              # GNS3 basics: interfaces, loopback, virtual vs physical nodes
│
├── OpenSSL-CA-12312491-ePortfolio/       # PKI & HTTPS
│   ├── OpenSSL-CA-12312491-Lab-Outputs.md
│   ├── steps-for-solution.md
│   ├── Activities-Evidence.md
│   ├── OpenSSL-CA-12312491.gns3project
│   └── images/
│
├── Password-Hashing-Lab-Solution/        # Password Security
│   ├── Password-Hashing-12312491.md
│   ├── Activities-Evidence.md
│   ├── Password-Hashing-12312491.gns3project
│   └── images/
│
├── SSH-Hardening-12312491-ePortfolio/    # SSH Hardening
│   ├── SSH-Hardening-12312491-Lab-Outputs.md
│   ├── Activities-Evidence.md
│   ├── SSH-Hardening-12312491.gns3project
│   ├── SSH-Hardening-12312491-admin.pcap
│   ├── SSH-Hardening-12312491-internal.pcap
│   └── images/
│
└── Auth-Kerberos-Week5/                  # Kerberos Authentication
    └── (write-up, screenshots, GNS3 project)
```

Each folder is self-contained: write-up, screenshots, and the GNS3 project file for that topic.

---

## Weekly activity log

- **Week 1:** [`Week1.md`](./Week1.md) — GNS3 basics
- **Week 2:** [`Activities-Evidence.md`](./OpenSSL-CA-12312491-ePortfolio/Activities-Evidence.md) — PKI & HTTPS
- **Week 3:** [`Activities-Evidence.md`](./Password-Hashing-Lab-Solution/Activities-Evidence.md) — Password security
- **Week 4:** [`Activities-Evidence.md`](./SSH-Hardening-12312491-ePortfolio/Activities-Evidence.md) — SSH hardening
- **Week 5:** [`Auth-Kerberos-Week5/`](./Auth-Kerberos-Week5/) — Kerberos authentication
- *(more weeks to be added as the term progresses)*

---

## Notes

- All labs are built and tested in **GNS3**; each service folder includes the `.gns3project` file so a topology can be reopened and reproduced.
- This is a working repo — content will keep growing week to week through Term 2 2026.

---

**Abishek Sapkota**
