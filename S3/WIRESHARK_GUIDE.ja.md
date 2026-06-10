## Wireshark — ラボA：DHCP と DNS の解析

> 🌐 [English](./WIRESHARK_GUIDE.md) | **日本語**

> [セッション3 — ネットワークサービスとセキュリティ](./README.ja.md) のコンパニオンガイド。
> [README](./README.ja.md#ハンズオンラボ) の **ラボA** に取り組む**前**に読んでください。
> Wireshark が初めて？ インストールと基礎は [セッション1 Wireshark ガイド](../S1/WIRESHARK_GUIDE.ja.md)、フィルタと Follow-Stream は [セッション2 ガイド](../S2/WIRESHARK_GUIDE.ja.md) から。

---

- [Wireshark — ラボA：DHCP と DNS の解析](#wireshark--ラボadhcp-と-dns-の解析)
- [ラボA — DHCP と DNS の解析](#ラボa--dhcp-と-dns-の解析)
  - [背景：DORA と名前解決](#背景dora-と名前解決)
  - [ステップA1 — DHCP トレースを取得](#ステップa1--dhcp-トレースを取得)
  - [ステップA2 — 4つの DORA メッセージを読む](#ステップa2--4つの-dora-メッセージを読む)
  - [ステップA3 — nslookup で名前を探る](#ステップa3--nslookup-で名前を探る)
  - [ステップA4 — ブラウジング中に DNS をキャプチャ](#ステップa4--ブラウジング中に-dns-をキャプチャ)
  - [ステップA5 — DNS クエリと応答を調べる](#ステップa5--dns-クエリと応答を調べる)
  - [ラボA 設問](#ラボa-設問)
  - [演習問題](#演習問題)
- [次のステップ](#次のステップ)

---

## ラボA — DHCP と DNS の解析

**目的:** 機器が**アドレスを得る（DHCP/DORA）**様子と**名前を解決する（DNS）**様子をキャプチャする — 「ネットワークがjust work する」2つのサービス。

> *「Wireshark Lab: DHCP v9」「Wireshark Lab: DNS v9.0」（* Computer Networking: A Top-Down Approach*, J.F. Kurose・K.W. Ross の補足）を基に翻案。*

### 背景：DORA と名前解決

機器がネットワークに参加するとき、まだIPがないのでIPを求めます。**DHCP**（Dynamic Host Configuration Protocol）は **DORA** として覚える4ステップのやり取りで答えます:

| # | メッセージ | From → To | 目的 |
|:---:|:---|:---|:---|
| 1 | **D**iscover | クライアント → ブロードキャスト | 「DHCPサーバはいますか？」 |
| 2 | **O**ffer | サーバ → クライアント | 「はい — このアドレスをどうぞ。」 |
| 3 | **R**equest | クライアント → ブロードキャスト | 「その提示アドレスをもらいます。」 |
| 4 | **A**ck | サーバ → クライアント | 「あなたのものです — リース・ゲートウェイ・DNSサーバです。」 |

> **Request** と **ACK** のみが*必須*です — ホストが既存アドレスを単に**更新**する場合は Discover/Offer を省略できます。1つのやり取りの4メッセージは1つの **Transaction ID（xid）** を共有し、これがクライアントが Offer/ACK を自分の Discover/Request と対応づける方法です。

<p align="center">
  <img src="./img/wireshark_DHCP%26DNS/dhcp_client_server_interaction.png" alt="DHCP DORA のクライアント・サーバ相互作用の図" width="420"><br>
  <em>図1 — DHCPサーバと到着するクライアント間の DORA のやり取り: 各メッセージは提示アドレス（<code>yiaddr</code>）、共有トランザクションID、リース期間を運ぶ。</em>
</p>

DHCP は **UDP** 上で動きます — サーバポート **67**、クライアントポート **68**。アドレスを得た後、機器は **DNS**（Domain Name System、**UDP ポート53**）を使い、`gaia.cs.umass.edu` のような名前を、**ローカルDNSサーバ**への **クエリ → 応答** でIPアドレスに変換します。

---

**第1部 — DHCP：ホストがアドレスを得る様子を見る**

### ステップA1 — DHCP トレースを取得

*4つすべて*の DORA メッセージをキャプチャするには、完全な**解放＋更新**を強制する必要があります。まず Wireshark の **Capture → Options** でインターフェイス名を確認し、解放コマンドを実行し、**キャプチャを開始**してから更新コマンドを実行します:

```sh
# macOS  (en0 = あなたのインターフェイス)
sudo ipconfig set en0 none      # ← 解放、その後 Wireshark を開始
sudo ipconfig set en0 dhcp      # ← Discover→Offer→Request→ACK を誘発

# Windows
ipconfig /release               # ← アドレスを手放す、その後 Wireshark を開始
ipconfig /renew                 # ← DORA を誘発

# Linux  (eth0 = あなたのインターフェイス)
sudo ip addr flush dev eth0 && sudo dhclient -r   # ← その後 Wireshark を開始
sudo dhclient eth0                                # ← DORA を誘発
```

更新後、数秒待ってからキャプチャを**停止**します。

> ライブできない（または4つ揃わなかった）？ 著者のトレース **`dhcp-wireshark-trace1-1.pcapng`** を `gaia.cs.umass.edu/wireshark-labs/wireshark-traces-9e.zip` から使ってください。

**`dhcp`** フィルタを適用すると、4つのメッセージが順に — すべて**1つの Transaction ID**（ここでは `0x184eabb6`）を共有して — 見えるはずです:

<p align="center">
  <img src="./img/wireshark_DHCP%26DNS/labA-step1-dora.png" alt="Discover・Offer・Request・ACK を示す Wireshark dhcp フィルタ" width="900"><br>
  <em>図2 — <code>dhcp</code> フィルタで <strong>Discover → Offer → Request → ACK</strong> を表示。アドレスに注目: Discover と Request は<strong><code>0.0.0.0</code> から <code>255.255.255.255</code> ブロードキャストへ</strong>（クライアントにまだIPがない）、Offer と ACK は<strong>サーバ <code>172.20.0.1</code> からクライアント <code>172.20.2.203</code> へ</strong> — そして4つすべて xid <code>0x184eabb6</code> を共有。</em>
</p>

### ステップA2 — 4つの DORA メッセージを読む

フィルタに **`dhcp`**（古いビルドでは **`bootp`**）と入力し、各メッセージをクリックして主要フィールドを読みます:

| メッセージ | 送信元IP | 宛先IP | 読むべき注目フィールド |
|:---|:---|:---|:---|
| **Discover** | `0.0.0.0` *（まだアドレスなし！）* | `255.255.255.255` *（ブロードキャスト）* | **Transaction ID**、Options 内の **Parameter Request List** — クライアントが*欲しい*設定の一覧（サブネットマスク、ルータ、DNSサーバ、ドメイン…） |
| **Offer** | DHCPサーバのIP | クライアント / ブロードキャスト | 同じ Transaction ID、提示アドレス + オプション |
| **Request** | `0.0.0.0` | `255.255.255.255` | UDP **送信元ポート68 → 宛先ポート67**、同じ Transaction ID |
| **ACK** | DHCPサーバのIP | クライアント | **Your (client) IP Address**、**IP Address Lease Time**、**Router**（ゲートウェイ）、**Domain Name Server** |

4つそれぞれを Wireshark で展開したもの（すべて Transaction ID `0x184eabb6` を共有）:

**① Discover** — クライアント（`0.0.0.0`）が「DHCPサーバはいますか？」をブロードキャスト — UDP **68 → 67** の *Boot Request*、欲しいオプションを挙げる **Parameter Request List** 付き。

<p align="center">
  <img src="./img/wireshark_DHCP%26DNS/dhcp_discover.png" alt="Wireshark で展開した DHCP Discover パケット" width="840"><br>
  <em>図3 — <strong>Discover</strong>: Boot Request、Client IP <code>0.0.0.0</code>、Message Type = Discover、<strong>Parameter Request List</strong>（サブネットマスク、ルータ、DNS…）付き。</em>
</p>

**② Offer** — サーバが応答（*Boot Reply*、UDP **67 → 68**）し、**Your (client) IP address** フィールドにアドレスを提示、リース・マスク・ルータ・DNS オプション付き。

<p align="center">
  <img src="./img/wireshark_DHCP%26DNS/dhcp_offer.png" alt="Wireshark で展開した DHCP Offer パケット" width="840"><br>
  <em>図4 — <strong>Offer</strong>: <strong>Your (client) IP address <code>172.20.2.203</code></strong>、DHCP Server Identifier <code>192.168.99.2</code>、Lease Time・Subnet Mask・Router・Domain Name Server オプション付き。</em>
</p>

**③ Request** — クライアントがその提示を受け入れるとブロードキャストし、選んだアドレスを **Option 50（Requested IP Address）** に、選んだサーバを **Option 54** に反映。

<p align="center">
  <img src="./img/wireshark_DHCP%26DNS/dhcp_request.png" alt="Wireshark で展開した DHCP Request パケット" width="840"><br>
  <em>図5 — <strong>Request</strong>: Message Type = Request、<strong>Option 50 Requested IP <code>172.20.2.203</code></strong>、Option 54 DHCP Server Identifier <code>192.168.99.2</code>。</em>
</p>

**④ ACK** — サーバが確認（UDP **67 → 68**）、リースが正式に。**Options** を展開して **lease time**・**Router（ゲートウェイ）**・**Subnet Mask**・**Domain Name Server** を読む — ホストがネットワークの残りに到達するための値です。

<p align="center">
  <img src="./img/wireshark_DHCP%26DNS/dhcp_ack.png" alt="Wireshark で展開した DHCP ACK パケット" width="840"><br>
  <em>図6 — <strong>ACK</strong>: Message Type = ACK、Your IP <code>172.20.2.203</code>、Lease Time・Subnet Mask <code>255.255.240.0</code>・Router・Domain Name Server 付き。</em>
</p>

---

**第2部 — DNS：名前をアドレスに変える**

### ステップA3 — nslookup で名前を探る

キャプチャの前に、**`nslookup`**（Windows・macOS・Linux 組み込み）で DNS の感覚をつかみます。特定の**レコードタイプ**を DNS サーバに尋ねます:

```sh
nslookup www.iitb.ac.in        # Type=A  — 名前 → その IPv4（および IPv6/AAAA）アドレス
nslookup -type=NS umass.edu    # Type=NS — ドメインの権威ネームサーバ
nslookup 128.119.245.12        # 逆引き — IP → その名前（gaia.cs.umass.edu）
```

出力は**どのDNSサーバが答えたか**と、答えが**権威的**（ドメイン自身のサーバから）か**非権威的**（キャッシュから）かを示します。*(nslookup 出力の詳細は [セッション2 UDP ラボ](../S2/WIRESHARK_GUIDE.ja.md#nslookup-とは何かなぜ-udp-を使うのか) を参照。)*

<p align="center">
  <img src="./img/wireshark_DNS/labA-step3-nslookup.png" alt="nslookup の A・NS・逆引きクエリ" width="480"><br>
  <em>図7 — <code>nslookup www.iitb.ac.in</code>（A レコード → <code>103.21.124.133</code>）、<code>nslookup -type=NS umass.edu</code>（<code>ns1/ns2/ns3.umass.edu</code> ネームサーバ）、<code>128.119.245.12</code> の逆引き → <code>gaia.cs.umass.edu</code>。</em>
</p>

### ステップA4 — ブラウジング中に DNS をキャプチャ

1. **DNSキャッシュをクリア**して、ルックアップが実際にネットワークに出るようにします（キャッシュ済みレコードはクエリを*送りません*）:
   ```sh
   sudo killall -HUP mDNSResponder        # macOS
   ipconfig /flushdns                     # Windows
   sudo resolvectl flush-caches           # Linux (Ubuntu 22.04+)
   ```
2. **ブラウザ**のキャッシュをクリアし、**キャプチャを開始**。
3. **`http://gaia.cs.umass.edu/kurose_ross/`** にアクセスし、キャプチャを**停止**。
4. 他のルックアップが消えるよう、**この名前だけ**でフィルタ:
   ```text
   dns.qry.name == "gaia.cs.umass.edu"
   ```
   （素の **`dns`** でも動きますが、`dns.qry.name == "…"` はこの1ホストについて尋ねるパケット*だけ*を表示 — はるかに読みやすい。）

**パケット一覧の読み方。** 各行が1つのDNSメッセージで、**Info** 列がどれかを示します。色を下のスクリーンショットと対応づけてください:

| **Info** 列で | それが何か |
|:---|:---|
| `Standard query 0x8d6a A gaia.cs.umass.edu` | **クエリ** — PCが **A**（IPv4）レコードを*尋ねている* |
| `Standard query response 0x8d6a A … 128.119.245.12` | **応答** — サーバがアドレスで*答えている* |
| 一致する **`0x8d6a`** | クエリと応答を対にする **Transaction ID** |
| **送信元 / 宛先** | あなたのPC（`172.20.2.203`）↔ DNSサーバ（`192.168.99.2`） |

Wireshark はこの対をクロスリンクします: **クエリ**をクリックすると DNS 層に **`[Response In: 5794]`**、**応答**をクリックすると **`[Request In: 5791]`**。

<p align="center">
  <img src="./img/wireshark_DNS/1.png" alt="gaia.cs.umass.edu のクエリと応答を示す dns.qry.name フィルタ" width="780"><br>
  <em>図8 — <code>dns.qry.name == "gaia.cs.umass.edu"</code> でフィルタ: <strong>Standard query</strong> 行と対になる <strong>Standard query response</strong> 行（Transaction ID で対）。（2つの対が見えます — ブラウザが **A** クエリ*と* **HTTPS** タイプのクエリを送るため。）</em>
</p>

### ステップA5 — DNS クエリと応答を調べる

パケットをクリックし、中央ペインで層を展開します。上から下へ（Frame → Ethernet → IP → UDP → DNS）読みますが、重要なのは **UDP**（ポート）と **Domain Name System**（質問と答え）の2つです。

**クエリ — 図9。** **User Datagram Protocol** を展開: **宛先ポートは `53`**（ウェルノウンDNSポート）、送信元はPCが選んだランダムな高番号ポート。次に **Domain Name System (query)** を展開:

- **Transaction ID `0x8d6a`** — 応答が反映すべきタグ。
- **Flags `0x0100 Standard query`** — 「これは質問」の印。
- **Questions: 1 · Answer RRs: 0** — 1つ尋ね、まだ何も答えなし。
- **Queries → `gaia.cs.umass.edu: type A`** — 要求した正確な名前とレコードタイプ。
- **`[Response In: 5794]`** — Wireshark のクリック可能な応答へのリンク。

<p align="center">
  <img src="./img/wireshark_DNS/2.png" alt="DNS クエリ: UDP ポート53へ、Questions 1、Answer RRs 0" width="780"><br>
  <em>図9 — <code>gaia.cs.umass.edu</code> の<strong>クエリ</strong>: <strong>UDP → Dst Port 53</strong>、Transaction ID <code>0x8d6a</code>、<strong>Questions: 1, Answer RRs: 0</strong> — 尋ねていて、答えていない。</em>
</p>

**応答 — 図10。** 対応する応答（同じ `0x8d6a`）を選択。**UDP** では**送信元ポートが今 `53`** — ポートが**入れ替わった**（サーバ → あなた）。**Domain Name System (response)** で:

- 同じ **Transaction ID `0x8d6a`** — *あなたの*クエリに答えている証拠。
- **Flags `0x8180 Standard query response, No error`**。
- **Questions: 1 · Answer RRs: 1** — 質問を繰り返し、**1つの答え**を追加。
- **Answers → `gaia.cs.umass.edu: type A, addr 128.119.245.12`** — 実際の **名前 → IPv4** 対応。*これがブラウザを接続させる行です。*
- **`[Request In: 5791]`** と **`[Time: … ms]`** — 対応するクエリと、ルックアップにかかった時間。

<p align="center">
  <img src="./img/wireshark_DNS/4.png" alt="A レコードの答えを運ぶ DNS 応答" width="780"><br>
  <em>図10 — <strong>応答</strong>（Transaction ID <code>0x8d6a</code>）: UDP <strong>Src Port 53</strong>、<strong>Answer RRs: 1</strong> — <code>gaia.cs.umass.edu → 128.119.245.12</code> を解決する <strong>A</strong> レコード。</em>
</p>

> 💡 **すべての応答がアドレスを運ぶわけではない。** 現代のブラウザは **HTTPS タイプ**のクエリ（HTTP/3 のヒント）も撃ちます。このホストに `HTTPS` レコードがないため、応答は **Answer RRs: 0** に **Authority**（**SOA**）レコードを添えて戻ります — アドレスの代わりにゾーンの権威情報です。完全に正常で、名前を実際に解決したのは図10の **A** 応答です。

<p align="center">
  <img src="./img/wireshark_DNS/3.png" alt="権威 SOA レコードを持ち答えのない DNS 応答" width="780"><br>
  <em>図11 — ブラウザの **HTTPS タイプ** クエリの応答（Transaction ID <code>0x3e15</code>）: <strong>Answer RRs: 0, Authority RRs: 1</strong> — SOA レコードで、アドレスなし。</em>
</p>

### ラボA 設問

**まず自分で考えてから「答えを表示」をクリック。**

**Q1.** **DHCP Discover** は UDP と TCP のどちらで送られる？ その**送信元**と**宛先**IPアドレスの特別な点は？

<details>
<summary>💡 答えを表示</summary>

**UDP**（サーバポート67、クライアントポート68）。**送信元IPは `0.0.0.0`** — クライアントは*まだアドレスがない*ので本物を使えません。**宛先は `255.255.255.255`** — 制限付きブロードキャストアドレスで、クライアントはどのDHCPサーバのアドレスも知らず、セグメント上の全ホストに届ける必要があるからです。
</details>

**Q2.** **Discover・Offer・Request・ACK** を1つのやり取りとして結ぶフィールドは？

<details>
<summary>💡 答えを表示</summary>

**Transaction ID（xid）** — クライアントが Discover/Request に入れるランダムな番号で、サーバが Offer/ACK にコピーします。複数のクライアント（やサーバ）が同時に動いていても、クライアントが応答を自分の要求と対応づけられます。
</details>

**Q3.** **DHCP Request** の **UDP 送信元・宛先ポート**は？ なぜそうなる？

<details>
<summary>💡 答えを表示</summary>

**送信元ポート68（クライアント）→ 宛先ポート67（サーバ）。** DHCP はこれらの「ウェルノウン」ポートを固定し、サーバは常に67、クライアントは68で待ち受けます — クライアントがIPを持つ前は他に宛先指定の方法がないため必要です。（Offer/ACK は逆方向、67 → 68。）
</details>

**Q4.** **DHCP ACK** で、**割り当てクライアントIP**を運ぶフィールドは？ **リース時間**と**デフォルトゲートウェイ**を与えるオプションは？

<details>
<summary>💡 答えを表示</summary>

割り当てアドレスは **"Your (client) IP Address"** フィールドにあります。**IP Address Lease Time** オプションがリースの有効期間、**Router** オプションが**デフォルトゲートウェイ**（最初のホップのルータ）です。**Domain Name Server** オプションが DNS リゾルバを列挙します。
</details>

**Q5.** **DNS** のクエリ/応答は UDP と TCP のどちら？ クエリの**宛先ポート**と応答の**送信元ポート**は？

<details>
<summary>💡 答えを表示</summary>

**UDP**、両方ともポート **53** — クエリは宛先ポート53*へ*送られ、応答は送信元ポート53*から*来ます（戻りでポートが入れ替わる）。DNS はこうした小さな一発のルックアップに UDP を使い、大きな応答（ゾーン転送など）でのみ TCP に切り替えます。
</details>

**Q6.** あなたの DNS クエリはどの **IPアドレス**へ送られ、それは何のサーバ？

<details>
<summary>💡 答えを表示</summary>

あなたの**ローカル / デフォルト DNS サーバ** — DHCP **ACK**（Domain Name Server オプション）でホストが与えられたリゾルバアドレスです。コンピュータは常に*自分の*リゾルバに尋ね、リゾルバが代わりにルート・TLD・権威サーバに問い合わせる再帰的/反復的な作業を行います。
</details>

**Q7.** DNS の**クエリ**と**応答**で「questions」と「answers」はいくつ？

<details>
<summary>💡 答えを表示</summary>

**クエリ**は **1 question、0 answers**（尋ねていて、答えていない）。**応答**はその **1 question** を繰り返し、**1つ以上の answer** を追加（例 A レコード、しばしば AAAA も、ときに CNAME 連鎖）。Wireshark は DNS ヘッダの *Questions* / *Answer RRs* フィールドにこれらの数を表示します。
</details>

**Q8.** 名前 → **IPv4** を対応づけるレコードタイプは？ **`-type=NS`** クエリは何を返し、**DNSキャッシュ**は繰り返しのルックアップをどう変える？

<details>
<summary>💡 答えを表示</summary>

**`A`** レコードが名前 → IPv4（**`AAAA`** → IPv6）。**`-type=NS`** クエリはドメインの**権威ネームサーバ**を返します（たいていそのIPも「おまけ」で）。**キャッシュ**があると、さっき解決した名前の2回目のルックアップは **DNS クエリを一切送りません** — ホスト（やローカルサーバ）がレコードのTTLが切れるまで**リゾルバキャッシュ**から答えます。これが、クエリをワイヤ上で*見る*のにキャッシュのクリア（ステップA4）が必要な理由です。
</details>

---

### 演習問題

DHCP と DNS を定着させるため、自分でやってみましょう。終えたら**各チェックを付け**、結果のスクリーンショットやメモを保存してください。

- [ ] **1. 自分の DORA をキャプチャ。** キャプチャしながら解放＋更新を強制し、**`dhcp`** でフィルタ。*記録:* 4メッセージが共有する **Transaction ID**、そして Discover/Request が `0.0.0.0 → 255.255.255.255`、Offer/ACK がサーバ発であることを確認。
- [ ] **2. ACK を読む。** **DHCP ACK** を開き **Options** を展開。*記録:* **割り当てIP**（Your client IP）、**リース時間**、**Router**（ゲートウェイ）、**Domain Name Server** — これらが今ホストが使うもの。
- [ ] **3. 3つの nslookup。** `nslookup <名前>`、`nslookup -type=NS <ドメイン>`、IPの**逆引き**を実行。*記録:* 返った A アドレス、ドメインのネームサーバ、どの DNS サーバが答えたか（権威か否か）。
- [ ] **4. ライブの DNS ルックアップを捕える。** DNSキャッシュをフラッシュし、キャプチャして新しいサイトを閲覧、`dns.qry.name == "<そのホスト>"` でフィルタ。*記録:* **クエリの宛先ポート**と**応答の送信元ポート**、各々の **Questions/Answer RRs** 数。
- [ ] **5. 答えを見つける。** 対応する**応答**で **Answers** を開く。*記録:* **レコードタイプ**（A / AAAA / CNAME）と名前が解決した **IPアドレス**。
- [ ] **ストレッチ — キャッシュを証明。** 同じ `nslookup`（または同じ名前の閲覧）をキャプチャしながら**2回**実行。*記録:* なぜ**2回目**は **DNS パケットを送らない**か — そして何がキャッシュをクリアして送らせるか。

> [!TIP]
> 各 *記録* 行を成果物として扱いましょう — これらがそろえば、完全な **DORA** のやり取りを読み、**ACK オプション**を解読し、接続を可能にするアドレスまで **DNS クエリ → 応答** を追える証明になります。

---

## 次のステップ

- [Packet Tracer ラボB・C](./PACKET_TRACER_GUIDE.ja.md) で**サーバ側**を構築: ルータを **SSH** + **FTP** サービスで保護（ラボB）、次に自分の **DHCP・DNS サーバ**を動かして PC が自動構成し名前で閲覧（ラボC）。
- **宿題（README より）:** Kurose & Ross の **DNS** Wireshark ラボを完了し、**DNS ポイズニング／スプーフィング**の1段落要約を書く。
- [セッション2 Wireshark ガイド](../S2/WIRESHARK_GUIDE.ja.md) に戻り、**TCP** ハンドシェイク（HTTP）と、ここで見た **コネクションレスの UDP**（DHCP/DNS）を対比する。
