# OPNsense Firewall Lab — Solution Guide

**Student ID:** 12312491
**Aim:** Build a segmented network with OPNsense, configure firewall rules to control traffic between WAN, LAN and DMZ zones, and verify that only permitted traffic passes.

This guide walks through every task in order, with the exact command or GUI field to use and where to run/enter it. It follows the provided `OPNsense-Firewall-Template.gns3project`, which already has the topology wired and the three hosts pre-addressed — that's noted where it applies so you don't redo work that's already done for you.

---

## What you need to submit

| # | File | What it is |
|---|---|---|
| 1 | `OPNsense-Firewall-12312491.gns3project` | Exported project |
| 2 | `OPNsense-Firewall-12312491-network.png` | Screenshot of the topology |
| 3 | `OPNsense-Firewall-12312491-rules-lan.png`, `-rules-dmz.png`, `-rules-wan.png` | Firewall rules per interface |
| 4 | `OPNsense-Firewall-12312491-log.png` | Live log showing blocked traffic |

Take each **Capture** screenshot the moment its task says to — the state doesn't persist once you move on.

> **GUI changes only take effect after Apply.** Every OPNsense change is Save → **Apply changes** (the orange banner). A rule that "should work" but doesn't is almost always a missed Apply.

> **Keep your work.** GNS3 rebuilds a node from its image on reopen, so anything outside a kept directory is lost, and any running service (like the DMZ web server) needs restarting after a reopen.

---

## Topology reference

| Node | Segment | OPNsense port | Address |
|---|---|---|---|
| OPNsense | all three | `em0`–`em2` (`vtnet0`–`vtnet2`) | `.1` in each subnet |
| WANHost | WAN `203.0.113.0/24` | `em0` / `vtnet0` | `203.0.113.10/24` |
| LANHost (Firefox Host) | LAN `192.168.1.0/24` | `em1` / `vtnet1` | `192.168.1.10/24` |
| DMZHost | DMZ `172.16.1.0/24` | `em2` / `vtnet2` | `172.16.1.10/24` |

GNS3 labels the firewall's ports `em0`–`em2`; OPNsense's own console calls the same cards `vtnet0`–`vtnet2`. `em0`→WAN, `em1`→LAN, `em2`→DMZ throughout.

---

## Task 1: Build the topology and address the hosts

**In the template project this is already done** — `Switch1/2/3`, `LANHost`, `DMZHost`, `WANHost` and `OPNsense` are already placed and wired (`OPNsense` adapter 0 → WAN switch → `WANHost`; adapter 1 → LAN switch → `LANHost`; adapter 2 → DMZ switch → `DMZHost`), and each host's `/etc/network/interfaces` already has its static block uncommented with the addresses above. You only need to:

1. Open `OPNsense-Firewall-Template.gns3project` in GNS3.
2. Start all four nodes (right-click each → **Start**, or select all and use the toolbar's start-all button).

If you're building from scratch instead of the template, on each **stopped** host:

1. Right-click the host → **Configure**.
2. Open **Edit network configuration**.
3. Uncomment the static block (delete the leading `#` characters) and set that host's `address`, `netmask` (dotted-quad, e.g. `255.255.255.0`, not `/24`), and `gateway`. The `gateway` line is required — it's the host's default route.

Example, the WAN host's finished block:

```
auto eth0
iface eth0 inet static
    address 203.0.113.10
    netmask 255.255.255.0
    gateway 203.0.113.1
```

**Checkpoint:** all four nodes started; each host on its own switch with only OPNsense bridging the three segments.

**Capture:** `OPNsense-Firewall-12312491-network.png`

---

## Task 2: Assign the OPNsense interfaces at the console

Run entirely at the **OPNsense console** (double-click the OPNsense node in GNS3).

1. Wait for the boot menu (boot takes a few minutes) and log in:
   - Username: `root`
   - Password: `opnsense`
2. Choose option `1` — **Assign interfaces** — and answer the prompts in this order:

| Prompt | Answer |
|---|---|
| `Do you want to configure LAGGs now?` | `N` |
| `Do you want to configure VLANs now?` | `N` |
| `Enter the WAN interface name` | `vtnet0` |
| `Enter the LAN interface name` | `vtnet1` |
| `Enter the Optional interface 1 name` | `vtnet2` (this is the DMZ) |
| `Enter the Optional interface 2 name` | *(Enter — leave empty)* |
| `Do you want to proceed?` | `y` |

> OPNsense boots with LAN and WAN swapped (`LAN (vtnet0)` / `WAN (vtnet1)`), so this step is a genuine reassignment, not a no-op. `OPT1` = DMZ for the rest of the lab.

3. Choose option `2` — **Set interface IP address** — once per interface, **WAN → LAN → DMZ**:

| Interface | IPv4 address | Bit count | Upstream gateway |
|---|---|---|---|
| `3 - WAN (vtnet0)` | `203.0.113.1` | `24` | *(Enter — none)* |
| `1 - LAN (vtnet1)` | `192.168.1.1` | `24` | *(Enter — none)* |
| `2 - OPT1 (vtnet2)` | `172.16.1.1` | `24` | *(Enter — none)* |

For every interface, answer these the same way:

| Prompt | Answer |
|---|---|
| `Configure IPv4 address ... via DHCP?` | `n` |
| `Configure IPv6 address ... via WAN tracking?` (LAN only) | `n` |
| `Configure IPv6 address ... via DHCP6?` | `n` |
| `Enter the new ... IPv6 address` | *(Enter — none)* |
| `Do you want to enable the DHCP server on ...?` | `n` |
| `Do you want to change the web GUI protocol from HTTPS to HTTP?` | `n` |
| `Do you want to generate a new self-signed web GUI certificate?` | `n` |
| `Restore web GUI access defaults?` | `n` |

Watch for: type `n` every time (the WAN's two DHCP prompts default to *yes*); address and mask are **separate** prompts (`203.0.113.1` then `24`, never combined); set the LAN too even though it already shows `192.168.1.1/24`.

**Checkpoint** — console banner reads:

```
 LAN (vtnet1)    -> v4: 192.168.1.1/24
 OPT1 (vtnet2)   -> v4: 172.16.1.1/24
 WAN (vtnet0)    -> v4: 203.0.113.1/24
```

---

## Task 3: Start the DMZ web server

If any host has no address (started before its config was edited), fix it live at that host's console instead of restarting it. On the **WAN host**:

```bash
ip addr add 203.0.113.10/24 dev eth0
ip link set eth0 up
ip route add default via 203.0.113.1
```

(Same pattern on the DMZ host with `172.16.1.10/24` via `172.16.1.1`, and the LAN host with `192.168.1.10/24` via `192.168.1.1`.)

On the **DMZ host**, start a web server on port 80:

```bash
mkdir -p /var/www
echo "<h1>DMZ Web Server</h1>" > /var/www/index.html
cd /var/www && python3 -m http.server 80 &
```

Leave it running for the whole lab. Confirm it serves locally, still on the **DMZ host**:

```bash
curl http://localhost/
```

Must return `<h1>DMZ Web Server</h1>`.

Only the **LAN host** can ping its gateway at this point (`ping 192.168.1.1` from the LAN host) — the DMZ and WAN hosts cannot ping theirs yet, and won't be able to until their firewall rules exist in Task 5. That's expected. Test those two segments from the firewall side instead: at the **OPNsense console**, choose option `7` (**Ping host**) and enter:

```
172.16.1.10
203.0.113.10
```

Both should reply.

**Checkpoint** — LAN host pings `192.168.1.1`; firewall (option 7) pings both `172.16.1.10` and `203.0.113.10`; `curl http://localhost/` on the DMZ host returns the DMZ page.

---

## Task 4: Reach the OPNsense web GUI from the LAN

The LAN host is a Firefox Host, so its console is **VNC**, not a terminal.

1. In the GNS3 node list, find the LAN host's VNC port — shown as `vnc://<gns3-vm-ip>:<vnc-port>`, usually **5900**.
2. On the **GNS3 VM's own shell**, start the noVNC bridge (VNC port first, web port second):

   ```bash
   ./start-vnc.sh 5900 6080
   ```

3. In a browser on your own machine, open:

   ```
   http://<gns3-vm-ip>:6080/vnc.html
   ```

4. Click **Connect**. The LAN host's desktop appears.
5. Open the bottom-left menu → **Terminal**. You'll need this shell again in Task 7.

In the LAN host's **Firefox**:

1. Browse to `https://192.168.1.1`.
2. Click **Advanced...** → **Accept the Risk and Continue** on the security warning.
3. Log in: `root` / `opnsense`.
4. OPNsense opens the setup wizard. **Do not run it** — click the **OPNsense logo** (top left) to leave.

Then unblock TEST-NET-3 on the WAN (this lab's WAN range, `203.0.113.0/24`, is a bogon range by default):

1. Go to **Interfaces → [WAN]**.
2. Under *Generic configuration*, untick both:
   - **Block private networks**
   - **Block bogon networks**
3. **Save** (bottom of page) → **Apply changes** (orange banner button).

**Checkpoint** — GUI reachable, off the wizard page, and **Interfaces → WAN** shows both boxes unticked with no pending-changes banner.

---

## Task 5: Create the aliases and write the LAN and DMZ rules

### Aliases

**Firewall → Aliases** → orange **+** (leave OPNsense's own factory aliases alone; yours are added alongside):

| Name | Type | Content |
|---|---|---|
| `LAN_NET` | Network(s) | `192.168.1.0/24` |
| `DMZ_NET` | Network(s) | `172.16.1.0/24` |
| `DMZ_WEBSERVER` | **Host(s)** | `172.16.1.10` |

Content is a tag field: type the value, press Enter. **Save** each, then click **Apply** at the foot of the Aliases page — an alias saved but not applied matches nothing.

### Delete the factory LAN rules

**Firewall → Rules → LAN** already has two rules to remove:

1. Tick **Default allow LAN to any rule**.
2. Tick **Default allow LAN IPv6 to any rule**.
3. Click the **bin** icon in the table header.
4. Confirm **Yes**.

Leave the collapsed *Automatically generated rules* group (the anti-lockout rule) alone. If either *Default allow* rule stays above your own, it matches everything first and your rules never fire.

### LAN rules (top → bottom, in this exact order)

| # | Action | Protocol | Source | Destination | Port | Log |
|---|---|---|---|---|---|---|
| 1 | Pass | TCP | `LAN_NET` | any | HTTP (80) | No |
| 2 | Pass | TCP | `LAN_NET` | any | HTTPS (443) | No |
| 3 | Pass | TCP/UDP | `LAN_NET` | any | DNS (53) | No |
| 4 | Pass | TCP | `LAN_NET` | `DMZ_NET` | SSH (22) | No |
| 5 | Block | any | any | any | any | **Yes** |

One service per rule — the *Destination port range* field is a single from–to range, so 80 and 443 can't share a rule.

Rule 1 fields (orange **+** → **Edit Firewall rule**), everything else left at default:

| Field | Value |
|---|---|
| Action | Pass |
| Interface | LAN |
| Direction | in |
| TCP/IP Version | IPv4 |
| Protocol | TCP |
| Source | `LAN_NET` |
| Destination | any |
| Destination port range | from `HTTP`, to `HTTP` |

Rules 2–5, same form, changing only:

| Rule | Change from rule 1 |
|---|---|
| 2 | Destination port range: from `HTTPS`, to `HTTPS` |
| 3 | Protocol `TCP/UDP`; port range from `DNS`, to `DNS` |
| 4 | Destination `DMZ_NET`; port range from `SSH`, to `SSH` |
| 5 | Action **Block**; Protocol `any`; Source `any`; port range `any`; tick **Log**; Description `Block any (LAN)` |

Ports are chosen from the dropdown by service name, not typed as numbers. Rule 5's **Description** becomes the **Label** in the Task 8 log — name it exactly `Block any (LAN)`.

A newly saved rule lands at the **bottom** of the tab — drag the rows until the on-screen order matches the table (Block any last), then **Apply changes**. Confirm the order before testing.

### DMZ rules

**Firewall → Rules → OPT1** (this tab is the DMZ; no factory rule to delete here):

| # | Action | Protocol | Source | Destination | Port | Log |
|---|---|---|---|---|---|---|
| 1 | Pass | TCP/UDP | `DMZ_NET` | any | DNS (53) | No |
| 2 | Block | any | any | any | any | **Yes** |

Rule 2's Description: `Block any (DMZ)`.

**Checkpoint** — LAN tab: five rules, Block any last, neither factory Default-allow rule active. OPT1 tab: two rules, Block any last. Both applied.

**Capture:** `OPNsense-Firewall-12312491-rules-lan.png`, `OPNsense-Firewall-12312491-rules-dmz.png`

---

## Task 6: Publish the DMZ web server to the WAN with a port forward

**Firewall → NAT → Port Forward** → **+** → **Edit Redirect entry**:

| Field | Value |
|---|---|
| Interface | WAN |
| TCP/IP Version | IPv4 |
| Protocol | TCP |
| Destination | **WAN address** *(dropdown value, not typed)* |
| Destination port range | from `HTTP`, to `HTTP` |
| Redirect target IP | `DMZ_WEBSERVER` |
| Redirect target port | `HTTP` |
| Filter rule association | **Add associated filter rule** *(default — leave it)* |

Leave everything else at default. **Save** → **Apply changes**.

OPNsense writes the matching WAN pass rule for you — confirm at **Firewall → Rules → WAN**:

| Action | Protocol | Source | Destination | Port |
|---|---|---|---|---|
| Pass | IPv4 TCP | any | `DMZ_WEBSERVER` | 80 (HTTP) |

That row has only a bin icon (no edit/copy) — it belongs to the NAT entry, so change it by editing the port forward, not the rule directly.

**Checkpoint** — WAN tab shows the auto-created Pass rule for `DMZ_WEBSERVER:80`.

**Capture:** `OPNsense-Firewall-12312491-rules-wan.png`

---

## Task 7: Test what is allowed and what is blocked

Run each set of commands from the named host's console/terminal. A blocked test is a **pass** for your firewall.

**LAN host:**

```bash
curl http://172.16.1.10/
ping -c 3 203.0.113.10
ssh 172.16.1.10
```

**WAN host:**

```bash
curl http://203.0.113.1/
ping -c 3 192.168.1.10
curl -m 5 http://172.16.1.10/
```

**DMZ host:**

```bash
ping -c 3 192.168.1.10
curl -m 5 http://192.168.1.10/
```

### Expected results

| From | Test | Expected result | What it proves |
|---|---|---|---|
| LAN | HTTP to DMZ web server | `<h1>DMZ Web Server</h1>` | LAN rule 1 permits LAN→DMZ web, statefully |
| LAN | Ping WAN host | `100% packet loss` | No ICMP rule on LAN — hits **Block any** |
| LAN | SSH to DMZ host | `Connection refused` | LAN rule 4 let it through; DMZ has no SSH server |
| WAN | HTTP to WAN address | `<h1>DMZ Web Server</h1>` | Port forward reaches the DMZ web server |
| WAN | Ping LAN host | `100% packet loss` | WAN default deny — only the forwarded flow is allowed in |
| WAN | HTTP direct to DMZ host | `<h1>DMZ Web Server</h1>` | The port-forward's rule matches the *translated* destination, so it also passes direct traffic to `172.16.1.10:80` |
| DMZ | Ping LAN host | `100% packet loss` | DMZ **Block any** — DMZ can't start anything toward LAN |
| DMZ | HTTP to LAN host | Times out | Same rule, for TCP as well as ICMP |

Reading the failures: `Connection refused` means the packet arrived and the firewall passed it — that's a pass. Timing out / `100% packet loss` means nothing answered at all — that's what a **Block** looks like from outside (hence `-m 5` / `-c 3` in the commands).

If a result doesn't match, check in order: (1) pending-changes banner — Apply it; (2) rule order on the tab — Block any must be last, no leftover Default-allow above yours; (3) routing — a host that can't reach its own gateway has a wrong address or missing `gateway` line, no rule fixes that; (4) WAN-only failures — re-check both bogon/private boxes are unticked; (5) an alias saved but never applied matches nothing.

**Checkpoint** — every row above matches.

---

## Task 8: Log the blocked traffic and review it

1. Confirm **Log** is ticked on both your **Block any** rules (**Firewall → Rules → LAN** and **→ OPT1** — edit if not).
2. Go to **Firewall → Log Files → Live View**.
3. Set the filter row: `action` `contains` `block`, press **+**. A badge shows the active filter (click it to remove later).
4. Re-run the three blocked tests from Task 7 — LAN→WAN ping, DMZ→LAN ping, WAN→LAN ping — and watch entries arrive.

Blocked packets appear as red rows, each showing interface, time, source, destination, protocol, and the **Label** of the rule that stopped it:

```
Interface  Time                  Source        Destination   Proto  Label
opt1       2026-08-01T02:27:32   172.16.1.10   192.168.1.10  icmp   Block any (DMZ)
opt1       2026-08-01T02:27:31   172.16.1.10   192.168.1.10  icmp   Block any (DMZ)
opt1       2026-08-01T02:27:30   172.16.1.10   192.168.1.10  icmp   Block any (DMZ)
wan        2026-08-01T02:27:25   203.0.113.10  192.168.1.10  icmp   Default deny / state violation rule
wan        2026-08-01T02:27:24   203.0.113.10  192.168.1.10  icmp   Default deny / state violation rule
wan        2026-08-01T02:27:23   203.0.113.10  192.168.1.10  icmp   Default deny / state violation rule
lan        2026-08-01T02:27:18   192.168.1.10  203.0.113.10  icmp   Block any (LAN)
lan        2026-08-01T02:27:17   192.168.1.10  203.0.113.10  icmp   Block any (LAN)
lan        2026-08-01T02:27:15   192.168.1.10  203.0.113.10  icmp   Block any (LAN)
```

Three rows per test — each `ping -c 3` sends three packets. The WAN block isn't yours (nothing on the WAN tab matches it), so it's caught and logged by OPNsense's built-in default deny automatically. The **ⓘ** at the end of a row opens the detail view.

**Checkpoint** — all three blocked flows appear in Live View on `lan`, `opt1` and `wan`, with `wan` labelled *Default deny / state violation rule* and `lan`/`opt1` carrying your own rule descriptions.

**Capture:** `OPNsense-Firewall-12312491-log.png`

### Shut down before exporting

The configuration lives on OPNsense's virtual disk — shut it down properly rather than just stopping the GNS3 node. At the **OPNsense console**, choose option `5` (**Power off system**):

```
Enter an option: 5

The system will halt and power off. Do you want to proceed? [y/N]: y
```

Wait for it to halt, then stop the remaining nodes and export the project (item 1). Only OPNsense needs the clean shutdown — the hosts can be stopped any time.

---

## Reflection questions (answer separately, in your own words)

The lab also asks for short written answers to these — they're conceptual, so they're not covered by this technical guide:

1. Why does rule ordering matter under "first match wins," and what happens if a broad "allow all" rule sits above a specific deny rule?
2. Why put the web server in a DMZ rather than the LAN, and how does that limit the impact of a compromised web server?
3. How does stateful inspection differ from simple packet filtering, and what does it buy you?
4. Why have an explicit "deny all" at the end of each interface's rules, even though OPNsense has an implicit deny already?
5. In Task 7 the WAN host reached the DMZ web server directly, not just via the published WAN address — why, in terms of the order the firewall translates vs. filters a packet? What would you change to make the service reachable *only* via the WAN address, and what's the trade-off?
