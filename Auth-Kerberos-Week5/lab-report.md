# Lab Report: Kerberos SSO Authentication (Auth-Kerberos)

**Student ID:** 12312491
**Project:** `Auth-Kerberos-12312491.gns3project`
**Realm:** `EXAMPLE.COM`

## Aim

Use a Kerberos Key Distribution Centre (KDC) to grant a ticket to a client, enabling SSH login to a server without a password.

## Topology

Three GNS3 Kerberos Host nodes on one switched LAN:

| Node | Fully qualified name | Address | Role |
|---|---|---|---|
| `kdc` | `kdc.example.com` | `10.10.1.10/24` | Key Distribution Centre, issues tickets |
| `server` | `server.example.com` | `10.10.1.20/24` | SSH server logged in to |
| `client` | `client.example.com` | `10.10.1.30/24` | Runs `kinit` and `ssh` |

## Task by task summary

### Task 1: Topology, addressing, naming

Three hosts and a switch built, each node given its FQDN and address per the table above, and `/etc/hosts` populated with all three name mappings on every node. `ping` between `client`, `server`, and `kdc` by name confirmed name resolution before any Kerberos step.

### Task 2: Initialise the KDC

`kdb5_util create -s -P abishekkerberos` created the realm database on `kdc`. An empty `/var/lib/krb5kdc/kadm5.acl` was created, then `krb5kdc` and `kadmind` were started. Both confirmed running via `pgrep`.
**Evidence:** `images/task02-kdc-database-daemons.png`

### Task 3: Principals and keytab export

Via `kadmin.local` on `kdc`:
- `student@EXAMPLE.COM`: user principal (password based)
- `host/server.example.com@EXAMPLE.COM`: host principal (randkey)
- `host/client.example.com@EXAMPLE.COM`: host principal (randkey)
- Server key exported to `/tmp/server.keytab` via `ktadd` (two entries: aes256 and aes128)

Note: the first attempt at creating `host/client.example.com` silently didn't run (two commands pasted too close together). Caught via `listprincs` and re-run successfully.
**Evidence:** `images/task03-principals-keytab-export.png`, `images/task03-troubleshoot-listprincs-shell.png`, `images/task03-client-principal-created.png`

### Task 4: Install the keytab on the server

Transfer was attempted first via `scp`, which failed twice: once because `sshd`/`sshd.krb5` wasn't running yet on `server` (host keys hadn't been generated, `ssh-keygen -A` fixed this), and once on a rejected root password. The transfer was ultimately completed via the documented HTTP method instead: `python3 -m http.server 8080` on `kdc`, fetched with `curl` on `server` straight to `/etc/krb5.keytab`.
Verified with `klist -k /etc/krb5.keytab`: `host/server.example.com@EXAMPLE.COM` listed twice (KVNO 2, both encryption types).
**Evidence:** `images/task04-scp-connection-refused.png`, `images/task04-troubleshoot-sshd-not-found.png`, `images/task04-troubleshoot-find-sshd-krb5.png`, `images/task04-hostkeys-generated-mv-before-transfer.png`, `images/task04-scp-auth-failed.png`, `images/task04-http-server-transfer-success.png`, `images/task04-checkpoint-klist-keytab-confirmed.png`

### Task 5: Enable GSSAPI, start the Kerberos SSH daemon

`GSSAPIAuthentication yes` and `GSSAPICleanupCredentials yes` uncommented in `/etc/ssh/sshd_config` on `server`. `sshd.krb5` (the image's Kerberos capable daemon; no plain `sshd` binary exists on this image) required host keys generated via `ssh-keygen -A` before it would start. Confirmed listening via `pgrep -f sshd.krb5`.
**Evidence:** `images/task05-checkpoint-gssapi-daemon-running.png`

### Task 6: Capture start, obtain ticket on client

Packet capture started on the `client` to switch link in GNS3 before any Kerberos traffic was generated. `kinit student` (password `abishekkerberos`) succeeded silently. `klist` confirmed `Default principal: student@EXAMPLE.COM` and a `krbtgt/EXAMPLE.COM@EXAMPLE.COM` ticket.
**Required capture (item 3):** `images/Auth-Kerberos-12312491-klist.png`

### Task 7: Kerberos only login, capture analysis

```
ssh -o GSSAPIAuthentication=yes -o PasswordAuthentication=no \
    -o PreferredAuthentications=gssapi-with-mic student@server.example.com
```
Logged in to `server` with **no password prompt**. `klist` afterward showed a second entry, a service ticket for `host/server.example.com`. `kdestroy` followed by an identical login attempt was refused (`Permission denied`), confirming the ticket, not some other credential, carried the authentication.

Capture stopped and saved. Filtering on `kerberos` in Wireshark showed exactly four packets, all UDP/88 between `client` (10.10.1.30) and `kdc` (10.10.1.10):
- AS-REQ / AS-REP at t=0.000000 / 0.000569 (from `kinit`)
- TGS-REQ / TGS-REP at t=230.385111 / 230.385855, roughly 230s later, triggered by the SSH connection rather than by `kinit`

**Required capture (item 4):** `images/Auth-Kerberos-12312491-ssh.png`
**Required capture (item 5):** `Auth-Kerberos-client.pcap`
**Supporting evidence:** `images/task07-checkpoint-klist-two-tickets-kdestroy-denied.png`, `images/task07-wireshark-kerberos-filter-4packets.png`, `images/task07-wireshark-as-req-detail.png`

## Submission checklist

| Item | Status |
|---|---|
| 1. Exported project (`Auth-Kerberos-12312491.gns3project`) | Included |
| 2. Network screenshot (`Auth-Kerberos-12312491-network.png`) | Not yet captured; take a screenshot of the GNS3 topology/canvas and add it to `images/` |
| 3. `klist` screenshot | Included: `images/Auth-Kerberos-12312491-klist.png` |
| 4. SSH login screenshot | Included: `images/Auth-Kerberos-12312491-ssh.png` |
| 5. Client packet capture (`.pcap`) | Included: `Auth-Kerberos-client.pcap` |

## Write up questions

Not yet answered. The five reflection questions (KDC's role, what a keytab is, what Kerberos sends instead of a password, KDC down scenario, and the AS/TGS timing comparison) still need to be written up as prose. The Task 7 Wireshark evidence above, particularly the AS-REQ `padata` fields and the AS/TGS timestamp gap, is the direct evidence to cite for questions 3 and 5.
