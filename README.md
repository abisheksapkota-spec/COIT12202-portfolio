# COIT12202: Hardened Services Portfolio

Individual portfolio for Assessment 1 (COIT12202, Network Security Concepts): three hardened network services built and tested in GNS3, plus evidence for the Week 1 to 4 interactive activities. Each folder below is self-contained; write-up, screenshots, and the GNS3 project file for that service.

The video walkthrough for this portfolio is submitted separately via Echo360, per the assessment instructions, and is not included in this repository.

## Repository structure

```
.
├── Week1.md                              # GNS3 basics: interfaces, loopback, virtual vs physical nodes
│
├── OpenSSL-CA-12312491-ePortfolio/       # Service 1 — PKI & HTTPS
│   ├── OpenSSL-CA-12312491-Lab-Outputs.md
│   ├── steps-for-solution.md
│   ├── Activities-Evidence.md
│   ├── OpenSSL-CA-12312491.gns3project
│   └── images/
│
├── Password-Hashing-Lab-Solution/        # Service 2 — Password Security
│   ├── Password-Hashing-12312491.md
│   ├── Activities-Evidence.md
│   ├── Password-Hashing-12312491.gns3project
│   └── images/
│
└── SSH-Hardening-12312491-ePortfolio/    # Service 3 — SSH Hardening
    ├── SSH-Hardening-12312491-Lab-Outputs.md
    ├── Activities-Evidence.md
    ├── SSH-Hardening-12312491.gns3project
    ├── SSH-Hardening-12312491-admin.pcap
    ├── SSH-Hardening-12312491-internal.pcap
    └── images/
```

## Services

| Service | Summary | Write-up |
|---|---|---|
| **PKI & HTTPS** | Two-tier CA (root + intermediate), server certificate issued from the chain, HTTPS verified end-to-end, TLS handshake captured and analysed. | [`OpenSSL-CA-12312491-Lab-Outputs.md`](OpenSSL-CA-12312491-ePortfolio/OpenSSL-CA-12312491-Lab-Outputs.md) |
| **Password Security** | Hash algorithms compared (MD5, SHA-512, yescrypt), PAM password-quality and account-lockout policies configured and tested, cracking speed demonstrated with John the Ripper. | [`Password-Hashing-12312491.md`](Password-Hashing-Lab-Solution/Password-Hashing-12312491.md) |
| **SSH Hardening** | Ed25519 key-based login, hardened `sshd_config`, fail2ban blocking brute-force attempts, and an SSH tunnel through a bastion host to an internal service. | [`SSH-Hardening-12312491-Lab-Outputs.md`](SSH-Hardening-12312491-ePortfolio/SSH-Hardening-12312491-Lab-Outputs.md) |

## Activity evidence

Evidence for the required Week 1–4 interactive activities is committed alongside each service:

- Week 1 — [`Week1.md`](Week1.md)
- Week 2 — [`Activities-Evidence.md`](OpenSSL-CA-12312491-ePortfolio/Activities-Evidence.md)
- Week 3 — [`Activities-Evidence.md`](Password-Hashing-Lab-Solution/Activities-Evidence.md)
- Week 4 — [`Activities-Evidence.md`](SSH-Hardening-12312491-ePortfolio/Activities-Evidence.md)

## Student

Abishek Sapkota — Student ID 12312491
