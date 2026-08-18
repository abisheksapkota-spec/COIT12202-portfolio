# Solution Guide: Kerberos SSO Authentication

Consolidated command reference for all seven tasks, including the troubleshooting fixes needed on this image. Realm: `EXAMPLE.COM`. Replace `<studentid>` with your own throughout.

## Task 1: Build the topology, address and name the three nodes

1. Create a GNS3 project named `Auth-Kerberos-<studentid>`.
2. Add three **Kerberos Host** nodes plus one Ethernet switch; connect all three hosts to the switch.
3. Label the hosts `kdc`, `server`, `client`. Start all nodes.
4. On each node (using its own values):
   ```
   hostname <fqdn>
   ip addr add <address>/24 dev eth0
   ip link set eth0 up
   ```

   | Node | FQDN | Address |
   |---|---|---|
   | kdc | kdc.example.com | 10.10.1.10 |
   | server | server.example.com | 10.10.1.20 |
   | client | client.example.com | 10.10.1.30 |

5. On **every** node: `nano /etc/hosts`, leave the `localhost` lines, append:
   ```
   10.10.1.10 kdc.example.com kdc
   10.10.1.20 server.example.com server
   10.10.1.30 client.example.com client
   ```
6. Confirm the realm config ships pre-set: `cat /etc/krb5.conf` (realm `EXAMPLE.COM`).

**Checkpoint:** from `client`, `ping -c1 server.example.com` and `ping -c1 kdc.example.com` succeed.
**Capture:** GNS3 topology screenshot, saved as `Auth-Kerberos-<studentid>-network.png`.

## Task 2: Initialise the Kerberos database and start the KDC

On `kdc`:
```
kdb5_util create -s -P abishekkerberos
touch /var/lib/krb5kdc/kadm5.acl
krb5kdc
kadmind
pgrep krb5kdc
pgrep kadmind
```
Both daemons background themselves and print nothing on success. `pgrep` must return a PID for each.

**Checkpoint:** `pgrep krb5kdc` and `pgrep kadmind` each return a PID.

## Task 3: Create the principals and export the server's keytab

On `kdc`:
```
kadmin.local
```
At the `kadmin.local:` prompt, run these one at a time rather than pasted together (commands can silently drop if pasted too fast):
```
addprinc -pw abishekkerberos student
addprinc -randkey host/server.example.com
addprinc -randkey host/client.example.com
ktadd -k /tmp/server.keytab host/server.example.com
listprincs
quit
```
`listprincs` should show `student@EXAMPLE.COM`, `host/server.example.com@EXAMPLE.COM`, `host/client.example.com@EXAMPLE.COM` (plus Kerberos defaults). `ktadd` should report two entries (aes256, aes128).

**Checkpoint:** all three principals exist; `/tmp/server.keytab` exists on `kdc`.

## Task 4: Install the keytab on the server

This image has no plain `sshd`, only `sshd.krb5`, and it refuses to start without host keys. Do this first on `server`, since Task 4's transfer needs a working SSH/HTTP path either way:
```
ssh-keygen -A
```

### Preferred method: HTTP (no credentials needed)

On `kdc`:
```
cd /tmp && python3 -m http.server 8080 &
```
On `server`:
```
curl -s -o /etc/krb5.keytab http://kdc.example.com:8080/server.keytab
```

### Alternative: scp

Requires a running `sshd.krb5` on `server` (`/usr/sbin/sshd.krb5`) and a known root password (`passwd` on `server`'s own console, since there is no documented default). From `kdc`:
```
scp /tmp/server.keytab root@server.example.com:/tmp/server.keytab
```
then on `server`:
```
mv /tmp/server.keytab /etc/krb5.keytab
chmod 600 /etc/krb5.keytab
```

### Verify (on `server`, either method)
```
klist -k /etc/krb5.keytab
```
Expect `host/server.example.com@EXAMPLE.COM` listed twice (KVNO 2, aes256 plus aes128).

**Checkpoint:** `klist -k /etc/krb5.keytab` on `server` lists `host/server.example.com@EXAMPLE.COM`.

## Task 5: Enable GSSAPI and start the Kerberos SSH daemon

On `server`:
```
nano /etc/ssh/sshd_config
```
Find and uncomment:
```
GSSAPIAuthentication yes
GSSAPICleanupCredentials yes
```
Save (`Ctrl+O`, `Enter`, `Ctrl+X`), then:
```
grep -i gssapi /etc/ssh/sshd_config
ssh-keygen -A            # if not already done in Task 4
/usr/sbin/sshd.krb5 -t   # prints nothing if config is valid
/usr/sbin/sshd.krb5
pgrep -f sshd.krb5
```

**Checkpoint:** `grep -i gssapi` shows both options `yes` with no `#`; `pgrep -f sshd.krb5` returns a PID.

## Task 6: Start a packet capture, obtain a Kerberos ticket on the client

1. In GNS3: right click the `client` to switch link, choose **Start capture**. Do this before `kinit`; leave it running through Task 7.
2. On `client`:
   ```
   kinit student
   ```
   Password: `abishekkerberos`. Succeeds silently.
3. ```
   klist
   ```
   Expect `Default principal: student@EXAMPLE.COM` and a `krbtgt/EXAMPLE.COM@EXAMPLE.COM` ticket.

**Checkpoint:** as above.
**Capture:** `Auth-Kerberos-<studentid>-klist.png`.

## Task 7: Log in using only Kerberos, then read the capture

On `client`:
```
ssh -o GSSAPIAuthentication=yes -o PasswordAuthentication=no \
    -o PreferredAuthentications=gssapi-with-mic student@server.example.com
```
Accept the host key prompt on first connection. Should log in with no password prompt.

```
exit
klist
```
A second ticket, `host/server.example.com@EXAMPLE.COM`, now appears alongside the `krbtgt`.

Prove it is really Kerberos:
```
kdestroy
ssh -o GSSAPIAuthentication=yes -o PasswordAuthentication=no \
    -o PreferredAuthentications=gssapi-with-mic student@server.example.com
```
This must now fail with `Permission denied`.

Stop the capture (GNS3: right click the link, choose **Stop capture**), save as `Auth-Kerberos-<studentid>-client.pcap`.

In Wireshark, filter:
```
kerberos
```
Expect exactly 4 packets, all UDP/88 between `client` and `kdc`: AS-REQ/AS-REP (from `kinit`), then TGS-REQ/TGS-REP (appearing after the SSH connection starts). Clear the filter to see the SSH session itself as ordinary TCP/22.

**Checkpoint:** login succeeds with no password while holding a ticket; refused after `kdestroy`.
**Captures:** `Auth-Kerberos-<studentid>-ssh.png` (successful login), `Auth-Kerberos-<studentid>-client.pcap`.

## If the Kerberos login fails, check in this order

1. **Names.** `/etc/hosts` correct on all three nodes; SSH to the FQDN, not the IP.
2. **The KDC.** `pgrep krb5kdc` returns a PID on `kdc`.
3. **The keytab.** `klist -k /etc/krb5.keytab` on `server` lists the host principal, not an empty file.
4. **GSSAPI.** `grep -i gssapi /etc/ssh/sshd_config` shows `yes`, no `#`.
5. **The daemon.** `pgrep -f sshd.krb5` returns a PID (note: `sshd.krb5`, not `sshd`).
6. **The ticket.** `klist` on `client` shows an unexpired `krbtgt`.
7. **The realm's case.** `EXAMPLE.COM` is upper case; case sensitive.
8. **The clock.** All three nodes must agree on the time.

## Other gotchas hit during this run (not in the original doc)

- Commands pasted too fast into `kadmin.local` can silently drop; always confirm with `listprincs` before moving on.
- `kadmin.local` subcommands only work inside the `kadmin.local:` prompt; running `listprincs` or `addprinc` at the plain shell (`/ #`) gives `not found`.
- `sshd.krb5` refuses to start with `no hostkeys available` until `ssh-keygen -A` has been run at least once on `server`.
- No default root password exists on these images for SSH login between nodes. If using `scp` instead of the HTTP method, set one first with `passwd` on the target node's own console.
