## Cisco Packet Tracer — Labs B & C: SSH/FTP + DHCP/DNS Server

> 🌐 **English** | [日本語](./PACKET_TRACER_GUIDE.ja.md)

> Companion guide to [Session 3 — Network Services & Security](./README.md).
> Read this **before** doing **Lab B** and **Lab C** in the [README](./README.md#hands-on-lab).
> New to Packet Tracer? Start with the [Session 1 Packet Tracer guide](../S1/PACKET_TRACER_GUIDE.md) for installation, the window tour, and CLI basics.

---

- [Lab B — SSH \& FTP on the Multi-Subnet Network](#lab-b--ssh--ftp-on-the-multi-subnet-network)
  - [Step B1 — Reuse the Session 2 network + add an FTP server](#step-b1--reuse-the-session-2-network--add-an-ftp-server)
  - [Step B2 — Configure SSH on the router](#step-b2--configure-ssh-on-the-router)
  - [Step B3 — Log in with SSH (and see why not Telnet)](#step-b3--log-in-with-ssh-and-see-why-not-telnet)
  - [Step B4 — Configure the FTP server](#step-b4--configure-the-ftp-server)
  - [Step B5 — Transfer a file with the FTP client](#step-b5--transfer-a-file-with-the-ftp-client)
  - [Step B6 — Security wrap-up: encrypted vs plaintext](#step-b6--security-wrap-up-encrypted-vs-plaintext)
  - [Bonus — SSH \& FTP on Real Machines](#bonus--ssh--ftp-on-real-machines)
    - [SSH — secure remote shell (+ encrypted file copy)](#ssh--secure-remote-shell--encrypted-file-copy)
    - [File transfer — prefer SFTP, fall back to FTP](#file-transfer--prefer-sftp-fall-back-to-ftp)
- [Lab B Questions](#lab-b-questions)
- [Lab C — DHCP \& DNS, continuing from Lab B](#lab-c--dhcp--dns-continuing-from-lab-b)
  - [Step C1 — Add a web page + DNS to the server](#step-c1--add-a-web-page--dns-to-the-server)
  - [Step C2 — Configure DHCP pools on the router](#step-c2--configure-dhcp-pools-on-the-router)
  - [Step C3 — Switch the PCs to DHCP](#step-c3--switch-the-pcs-to-dhcp)
  - [Step C4 — Verify: auto-address \& reach the server by name](#step-c4--verify-auto-address--reach-the-server-by-name)
  - [Step C5 — Open the browser and see the web page](#step-c5--open-the-browser-and-see-the-web-page)
  - [Step C6 — Watch DORA in Simulation Mode](#step-c6--watch-dora-in-simulation-mode)
- [Lab C Questions](#lab-c-questions)
    - [Practice Exercises](#practice-exercises)
- [Next Steps](#next-steps)

---

### What You'll Build

These two labs **reuse the multi-subnet network you built in [Session 2 Lab B](../S2/PACKET_TRACER_GUIDE.md)** — the three-department company (Engineering / Marketing / Management) routed by a single **2911** router — and add the **application-layer services** every real network runs:

- **Lab B** — **SSH** (secure remote management of the router) and **FTP** (a file-transfer server).
- **Lab C** — **DHCP** (the router auto-addresses the PCs) and **DNS** (reach the server by name). **Lab C continues directly from the Lab B file.**

| Device | Role | Address |
|:---|:---|:---|
| **Router2 (2911)** | SSH-managed router; later the **DHCP** server | gateways `192.168.0.1` / `.65` / `.97` |
| **FTP-server** (new) | **FTP** server; later also the **DNS** server | `192.168.0.98 /28`, gw `192.168.0.97` |
| **PC1–PC6** (from S2) | SSH/FTP clients; later **DHCP** clients | the S2 addressing plan |

> If you saved your `.pkt` from Session 2, just open it. If not, rebuild the [S2 topology](../S2/PACKET_TRACER_GUIDE.md#step-b2--build-the-topology) first — these labs assume the router interfaces and PC addresses already work (a cross-subnet `ping` succeeds).

---

# Lab B — SSH & FTP on the Multi-Subnet Network

**⏱️ ~30 min · Objective:** secure the router's remote management with **SSH**, stand up an **FTP** server, and transfer a file across the subnets — then see why one is safe to sniff and the other is not.

## Step B1 — Reuse the Session 2 network + add an FTP server

1. **Open your Session 2 `.pkt`** (the Engineering / Marketing / Management network).
2. From the device panel choose **End Devices → Server** and drop one on the canvas; name it **`FTP-server`**.

<p align="center">
  <img src="./img/SSH%26FTP/ftp-server-select-icon.png" alt="Selecting the Server device from the End Devices panel" width="720"><br>
  <em>Fig. 1 — Pick the <strong>Server</strong> (<code>Server-PT</code>) from <strong>End Devices</strong>.</em>
</p>

3. **Which switch?** Cable it to **`SW-Mgt`** — the **Management** department's `2960` switch. **A device must plug into the switch of the subnet whose IP range it uses**, and the server gets a *Management* address (`192.168.0.96/28`). Wiring it to `SW-Eng`/`SW-Mkt` would put it on the wrong subnet (wrong gateway, unreachable). Use a **free access port** — PC5/PC6 use `Fa0/1`/`Fa0/2`, so use **`Fa0/3`**:

   | Link | From | To |
   |:---|:---|:---|
   | FTP server → Management switch | `FTP-server` `FastEthernet0` | `SW-Mgt` `Fa0/3` |

4. Give the server a **static IPv4 address** (Config → FastEthernet0): **`192.168.0.98`**, mask `255.255.255.240`.

<p align="center">
  <img src="./img/SSH%26FTP/labB-ftp-server-IP-setting.png" alt="Setting the FTP-server's static IPv4 address" width="560"><br>
  <em>Fig. 2 — FTP-server static IP: <code>192.168.0.98</code>, mask <code>255.255.255.240</code> (the Management <code>/28</code>).</em>
</p>

5. Set its **Default Gateway** to **`192.168.0.97`** (the router's `g0/2`, the Management gateway) under Config → Global Settings.

<p align="center">
  <img src="./img/SSH%26FTP/labB-ftp-server-gateway-setting.png" alt="Setting the FTP-server's default gateway" width="600"><br>
  <em>Fig. 3 — FTP-server <strong>Default Gateway</strong> = <code>192.168.0.97</code> (router <code>g0/2</code>).</em>
</p>

When you're done, the topology looks like this — the new server sits on the Management subnet:

<p align="center">
  <img src="./img/SSH%26FTP/labB-step1-topology.png" alt="The three-department network with the FTP-server added" width="780"><br>
  <em>Fig. 4 — The S2 three-department network with <code>FTP-server</code> (<code>192.168.0.98</code>) on the Management subnet.</em>
</p>

## Step B2 — Configure SSH on the router

By default a Cisco router only offers **Telnet**, which sends everything — including your password — in **plaintext**. We'll require **SSH** instead. SSH needs a **hostname**, a **domain name** (the two together name the encryption key), and an **RSA key** before it can encrypt anything:

```ios
Router> enable
Router# configure terminal
Router(config)# hostname R-CORE
R-CORE(config)# ip domain-name lab.local
R-CORE(config)# enable secret class123
R-CORE(config)# username admin privilege 15 secret cisco123
R-CORE(config)# crypto key generate rsa
  How many bits in the modulus [512]: 1024
R-CORE(config)# ip ssh version 2
R-CORE(config)# line vty 0 4
R-CORE(config-line)# transport input ssh
R-CORE(config-line)# login local
R-CORE(config-line)# end
R-CORE# write memory
```

| Command | Why it's needed |
|:---|:---|
| `hostname` + `ip domain-name` | The RSA key is named `hostname.domain` — SSH **refuses to generate** a key without both. |
| `crypto key generate rsa` (1024+) | Creates the public/private key pair that **encrypts** the session. |
| `username … secret` + `enable secret` | A local login account (SSH needs real credentials, not just a line password). |
| `ip ssh version 2` | Forces the stronger **SSHv2**. |
| `line vty 0 4` → `transport input ssh` | The 5 virtual terminal lines now accept **SSH only** — Telnet is refused. |
| `login local` | Authenticate vty logins against the local `username` database. |

<p align="center">
  <img src="./img/SSH%26FTP/labB-router-cli-ssh-setting.png" alt="Router CLI after enabling SSHv2" width="600"><br>
  <em>Fig. 5 — The router CLI: hostname/domain set, the RSA key generated (<code>R-CORE.lab.local</code>, 1024-bit), SSHv2 enabled, and the vty lines restricted to SSH.</em>
</p>

## Step B3 — Log in with SSH (and see why not Telnet)

From **PC1**'s **Desktop → Command Prompt** (Engineering), connect to the router as `admin`:

```text
PC> ssh -l admin 192.168.0.1
Password: cisco123
R-CORE>
```

You now have a remote, **encrypted** session on the router. Confirm it on the router with `show ip ssh` and `show ssh`.

> 💡 **Try Telnet and watch it fail:** `telnet 192.168.0.1` is **refused** — because `transport input ssh` removed Telnet from the vty lines. That refusal *is* the security improvement: Telnet would have carried your `admin` password across the wire in clear text.

<p align="center">
  <img src="./img/SSH%26FTP/labB-step3-ssh-login.png" alt="PC1 opening an SSH session to the router" width="460"><br>
  <em>Fig. 6 — From PC1: <code>ssh -l admin 192.168.0.1</code> → the <code>R-CORE#</code> prompt, a remote encrypted session.</em>
</p>

## Step B4 — Configure the FTP server

Click **`FTP-server` → Services tab → FTP**:

1. Turn the **FTP service On**.
2. Add a user account — **Username `student`**, **Password `ftppass`** — and tick the permissions **Write, Read, Delete, Rename, List** (shown as `RWDNL`).
3. Click **+ (Add)**.

<p align="center">
  <img src="./img/SSH%26FTP/labB-step4-ftp-server%E3%83%BCsetting.png" alt="FTP service On with the student account and permissions" width="760"><br>
  <em>Fig. 7 — FTP service <strong>On</strong>; add user <code>student</code> / <code>ftppass</code> with <strong>Write/Read/Delete/Rename/List</strong>, then <strong>Add</strong>.</em>
</p>

## Step B5 — Transfer a file with the FTP client

From **PC1** (Engineering) — a *different* subnet from the server — open **Desktop → Command Prompt** and run the built-in FTP client. The traffic is routed across the `2911` to the Management subnet. Log in, then list the server's files with `dir`:

```text
PC> ftp 192.168.0.98
Username: student
Password: ftppass
ftp> dir
```

<p align="center">
  <img src="./img/SSH%26FTP/labB-FTP-PC1-connecting.png" alt="PC1 logging into the FTP server and listing files" width="380"><br>
  <em>Fig. 8 — PC1 connects (<code>ftp 192.168.0.98</code>), logs in as <code>student</code>, and <code>dir</code> lists the server's files.</em>
</p>

**Upload** a file with `put`, then `dir` again to confirm it's now on the server:

```text
ftp> put sampleFile.txt
ftp> dir
```

<p align="center">
  <img src="./img/SSH%26FTP/labB-put-command-dir-checking.png" alt="put uploading a file, then dir showing it on the server" width="380"><br>
  <em>Fig. 9 — <code>put sampleFile.txt</code> uploads from the PC; the next <code>dir</code> shows it now stored on the server.</em>
</p>

**Download** it back with `get`, try `delete`, then `quit`:

```text
ftp> get sampleFile.txt
ftp> delete sampleFile.txt
ftp> quit
```

<p align="center">
  <img src="./img/SSH%26FTP/labB-get-delete-quit-FTP.png" alt="get, delete, and quit on the FTP client" width="560"><br>
  <em>Fig. 10 — <code>get</code> downloads the file, <code>delete</code> removes it from the server, <code>quit</code> ends the session. A successful round-trip proves FTP works <em>and</em> that your S2 routing carries a real application across subnets.</em>
</p>

## Step B6 — Security wrap-up: encrypted vs plaintext

Switch to **Simulation Mode** and send a little of each kind of traffic, or just reflect on what you configured:

- **SSH (TCP port 22)** encrypts the **whole** management session — a sniffer sees only ciphertext.
- **FTP (TCP port 21 control, 20 data)** sends the **username, password, and file contents in plaintext** — anyone capturing the link can read them (the same plaintext-vs-encrypted contrast as HTTP vs HTTPS).

> ⚠️ That's the lesson: **SSH replaced Telnet, and SFTP/FTPS replaced plain FTP**, for the same reason — never send credentials in clear text on a network someone else might be listening to.

---

## Bonus — SSH & FTP on Real Machines

Packet Tracer shows the *idea*; here's how to let one **real** computer reach another. The whole thing splits into two roles — and the work is almost all on one side:

| Role | What it must do |
|:---|:---|
| **Server** — the machine you connect **to** | **Install/enable the service** (SSH or FTP), have a **login account**, and **open the firewall port**. This is where all the setup happens. |
| **Client** — the machine you connect **from** | **Just run the client command** (`ssh` / `sftp` / `ftp`) pointed at the server's IP and account. Every modern OS already has these — usually nothing to install. |

> ⚠️ Only on machines and networks **you own**. A reachable SSH/FTP server is a real attack surface — use strong passwords or keys, and don't expose it to the public internet without a reason.

### SSH — secure remote shell (+ encrypted file copy)

**On the SERVER** (enable the SSH service, then note the machine's IP via `ipconfig` / `ip addr`):

| OS | Server setup |
|:---|:---|
| **Windows 10/11** | Settings → System → **Optional features** → Add **OpenSSH Server**. Then in PowerShell (admin): `Start-Service sshd` and `Set-Service sshd -StartupType Automatic`. (The installer opens TCP 22 in the firewall.) |
| **macOS** | System Settings → General → **Sharing** → turn on **Remote Login** — that *is* the SSH server. (Optionally restrict "Allow access for" to chosen users.) CLI: `sudo systemsetup -setremotelogin on`. |
| **Linux** | `sudo apt install openssh-server` → `sudo systemctl enable --now ssh` → open the port: `sudo ufw allow 22`. |

**On the CLIENT** (the `ssh` command is built into Windows 10+/macOS/Linux — nothing to install):

```sh
ssh username@<server-ip>          # e.g. ssh alice@192.168.1.50
```

Accept the host-key fingerprint the first time. *(Stronger than passwords: on the client run `ssh-keygen -t ed25519`, then `ssh-copy-id username@<server-ip>` to install your key on the server.)*

### File transfer — prefer SFTP, fall back to FTP

The instant the **SSH server** above is running, it is *also* an **SFTP** server — encrypted file transfer with **no extra setup**. From the **client**:

```sh
sftp username@<server-ip>         # then:  put localfile   /   get remotefile
scp localfile username@<server-ip>:/path/     # one-shot copy
```

*Only if you specifically need a classic (plaintext) **FTP** server — trusted LAN only:*

**On the SERVER:**

| OS | FTP server setup |
|:---|:---|
| **Windows** | Turn on **IIS → FTP Server** in *Windows Features*, then create an FTP site in **IIS Manager** (bind a folder + user); allow **TCP 21**. |
| **macOS** | The built-in FTP server was **removed** in modern macOS — install one (`brew install vsftpd`) or just use SFTP. |
| **Linux** | `sudo apt install vsftpd` → set `write_enable=YES` in `/etc/vsftpd.conf` → `sudo systemctl enable --now vsftpd` → `sudo ufw allow 21`. |

**On the CLIENT:** the command-line `ftp <server-ip>`, or a GUI that speaks both FTP and SFTP — **FileZilla** (all OSes), **WinSCP** (Windows), **Cyberduck** (macOS).

> **Firewall is the #1 gotcha.** If the client gets "connection refused / timed out," either the server's service isn't running or its port is blocked — **22** for SSH/SFTP, **21** (plus data ports) for FTP.
>
> 💡 Same lesson as the lab: **SSH/SFTP encrypt everything; plain FTP and Telnet send your password in clear text.** In the real world, use **SSH/SFTP** and only fall back to FTP for legacy systems.

---

# Lab B Questions

**Try each one first, then click "Show answer".**

**Q1.** Why configure **SSH** instead of **Telnet** for managing the router?

<details>
<summary>💡 Show answer</summary>

**SSH encrypts** the entire session; **Telnet sends everything — including your password — in plaintext.** Anyone capturing the link during a Telnet login reads the admin credentials directly. `transport input ssh` removes Telnet from the vty lines so only encrypted logins are accepted.
</details>

**Q2.** Why must you set a **hostname** and **`ip domain-name`** *before* `crypto key generate rsa`?

<details>
<summary>💡 Show answer</summary>

The RSA key pair is **named `hostname.domain-name`**. With the default hostname `Router` and no domain set, IOS has no fully-qualified name to label the key, so it **refuses to generate** it. Setting both gives the key a unique identity (e.g. `R-CORE.lab.local`).
</details>

**Q3.** What does `transport input ssh` on `line vty 0 4` actually do?

<details>
<summary>💡 Show answer</summary>

It restricts the **5 virtual terminal (vty) lines** — the remote-login slots — to accept **SSH connections only**, refusing Telnet. (`login local` then tells those lines to authenticate against the local `username`/`secret` database.)
</details>

**Q4.** What **ports** do SSH and FTP use, and why did the PC1→server FTP work even though they're on different subnets?

<details>
<summary>💡 Show answer</summary>

**SSH = TCP 22.** **FTP = TCP 21 (control) + 20 (data).** PC1 (Engineering, `192.168.0.0/26`) and the FTP-server (Management, `192.168.0.96/28`) are different subnets, so the traffic is **routed through the 2911** — the same inter-subnet routing you built in Session 2 now carries a real application.
</details>

**Q5.** FTP moved your file fine — so what's the **security problem** with it, and what should you use instead?

<details>
<summary>💡 Show answer</summary>

Plain FTP sends the **login and the file contents in clear text**, so anyone sniffing the path can capture both. Use **SFTP** (file transfer over SSH) or **FTPS** (FTP over TLS) to encrypt the transfer — the same plaintext-vs-encrypted issue as HTTP vs HTTPS.
</details>

---

# Lab C — DHCP & DNS, continuing from Lab B

**Objective:** make the network **self-configuring**. **Lab C picks up exactly where Lab B left off — keep the same `.pkt`.** Right now every PC has a **static** address you typed by hand, and you reach the server by its **IP**. Now the router will hand out addresses with **DHCP**, and the server will resolve **names** with **DNS** — so a PC can open a browser to **`http://www.lab.local`** (or `ftp ftp.lab.local`) with no IP typed anywhere.

You'll add three things **on top of the existing network** (no new topology):

| Add | Where | Result |
|:---|:---|:---|
| **HTTP + DNS services + A records** | the existing **FTP-server** (a server runs many services at once) | `www.lab.local` / `ftp.lab.local` → `192.168.0.98` |
| **DHCP pools** (one per department) | **R-CORE** (the router from Lab B) | PCs auto-get IP, gateway, and DNS |
| **Static → DHCP** | **PC1–PC6** | no more hand-typed addresses |

> This builds directly on the [Wireshark Lab A](./WIRESHARK_GUIDE.md) analysis — you're now running the **server side** of the very DORA and DNS exchanges you captured there.

## Step C1 — Add a web page + DNS to the server

One box can serve **web pages**, **resolve names**, *and* serve files at once. We'll turn on **HTTP** (so there's a page to view) and **DNS** (so it has a name), then PCs can just open a browser to **`www.lab.local`**.

**1. Turn on the web server.** Click **`FTP-server` → Services tab → HTTP** and make sure **HTTP** is **On**. Packet Tracer already ships a default `index.html` ("Cisco Packet Tracer — Welcome…") — you can edit its text to confirm it's *your* page, then **Save**.

<p align="center">
  <!-- ![HTTP service](./img/labC-step1-http.png) -->
  <em>Fig. 11 — 📸 <code>img/labC-step1-http.png</code>: the HTTP service On with its default <code>index.html</code> page.</em>
</p>

**2. Turn on DNS and add the names.** Click **Services → DNS**, set the service **On**, and add two **A records** pointing at this server. For each: type the **Name** and **Address**, leave Type = **A Record**, and click **Add**.

| Name | Type | Address |
|:---|:---:|:---|
| `www.lab.local` | A | `192.168.0.98` |
| `ftp.lab.local` | A | `192.168.0.98` |

<p align="center">
  <img src="./img/DHCP%26DNS/labC-DNS-setting.png" alt="Adding an A record on the server's DNS service" width="760"><br>
  <em>Fig. 12 — DNS service turned <strong>On</strong>; enter Name + Address and click <strong>Add</strong> (here <code>ftp.lab.local → 192.168.0.98</code>).</em>
</p>

After adding both, the **Resource Records** list shows the two names resolving to the one server:

<p align="center">
  <img src="./img/DHCP%26DNS/labC-DNS-HTTP-table.png" alt="The DNS resource-records table with both A records" width="680"><br>
  <em>Fig. 13 — Both A records: <code>ftp.lab.local</code> and <code>www.lab.local</code> → <code>192.168.0.98</code>.</em>
</p>

## Step C2 — Configure DHCP pools on the router

Each department is a **different subnet**, so each needs its **own DHCP pool** (its own `network` and `default-router`). First **exclude** the static addresses — the three gateways and the server — then define one pool per department, all pointing clients at the new DNS server:

```ios
R-CORE# configure terminal
R-CORE(config)# ip dhcp excluded-address 192.168.0.1
R-CORE(config)# ip dhcp excluded-address 192.168.0.65
R-CORE(config)# ip dhcp excluded-address 192.168.0.97 192.168.0.98

! Engineering — 192.168.0.0/26
R-CORE(config)# ip dhcp pool ENG
R-CORE(dhcp-config)# network 192.168.0.0 255.255.255.192
R-CORE(dhcp-config)# default-router 192.168.0.1
R-CORE(dhcp-config)# dns-server 192.168.0.98
R-CORE(dhcp-config)# exit
! Marketing — 192.168.0.64/27
R-CORE(config)# ip dhcp pool MKT
R-CORE(dhcp-config)# network 192.168.0.64 255.255.255.224
R-CORE(dhcp-config)# default-router 192.168.0.65
R-CORE(dhcp-config)# dns-server 192.168.0.98
R-CORE(dhcp-config)# exit
! Management — 192.168.0.96/28
R-CORE(config)# ip dhcp pool MGT
R-CORE(dhcp-config)# network 192.168.0.96 255.255.255.240
R-CORE(dhcp-config)# default-router 192.168.0.97
R-CORE(dhcp-config)# dns-server 192.168.0.98
R-CORE(dhcp-config)# end
```

**What each command does.** The config has two parts: a few **global** exclusions, then one **pool block** per subnet.

| Command | Mode | What it does |
|:---|:---|:---|
| `ip dhcp excluded-address 192.168.0.1` | global config | **Reserves** the Engineering **gateway** so DHCP never leases it to a PC. (`.65` and `.97` do the same for the other two gateways.) |
| `ip dhcp excluded-address 192.168.0.97 192.168.0.98` | global config | The two-address form reserves a **range** — here the Management gateway **and** the server. Without this, DHCP could hand out `.98` and collide with the server. |
| `ip dhcp pool ENG` | global config | **Creates a pool** named `ENG` and drops you into pool-config mode (prompt changes to `(dhcp-config)#`). The name is just a label for *you*. |
| `network 192.168.0.0 255.255.255.192` | dhcp-config | The **subnet to lease from** — every address in `192.168.0.0/26` *except* the excluded ones and the network/broadcast addresses. The mask is also handed to clients. |
| `default-router 192.168.0.1` | dhcp-config | The **gateway** clients receive (**DHCP Option 3**). Must be this subnet's router interface, or PCs can't leave the subnet. |
| `dns-server 192.168.0.98` | dhcp-config | The **DNS resolver** clients receive (**DHCP Option 6**) — the same for all three pools, since one server resolves names for everyone. |
| `exit` | dhcp-config | Leaves pool-config so you can start the **next** pool. (`end` jumps straight back to privileged EXEC.) |

> 🔑 **How does the router know which pool to use?** You don't tell it. When a PC's broadcast **DHCP Discover** arrives on an interface (say `g0/1`, which has IP `192.168.0.65`), the router looks at **that interface's own subnet** and serves the pool whose `network` matches — here `MKT`. So **the pool's `network` must line up with a router interface's subnet**, and the `default-router` should be that same interface's IP. That's why one router can run three pools at once and each department gets the right gateway automatically.

**But how does it know which switch is ENG vs MKT vs MGT?** It doesn't — *not by name*. The router never sees "`SW-Eng`"; it only knows **which of its own interfaces a frame came in on**. The link is **the cabling**: each department's switch is wired to exactly **one** router interface, and you gave that interface an IP inside that department's subnet. So the chain is fixed by *how you plugged it in*:

| Department | Switch | → cabled to router | Interface IP | matches `network` | Pool served |
|:---|:---|:---:|:---|:---|:---:|
| Engineering | `SW-Eng` | **`g0/0`** | `192.168.0.1` | `192.168.0.0/26` | **ENG** |
| Marketing | `SW-Mkt` | **`g0/1`** | `192.168.0.65` | `192.168.0.64/27` | **MKT** |
| Management | `SW-Mgt` | **`g0/2`** | `192.168.0.97` | `192.168.0.96/28` | **MGT** |

Read a row left-to-right: a Marketing PC's Discover travels `SW-Mkt → g0/1`; the router sees it **arrived on `g0/1`**, whose `192.168.0.65` lives in `192.168.0.64/27`, so it answers from the **MKT** pool and hands back the `.65` gateway. The "department" is really just **a switch + a router interface + a subnet that all line up** — the names (`SW-Eng`, `ENG`) are only labels for *you*.

> ⚠️ **This is the #1 thing to get right.** If you cable `SW-Mgt` into `g0/0` by mistake, Management PCs will be offered **Engineering** addresses (wrong subnet, wrong gateway) and nothing will route. The switch, the router interface, and the pool's `network` must all describe the **same subnet**.

> 💡 **Why three pools (not one)?** A single pool serves a single `network`/gateway pair. Three departments = three subnets (`/26`, `/27`, `/28`) with three different gateways, so you need **three pools**. The `dns-server` line is identical in all three because one DNS server (`192.168.0.98`) answers for the whole company.

> ⚠️ **Order & prompt matter:** set the `excluded-address` lines **before** PCs request leases (or the router may already have handed one out), and remember `network`/`default-router`/`dns-server` only work **inside** a pool (`R-CORE(dhcp-config)#`) — *after* `ip dhcp pool …`.

<p align="center">
  <img src="./img/DHCP%26DNS/labC-step2-dhcp.png" alt="Router CLI after defining the three DHCP pools" width="760"><br>
  <em>Fig. 14 — The router CLI: the three excluded addresses, then one pool per department (ENG/MKT/MGT), each with its <code>network</code>, <code>default-router</code>, and the shared <code>dns-server 192.168.0.98</code>.</em>
</p>

## Step C3 — Switch the PCs to DHCP

On each PC: **Desktop → IP Configuration → DHCP**. Each PC drops its static address and **leases one from its department's pool**, plus the gateway and the DNS server (`192.168.0.98`).

<p align="center">
  <img src="./img/DHCP%26DNS/labC-step3-pcdhcp.png" alt="A PC's IP Configuration set to DHCP" width="560"><br>
  <em>Fig. 15 — PC1 set to <strong>DHCP</strong> — it leased <code>192.168.0.2</code> / <code>255.255.255.192</code> from the Engineering pool, no typing required.</em>
</p>

## Step C4 — Verify: auto-address & reach the server by name

First, on a PC's **Desktop → Command Prompt**, check what DHCP gave you:

1. `ipconfig /all` — confirm the **DHCP-leased** IPv4 address, the **gateway**, the **DHCP Server**, and the **DNS Server `192.168.0.98`** — all learned automatically.

<p align="center">
  <img src="./img/DHCP%26DNS/labC-IPconfig.png" alt="ipconfig /all output on PC1" width="640"><br>
  <em>Fig. 16 — <code>ipconfig /all</code>: IPv4 <code>192.168.0.2</code>, gateway <code>192.168.0.1</code>, <strong>DHCP Server</strong> <code>192.168.0.1</code>, and <strong>DNS Server</strong> <code>192.168.0.98</code>.</em>
</p>

2. `ping www.lab.local` — the **name** resolves (via DNS) to `192.168.0.98` and replies. (`ftp.lab.local` works too.)

<p align="center">
  <img src="./img/DHCP%26DNS/labC-ping-www.lab.local.png" alt="ping www.lab.local resolving to the server" width="540"><br>
  <em>Fig. 17 — <code>ping www.lab.local</code> resolves to <code>192.168.0.98</code> and gets 4 replies — DNS is working.</em>
</p>

## Step C5 — Open the browser and see the web page

Now the payoff — reach the site **by name** in a browser, exactly like the real web:

1. On a PC, open **Desktop → Web Browser**.
2. In the address bar type **`http://www.lab.local`** and press **Go**.
3. The page loads — with **no IP typed anywhere**.

<p align="center">
  <img src="./img/DHCP%26DNS/labC-step5-browser.png" alt="PC web browser showing the page at www.lab.local" width="680"><br>
  <em>Fig. 18 — PC1's <strong>Web Browser</strong> open at <code>http://www.lab.local</code> — the server's page loads, no IP typed.</em>
</p>

**The key idea — the client always asks DNS *first*, then the web server.** A browser can only open a connection to an **IP address**, but you gave it a **name**. So that one click triggered **two separate conversations**:

| # | From → To | Protocol · Port | Meaning |
|:---:|:---|:---|:---|
| 1 | PC → DNS server | **DNS** · UDP 53 | "What IP is `www.lab.local`?" |
| 2 | DNS server → PC | **DNS** reply | "It's `192.168.0.98`." |
| 3 | PC → Web server | **HTTP** · TCP 80 | "GET me the page at `/`" |
| 4 | Web server → PC | **HTTP** `200 OK` | the HTML page |

> 🔑 **Why ask DNS at all when the server is right there?** In *this* lab the DNS service and the web service happen to live on the **same box** (`192.168.0.98`) — but the PC **doesn't know that**. All it has is the *name*, so it **must** resolve it through DNS before it can connect. It's like looking a number up in a phone book before you can dial: name → number → call.

**See it happen (Simulation Mode):**

1. Switch to **Simulation** (bottom-right) → **Edit Filters** → keep only **DNS** and **HTTP** so nothing else clutters the view.

<p align="center">
  <img src="./img/DHCP%26DNS/labC-fliter-edit.png" alt="Edit Filters set to DNS and HTTP" width="700"><br>
  <em>Fig. 19 — <strong>Edit Filters</strong> → tick only <strong>DNS</strong> and <strong>HTTP</strong>.</em>
</p>

2. In the PC's Web Browser, type `http://www.lab.local` and click **Go**, then click **Play / Capture-Forward** and watch the order: a **DNS** packet leaves the PC **first**; its reply comes back; **then** an **HTTP** packet goes out.
3. Click each coloured packet → **PDU Information → OSI Model** to read its layers. The **DNS** reply is **UDP, source port 53**; the **HTTP** reply is **TCP, source port 80** — same server IP (`192.168.0.98`), two different services, and **DNS answered before HTTP could even start**:

<p align="center">
  <img src="./img/DHCP%26DNS/labC-pdu-dns-53.png" alt="PDU showing the DNS reply on UDP port 53" width="440">
  <img src="./img/DHCP%26DNS/labC-pdu-http-80.png" alt="PDU showing the HTTP reply on TCP port 80" width="440"><br>
  <em>Fig. 20 &amp; 21 — Left: the <strong>DNS</strong> reply (Layer 4 <strong>UDP src port 53</strong>, Layer 7 DNS). Right: the <strong>HTTP</strong> reply (Layer 4 <strong>TCP src port 80</strong>, Layer 7 HTTP). Both from <code>192.168.0.98</code> — the lookup first, the page second.</em>
</p>

> 💡 This is the whole point of DNS: it lets people use memorable **names** while computers keep working in **numbers** — and the lookup always happens *before* the real connection.

## Step C6 — Watch DORA in Simulation Mode

1. Click **Simulation** (bottom-right) and **Edit Filters** to keep **DHCP** (and DNS).
2. On a PC, toggle IP Configuration to **Static** then back to **DHCP** to trigger a fresh request.
3. Step through the **Discover → Offer → Request → ACK** packets — the very messages you read in [Wireshark Lab A](./WIRESHARK_GUIDE.md#step-a2--read-the-four-dora-messages), now produced by *your* router.

<p align="center">
  <img src="./img/DHCP%26DNS/labC-step6-dora.png" alt="DHCP packets stepping through Simulation Mode" width="700"><br>
  <em>Fig. 22 — Simulation Mode: a DHCP packet travelling PC1 → SW-Eng → Router2 in the Event List.</em>
</p>

Click a DHCP packet → **PDU Information** to confirm it's a real DHCP message: **UDP, server port 67 → client port 68**, sent from the router (`192.168.0.1`) to the broadcast address.

<p align="center">
  <img src="./img/DHCP%26DNS/labC-DORA-ack.png" alt="PDU of a DHCP packet showing UDP ports 67 and 68" width="460"><br>
  <em>Fig. 23 — A DHCP PDU: Layer 7 <strong>DHCP</strong>, Layer 4 <strong>UDP src 67 → dst 68</strong>, broadcast to <code>255.255.255.255</code> — exactly the DORA mechanics from Wireshark Lab A.</em>
</p>

---

# Lab C Questions

**Try each one first, then click "Show answer".**

**Q1.** Why must the three **gateways** (`.1`, `.65`, `.97`) and the **server** (`.98`) be **excluded** from the DHCP pools?

<details>
<summary>💡 Show answer</summary>

They're assigned **statically**. If a pool leased one of those addresses to a PC you'd get an **IP conflict** — two devices claiming one address (and you'd lose the gateway or the server). `ip dhcp excluded-address` reserves them so DHCP never hands them out.
</details>

**Q2.** Why does this network need **three** DHCP pools instead of one?

<details>
<summary>💡 Show answer</summary>

Each pool serves **one subnet** — its `network`/mask and its `default-router` are subnet-specific. The three departments are three different subnets (`/26`, `/27`, `/28`) with three different gateways, so each needs its own pool. The router matches a request to the right pool by the interface it arrives on.
</details>

**Q3.** A PC can now run `ftp ftp.lab.local`. What **two lookups** happen, in order?

<details>
<summary>💡 Show answer</summary>

First a **DNS** query resolves `ftp.lab.local` → `192.168.0.98` (you can't connect to a name, only an IP). Then the **FTP** client opens a TCP connection to that IP. Name resolution always precedes the data connection.
</details>

**Q4.** How did each PC learn the DNS server's address (`192.168.0.98`) without anyone typing it in?

<details>
<summary>💡 Show answer</summary>

From the **DHCP ACK** — the `dns-server 192.168.0.98` line in each pool is delivered as **DHCP Option 6**. The same ACK also carried the address (`network`) and gateway (`default-router`, Option 3). This is exactly the Option data you read in the Wireshark ACK in Lab A.
</details>

---

### Practice Exercises

Work through these to prove the build end-to-end. **Tick each box** as you finish it, and save a screenshot or note of the result (e.g. into `S3/img/`).

**Lab B — SSH & FTP:**
- [ ] **1. Harden + log in.** Configure SSH on the router, then `ssh -l admin 192.168.0.1` from a PC. *Record:* the `R-CORE#` prompt reached over SSH, and confirm `telnet 192.168.0.1` is **refused**.
- [ ] **2. Move a file across subnets.** From a PC in another department, `ftp 192.168.0.98`, log in as `student`, and `put` then `get` a file. *Record:* a successful round-trip (it crossed the `2911` router).
- [ ] **3. Why FTP is unsafe.** *Record:* in one line, why a sniffer can read your FTP login but not your SSH session — and what to use instead (**SFTP/FTPS**).

**Lab C — DHCP & DNS:**
- [ ] **4. Auto-address a PC.** Switch a PC to **DHCP**, then run `ipconfig /all`. *Record:* the leased IP, gateway, **DHCP Server**, and **DNS Server** it received — and which **pool** it came from.
- [ ] **5. Reach the server by name.** From a PC, `ping www.lab.local`, then open **`http://www.lab.local`** in the Web Browser. *Record:* the page loading with **no IP typed anywhere**.
- [ ] **6. See DNS-then-HTTP.** In **Simulation Mode** (filter DNS + HTTP), browse the name. *Record:* that a **DNS / UDP-53** packet fires **before** the **HTTP / TCP-80** packet.
- [ ] **Stretch — break the cabling on purpose.** Move `SW-Mgt`'s uplink to the **wrong** router interface (`g0/0` instead of `g0/2`) and renew a Management PC. *Record:* the **wrong (Engineering) address** it now leases, and explain — using the switch → interface → subnet → pool chain — why it broke.

> [!TIP]
> Treat each *Record* line as your deliverable — together they prove you can **secure** a router (SSH), **serve** files and pages (FTP/HTTP/DNS), and make a network **self-configure** (DHCP) — then diagnose it when the wiring is wrong.

---

# Next Steps

- Capture your own **FTP** login in Wireshark and read the username/password in **plaintext**, then confirm an **SSH** session is unreadable — live proof of what you configured in Lab B.
- **Homework (from the README):** the Kurose & Ross **DNS** Wireshark lab + a paragraph on **DNS poisoning/spoofing**.
- Save your `.pkt` file — Session 4 extends this same network with VLANs and routing.
