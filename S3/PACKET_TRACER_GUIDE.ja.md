## Cisco Packet Tracer — ラボB・C：SSH/FTP + DHCP/DNS サーバ

> 🌐 [English](./PACKET_TRACER_GUIDE.md) | **日本語**

> [セッション3 — ネットワークサービスとセキュリティ](./README.ja.md) のコンパニオンガイド。
> [README](./README.ja.md#ハンズオンラボ) の **ラボB** と **ラボC** に取り組む**前**に読んでください。
> Packet Tracer が初めて？ インストール・ウィンドウ案内・CLI基礎は [セッション1 Packet Tracer ガイド](../S1/PACKET_TRACER_GUIDE.ja.md) から。

---

- [ラボB — マルチサブネットネットワーク上の SSH と FTP](#ラボb--マルチサブネットネットワーク上の-ssh-と-ftp)
  - [ステップB1 — セッション2のネットワークを再利用し FTP サーバを追加](#ステップb1--セッション2のネットワークを再利用し-ftp-サーバを追加)
  - [ステップB2 — ルータに SSH を設定](#ステップb2--ルータに-ssh-を設定)
  - [ステップB3 — SSH でログイン（そしてなぜ Telnet ではないか）](#ステップb3--ssh-でログインそしてなぜ-telnet-ではないか)
  - [ステップB4 — FTP サーバを設定](#ステップb4--ftp-サーバを設定)
  - [ステップB5 — FTP クライアントでファイルを転送](#ステップb5--ftp-クライアントでファイルを転送)
  - [ステップB6 — セキュリティのまとめ：暗号化 vs 平文](#ステップb6--セキュリティのまとめ暗号化-vs-平文)
  - [ボーナス — 実機での SSH と FTP](#ボーナス--実機での-ssh-と-ftp)
    - [SSH — セキュアなリモートシェル（＋暗号化ファイルコピー）](#ssh--セキュアなリモートシェル暗号化ファイルコピー)
    - [ファイル転送 — SFTP を優先し FTP にフォールバック](#ファイル転送--sftp-を優先し-ftp-にフォールバック)
- [ラボB 設問](#ラボb-設問)
- [ラボC — DHCP と DNS（ラボBの続き）](#ラボc--dhcp-と-dnsラボbの続き)
  - [ステップC1 — サーバに Web ページと DNS を追加](#ステップc1--サーバに-web-ページと-dns-を追加)
  - [ステップC2 — ルータに DHCP プールを設定](#ステップc2--ルータに-dhcp-プールを設定)
  - [ステップC3 — PC を DHCP に切り替え](#ステップc3--pc-を-dhcp-に切り替え)
  - [ステップC4 — 確認：自動アドレスとサーバへ名前で到達](#ステップc4--確認自動アドレスとサーバへ名前で到達)
  - [ステップC5 — ブラウザを開いて Web ページを見る](#ステップc5--ブラウザを開いて-web-ページを見る)
  - [ステップC6 — シミュレーションモードで DORA を見る](#ステップc6--シミュレーションモードで-dora-を見る)
- [ラボC 設問](#ラボc-設問)
    - [演習問題](#演習問題)
- [次のステップ](#次のステップ)

---

### 構築するもの

この2つのラボは **[セッション2 ラボB](../S2/PACKET_TRACER_GUIDE.ja.md) で構築したマルチサブネットネットワーク** — 単一の **2911** ルータでルーティングされた3部門の会社（Engineering / Marketing / Management）— を**再利用**し、あらゆる実ネットワークが動かす**アプリケーション層サービス**を追加します:

- **ラボB** — **SSH**（ルータのセキュアなリモート管理）と **FTP**（ファイル転送サーバ）。
- **ラボC** — **DHCP**（ルータが PC を自動アドレス）と **DNS**（サーバに名前で到達）。**ラボCはラボBのファイルから直接続きます。**

| デバイス | 役割 | アドレス |
|:---|:---|:---|
| **Router2 (2911)** | SSH管理ルータ、後に **DHCP** サーバ | ゲートウェイ `192.168.0.1` / `.65` / `.97` |
| **FTP-server**（新規） | **FTP** サーバ、後に **DNS** サーバも | `192.168.0.98 /28`、gw `192.168.0.97` |
| **PC1–PC6**（S2 から） | SSH/FTP クライアント、後に **DHCP** クライアント | S2 のアドレッシング計画 |

> セッション2の `.pkt` を保存していれば開くだけ。なければ先に [S2 トポロジ](../S2/PACKET_TRACER_GUIDE.ja.md#ステップb2--トポロジを構築) を再構築 — これらのラボはルータインターフェイスと PC アドレスが既に動く（サブネット越え `ping` が成功する）前提です。

---

# ラボB — マルチサブネットネットワーク上の SSH と FTP

**⏱️ 約30分 · 目的:** ルータのリモート管理を **SSH** で保護し、**FTP** サーバを立て、サブネットをまたいでファイルを転送 — そして一方は盗聴しても安全で他方はそうでない理由を見る。

## ステップB1 — セッション2のネットワークを再利用し FTP サーバを追加

1. **セッション2の `.pkt` を開く**（Engineering / Marketing / Management ネットワーク）。
2. デバイスパネルで **End Devices → Server** を選びキャンバスに1台ドロップ、**`FTP-server`** と名付ける。

<p align="center">
  <img src="./img/SSH%26FTP/ftp-server-select-icon.png" alt="End Devices パネルから Server デバイスを選択" width="720"><br>
  <em>図1 — **End Devices** から **Server**（<code>Server-PT</code>）を選ぶ。</em>
</p>

3. **どのスイッチ？** **`SW-Mgt`** — **Management** 部門の `2960` スイッチに配線します。**デバイスは、自分が使う IP 範囲のサブネットのスイッチに挿す必要があり**、サーバは *Management* アドレス（`192.168.0.96/28`）を得ます。`SW-Eng`/`SW-Mkt` に配線すると誤ったサブネット（誤ゲートウェイ、到達不能）になります。**空きアクセスポート** — PC5/PC6 が `Fa0/1`/`Fa0/2` を使うので **`Fa0/3`** を使う:

   | リンク | From | To |
   |:---|:---|:---|
   | FTP サーバ → Management スイッチ | `FTP-server` `FastEthernet0` | `SW-Mgt` `Fa0/3` |

4. サーバに**静的 IPv4 アドレス**を与える（Config → FastEthernet0）: **`192.168.0.98`**、マスク `255.255.255.240`。

<p align="center">
  <img src="./img/SSH%26FTP/labB-ftp-server-IP-setting.png" alt="FTP-server の静的 IPv4 アドレスを設定" width="560"><br>
  <em>図2 — FTP-server 静的IP: <code>192.168.0.98</code>、マスク <code>255.255.255.240</code>（Management の <code>/28</code>）。</em>
</p>

5. **Default Gateway** を **`192.168.0.97`**（ルータの `g0/2`、Management ゲートウェイ）に、Config → Global Settings で設定。

<p align="center">
  <img src="./img/SSH%26FTP/labB-ftp-server-gateway-setting.png" alt="FTP-server のデフォルトゲートウェイを設定" width="600"><br>
  <em>図3 — FTP-server **Default Gateway** = <code>192.168.0.97</code>（ルータ <code>g0/2</code>）。</em>
</p>

完了するとトポロジはこうなります — 新サーバが Management サブネット上に座ります:

<p align="center">
  <img src="./img/SSH%26FTP/labB-step1-topology.png" alt="FTP-server を追加した3部門ネットワーク" width="780"><br>
  <em>図4 — Management サブネット上に <code>FTP-server</code>（<code>192.168.0.98</code>）を置いた S2 の3部門ネットワーク。</em>
</p>

## ステップB2 — ルータに SSH を設定

既定で Cisco ルータは **Telnet** しか提供せず、これはパスワードを含むすべてを**平文**で送ります。代わりに **SSH** を要求します。SSH は暗号化の前に **ホスト名**・**ドメイン名**（2つで暗号鍵を命名）・**RSA鍵** が必要です:

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

| コマンド | なぜ必要か |
|:---|:---|
| `hostname` + `ip domain-name` | RSA鍵は `hostname.domain` で命名 — 両方なしでは SSH が鍵生成を**拒否**。 |
| `crypto key generate rsa`（1024+） | セッションを**暗号化**する公開/秘密鍵ペアを作成。 |
| `username … secret` + `enable secret` | ローカルログインアカウント（SSH はラインパスワードでなく実資格情報が必要）。 |
| `ip ssh version 2` | より強力な **SSHv2** を強制。 |
| `line vty 0 4` → `transport input ssh` | 5つの仮想端末ラインが **SSH のみ**を受け付け — Telnet を拒否。 |
| `login local` | vty ログインをローカル `username` データベースで認証。 |

<p align="center">
  <img src="./img/SSH%26FTP/labB-router-cli-ssh-setting.png" alt="SSHv2 有効化後のルータCLI" width="600"><br>
  <em>図5 — ルータCLI: ホスト名/ドメイン設定、RSA鍵生成（<code>R-CORE.lab.local</code>、1024ビット）、SSHv2 有効化、vty ラインを SSH に限定。</em>
</p>

## ステップB3 — SSH でログイン（そしてなぜ Telnet ではないか）

**PC1** の **Desktop → Command Prompt**（Engineering）から、`admin` としてルータに接続:

```text
PC> ssh -l admin 192.168.0.1
Password: cisco123
R-CORE>
```

これでルータ上のリモートな**暗号化**セッションができました。ルータ上で `show ip ssh` と `show ssh` で確認。

> 💡 **Telnet を試して失敗を見る:** `telnet 192.168.0.1` は**拒否**されます — `transport input ssh` が vty ラインから Telnet を外したからです。その拒否*こそ*セキュリティ改善: Telnet なら `admin` パスワードを平文でワイヤに流していたはずです。

<p align="center">
  <img src="./img/SSH%26FTP/labB-step3-ssh-login.png" alt="PC1 がルータへ SSH セッションを開く" width="460"><br>
  <em>図6 — PC1 から: <code>ssh -l admin 192.168.0.1</code> → <code>R-CORE#</code> プロンプト、リモートの暗号化セッション。</em>
</p>

## ステップB4 — FTP サーバを設定

**`FTP-server` → Services タブ → FTP** をクリック:

1. **FTP サービスをオン**に。
2. ユーザアカウントを追加 — **ユーザ名 `student`**、**パスワード `ftppass`** — 権限 **Write, Read, Delete, Rename, List**（`RWDNL` と表示）にチェック。
3. **+ (Add)** をクリック。

<p align="center">
  <img src="./img/SSH%26FTP/labB-step4-ftp-server%E3%83%BCsetting.png" alt="student アカウントと権限付きで FTP サービスをオン" width="760"><br>
  <em>図7 — FTP サービス **On**、ユーザ <code>student</code> / <code>ftppass</code> を **Write/Read/Delete/Rename/List** で追加し **Add**。</em>
</p>

## ステップB5 — FTP クライアントでファイルを転送

**PC1**（Engineering）— サーバとは*別*サブネット — から **Desktop → Command Prompt** を開き、組み込みの FTP クライアントを実行。トラフィックは `2911` を越えて Management サブネットへルーティングされます。ログインし、`dir` でサーバのファイルを一覧:

```text
PC> ftp 192.168.0.98
Username: student
Password: ftppass
ftp> dir
```

<p align="center">
  <img src="./img/SSH%26FTP/labB-FTP-PC1-connecting.png" alt="PC1 が FTP サーバにログインしファイルを一覧" width="380"><br>
  <em>図8 — PC1 が接続（<code>ftp 192.168.0.98</code>）、<code>student</code> でログイン、<code>dir</code> がサーバのファイルを一覧。</em>
</p>

`put` でファイルを**アップロード**し、再度 `dir` でサーバ上にあることを確認:

```text
ftp> put sampleFile.txt
ftp> dir
```

<p align="center">
  <img src="./img/SSH%26FTP/labB-put-command-dir-checking.png" alt="put でファイルをアップロードし dir でサーバ上に表示" width="380"><br>
  <em>図9 — <code>put sampleFile.txt</code> が PC からアップロード、次の <code>dir</code> がサーバに保存されたことを表示。</em>
</p>

`get` で**ダウンロード**し直し、`delete` を試し、`quit`:

```text
ftp> get sampleFile.txt
ftp> delete sampleFile.txt
ftp> quit
```

<p align="center">
  <img src="./img/SSH%26FTP/labB-get-delete-quit-FTP.png" alt="FTP クライアントでの get・delete・quit" width="560"><br>
  <em>図10 — <code>get</code> がファイルをダウンロード、<code>delete</code> がサーバから削除、<code>quit</code> がセッション終了。往復成功は FTP が動くこと*かつ*S2 のルーティングが実アプリをサブネット越しに運ぶことを証明。</em>
</p>

## ステップB6 — セキュリティのまとめ：暗号化 vs 平文

**シミュレーションモード**に切り替えて各種トラフィックを少し送るか、設定したことを振り返ります:

- **SSH（TCP ポート22）** は管理セッション**全体**を暗号化 — 盗聴者には暗号文しか見えません。
- **FTP（TCP ポート21 制御、20 データ）** は**ユーザ名・パスワード・ファイル内容を平文**で送る — リンクをキャプチャすれば誰でも読めます（HTTP vs HTTPS と同じ平文 vs 暗号化の対比）。

> ⚠️ 教訓: **SSH が Telnet を、SFTP/FTPS が平文 FTP を置き換えた**のは同じ理由 — 誰かが聞いているかもしれないネットワークで資格情報を平文で送らないこと。

---

## ボーナス — 実機での SSH と FTP

Packet Tracer は*考え方*を示します。ここでは1台の**実**コンピュータが別の1台に到達する方法を示します。全体は2つの役割に分かれ、作業はほぼ片側に集中します:

| 役割 | やるべきこと |
|:---|:---|
| **サーバ** — 接続される**先**のマシン | **サービス（SSH または FTP）をインストール/有効化**し、**ログインアカウント**を持ち、**ファイアウォールのポートを開く**。設定はすべてここ。 |
| **クライアント** — 接続する**元**のマシン | サーバの IP とアカウントを指して **クライアントコマンド（`ssh` / `sftp` / `ftp`）を実行するだけ**。現代のOSはどれも標準装備 — 通常インストール不要。 |

> ⚠️ **自分が所有する**マシンとネットワークでのみ。到達可能な SSH/FTP サーバは実際の攻撃対象面です — 強いパスワードや鍵を使い、理由なく公衆インターネットに晒さないこと。

### SSH — セキュアなリモートシェル（＋暗号化ファイルコピー）

**サーバ側**（SSH サービスを有効化し、`ipconfig` / `ip addr` でマシンの IP を確認）:

| OS | サーバ設定 |
|:---|:---|
| **Windows 10/11** | 設定 → システム → **オプション機能** → **OpenSSH Server** を追加。次に PowerShell（管理者）で: `Start-Service sshd` と `Set-Service sshd -StartupType Automatic`。（インストーラがファイアウォールで TCP 22 を開く。） |
| **macOS** | システム設定 → 一般 → **共有** → **リモートログイン**をオン — それ*が* SSH サーバ。（任意で「アクセスを許可」を選んだユーザに制限。）CLI: `sudo systemsetup -setremotelogin on`。 |
| **Linux** | `sudo apt install openssh-server` → `sudo systemctl enable --now ssh` → ポートを開く: `sudo ufw allow 22`。 |

**クライアント側**（`ssh` コマンドは Windows 10+/macOS/Linux に組み込み — インストール不要）:

```sh
ssh username@<server-ip>          # 例 ssh alice@192.168.1.50
```

初回はホスト鍵フィンガープリントを承認。*(パスワードより強力: クライアントで `ssh-keygen -t ed25519` を実行し、`ssh-copy-id username@<server-ip>` で鍵をサーバに導入。)*

### ファイル転送 — SFTP を優先し FTP にフォールバック

上記の **SSH サーバ**が動いた瞬間、それは*同時に* **SFTP** サーバ — **追加設定なし**の暗号化ファイル転送 — でもあります。**クライアント**から:

```sh
sftp username@<server-ip>         # その後:  put localfile   /   get remotefile
scp localfile username@<server-ip>:/path/     # 一発コピー
```

*古典的な（平文）**FTP** サーバが特に必要な場合のみ — 信頼できる LAN のみ:*

**サーバ側:**

| OS | FTP サーバ設定 |
|:---|:---|
| **Windows** | *Windows の機能* で **IIS → FTP Server** をオンにし、**IIS マネージャ**で FTP サイトを作成（フォルダ + ユーザをバインド）、**TCP 21** を許可。 |
| **macOS** | 組み込み FTP サーバは現代の macOS で**削除**済み — 1つインストール（`brew install vsftpd`）するか SFTP を使う。 |
| **Linux** | `sudo apt install vsftpd` → `/etc/vsftpd.conf` で `write_enable=YES` → `sudo systemctl enable --now vsftpd` → `sudo ufw allow 21`。 |

**クライアント側:** コマンドラインの `ftp <server-ip>`、または FTP と SFTP 両対応の GUI — **FileZilla**（全OS）、**WinSCP**（Windows）、**Cyberduck**（macOS）。

> **ファイアウォールが最大の落とし穴。** クライアントが「connection refused / timed out」なら、サーバのサービスが動いていないかポートがブロックされています — SSH/SFTP は **22**、FTP は **21**（＋データポート）。
>
> 💡 ラボと同じ教訓: **SSH/SFTP はすべてを暗号化、平文 FTP と Telnet はパスワードを平文で送る。** 実世界では **SSH/SFTP** を使い、FTP はレガシーシステムにのみフォールバック。

---

# ラボB 設問

**まず自分で考えてから「答えを表示」をクリック。**

**Q1.** ルータ管理に **Telnet** でなく **SSH** を設定するのはなぜ？

<details>
<summary>💡 答えを表示</summary>

**SSH はセッション全体を暗号化**します。**Telnet はパスワードを含むすべてを平文で送ります。** Telnet ログイン中にリンクをキャプチャすれば管理者資格情報を直接読めます。`transport input ssh` が vty ラインから Telnet を外し、暗号化ログインだけを受け付けます。
</details>

**Q2.** なぜ `crypto key generate rsa` の*前*に **ホスト名** と **`ip domain-name`** を設定する必要がある？

<details>
<summary>💡 答えを表示</summary>

RSA 鍵ペアは **`hostname.domain-name` で命名**されます。既定ホスト名 `Router` でドメイン未設定だと、IOS は鍵にラベルする完全修飾名がないため**生成を拒否**します。両方を設定すると鍵に一意の身元（例 `R-CORE.lab.local`）が与えられます。
</details>

**Q3.** `line vty 0 4` の `transport input ssh` は実際に何をする？

<details>
<summary>💡 答えを表示</summary>

**5つの仮想端末（vty）ライン** — リモートログインの枠 — を **SSH 接続のみ**受け付けるよう制限し、Telnet を拒否します。（続く `login local` は、それらのラインにローカル `username`/`secret` データベースで認証するよう指示。）
</details>

**Q4.** SSH と FTP はどの**ポート**を使い、PC1→サーバの FTP は別サブネットなのになぜ動いた？

<details>
<summary>💡 答えを表示</summary>

**SSH = TCP 22。** **FTP = TCP 21（制御）+ 20（データ）。** PC1（Engineering、`192.168.0.0/26`）と FTP-server（Management、`192.168.0.96/28`）は別サブネットなので、トラフィックは **2911 を通してルーティング**されます — セッション2で構築した同じサブネット間ルーティングが今や実アプリを運びます。
</details>

**Q5.** FTP はファイルをちゃんと運んだ — では何が**セキュリティ問題**で、代わりに何を使うべき？

<details>
<summary>💡 答えを表示</summary>

平文 FTP は**ログインとファイル内容を平文**で送るので、経路を盗聴すれば両方を捕えられます。転送を暗号化するには **SFTP**（SSH 上のファイル転送）または **FTPS**（TLS 上の FTP）を使います — HTTP vs HTTPS と同じ平文 vs 暗号化の問題です。
</details>

---

# ラボC — DHCP と DNS（ラボBの続き）

**目的:** ネットワークを**自己構成可能**にする。**ラボCはラボBの終わったところからそのまま続きます — 同じ `.pkt` を使う。** 今は各 PC に手で打った**静的**アドレスがあり、サーバには **IP** で到達します。これからルータが **DHCP** でアドレスを配り、サーバが **DNS** で**名前**を解決 — PC がブラウザで **`http://www.lab.local`**（や `ftp ftp.lab.local`）を、IP をどこにも打たずに開けるようにします。

**既存ネットワークの上に**3つを追加します（新トポロジなし）:

| 追加 | 場所 | 結果 |
|:---|:---|:---|
| **HTTP + DNS サービス + A レコード** | 既存の **FTP-server**（1サーバが複数サービスを同時に動かす） | `www.lab.local` / `ftp.lab.local` → `192.168.0.98` |
| **DHCP プール**（部門ごとに1つ） | **R-CORE**（ラボBのルータ） | PC が IP・ゲートウェイ・DNS を自動取得 |
| **静的 → DHCP** | **PC1–PC6** | 手打ちアドレスはもう不要 |

> これは [Wireshark ラボA](./WIRESHARK_GUIDE.ja.md) の解析の上に直接構築されます — そこでキャプチャした DORA と DNS のやり取りの**サーバ側**を今動かしています。

## ステップC1 — サーバに Web ページと DNS を追加

1台で **Web ページの提供**・**名前解決**・*かつ*ファイル提供を同時にできます。**HTTP**（見るページがあるように）と **DNS**（名前を持つように）をオンにすれば、PC はブラウザで **`www.lab.local`** を開くだけで済みます。

**1. Web サーバをオンにする。** **`FTP-server` → Services タブ → HTTP** をクリックし **HTTP** が **On** であることを確認。Packet Tracer は既定の `index.html`（「Cisco Packet Tracer — Welcome…」）を同梱 — テキストを編集して*自分の*ページだと確認し、**Save**。

<p align="center">
  <!-- ![HTTP service](./img/labC-step1-http.png) -->
  <em>図11 — 📸 <code>img/labC-step1-http.png</code>: 既定の <code>index.html</code> ページ付きでオンの HTTP サービス。</em>
</p>

**2. DNS をオンにして名前を追加。** **Services → DNS** をクリックし、サービスを **On** にして、このサーバを指す **A レコード**を2つ追加。各々: **Name** と **Address** を入力、Type = **A Record** のまま、**Add** をクリック。

| Name | Type | Address |
|:---|:---:|:---|
| `www.lab.local` | A | `192.168.0.98` |
| `ftp.lab.local` | A | `192.168.0.98` |

<p align="center">
  <img src="./img/DHCP%26DNS/labC-DNS-setting.png" alt="サーバの DNS サービスに A レコードを追加" width="760"><br>
  <em>図12 — DNS サービスを **On**、Name + Address を入力し **Add**（ここでは <code>ftp.lab.local → 192.168.0.98</code>）。</em>
</p>

両方追加すると、**Resource Records** 一覧に2つの名前が1台のサーバに解決されることが表示されます:

<p align="center">
  <img src="./img/DHCP%26DNS/labC-DNS-HTTP-table.png" alt="2つの A レコードを持つ DNS リソースレコード表" width="680"><br>
  <em>図13 — 2つの A レコード: <code>ftp.lab.local</code> と <code>www.lab.local</code> → <code>192.168.0.98</code>。</em>
</p>

## ステップC2 — ルータに DHCP プールを設定

各部門は**別サブネット**なので、それぞれ**独自の DHCP プール**（独自の `network` と `default-router`）が必要です。まず静的アドレス — 3つのゲートウェイとサーバ — を**除外**し、次に部門ごとに1プールを定義し、すべてクライアントを新 DNS サーバに向けます:

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

**各コマンドの役割。** 設定は2部構成: いくつかの**グローバル**除外、次にサブネットごとの**プールブロック**1つ。

| コマンド | モード | 役割 |
|:---|:---|:---|
| `ip dhcp excluded-address 192.168.0.1` | グローバル設定 | Engineering の**ゲートウェイ**を**予約**し DHCP が PC にリースしないように。（`.65`・`.97` も他2ゲートウェイで同様。） |
| `ip dhcp excluded-address 192.168.0.97 192.168.0.98` | グローバル設定 | 2アドレス形式は**範囲**を予約 — ここでは Management ゲートウェイ**と**サーバ。これなしだと DHCP が `.98` を配ってサーバと衝突し得る。 |
| `ip dhcp pool ENG` | グローバル設定 | `ENG` という**プールを作成**しプール設定モードへ（プロンプトが `(dhcp-config)#` に）。名前は*あなた*用のラベルだけ。 |
| `network 192.168.0.0 255.255.255.192` | dhcp-config | **リース元のサブネット** — 除外分とネットワーク/ブロードキャストを*除く* `192.168.0.0/26` の全アドレス。マスクもクライアントに渡される。 |
| `default-router 192.168.0.1` | dhcp-config | クライアントが受け取る**ゲートウェイ**（**DHCP Option 3**）。このサブネットのルータインターフェイスでないと PC がサブネットを出られない。 |
| `dns-server 192.168.0.98` | dhcp-config | クライアントが受け取る **DNS リゾルバ**（**DHCP Option 6**）— 1サーバが全員の名前を解決するので3プールとも同じ。 |
| `exit` | dhcp-config | プール設定を抜けて**次の**プールを始められる。（`end` は privileged EXEC へ一気に戻る。） |

> 🔑 **ルータはどのプールを使うかどう知る？** あなたは教えません。PC のブロードキャスト **DHCP Discover** があるインターフェイス（例 IP `192.168.0.65` の `g0/1`）に着くと、ルータは**そのインターフェイス自身のサブネット**を見て `network` が一致するプール — ここでは `MKT` — を提供します。だから**プールの `network` はルータインターフェイスのサブネットと揃え**、`default-router` はその同じインターフェイスの IP にすべきです。これが1台のルータが3プールを同時に動かし各部門が正しいゲートウェイを自動で得る理由です。

**ではどのスイッチが ENG か MKT か MGT かどう知る？** 知りません — *名前ではなく*。ルータは「`SW-Eng`」を一切見ず、**自分のどのインターフェイスにフレームが着いたか**だけを知ります。つながりは**配線**です: 各部門のスイッチは正確に**1つ**のルータインターフェイスに配線され、そのインターフェイスにその部門のサブネット内の IP を与えました。だから連鎖は*どう挿したか*で固定されます:

| 部門 | スイッチ | → ルータへ配線 | インターフェイス IP | 一致する `network` | 提供プール |
|:---|:---|:---:|:---|:---|:---:|
| Engineering | `SW-Eng` | **`g0/0`** | `192.168.0.1` | `192.168.0.0/26` | **ENG** |
| Marketing | `SW-Mkt` | **`g0/1`** | `192.168.0.65` | `192.168.0.64/27` | **MKT** |
| Management | `SW-Mgt` | **`g0/2`** | `192.168.0.97` | `192.168.0.96/28` | **MGT** |

行を左から右に読む: Marketing PC の Discover は `SW-Mkt → g0/1` を通り、ルータはそれが **`g0/1` に着いた**と見て、その `192.168.0.65` は `192.168.0.64/27` にいるので **MKT** プールから答え、`.65` ゲートウェイを返します。「部門」は実は**揃ったスイッチ + ルータインターフェイス + サブネット**にすぎず、名前（`SW-Eng`・`ENG`）は*あなた*用のラベルだけです。

> ⚠️ **これが最重要ポイント。** `SW-Mgt` を誤って `g0/0` に配線すると、Management PC は **Engineering** アドレス（誤サブネット、誤ゲートウェイ）を提示され、何もルーティングされません。スイッチ・ルータインターフェイス・プールの `network` はすべて**同じサブネット**を表す必要があります。

> 💡 **なぜ3プール（1つでなく）？** 単一プールは単一の `network`/ゲートウェイ対を提供します。3部門 = 3サブネット（`/26`・`/27`・`/28`）で3つの異なるゲートウェイなので**3プール**必要です。`dns-server` 行が3つとも同一なのは、1つの DNS サーバ（`192.168.0.98`）が全社の名前を答えるからです。

> ⚠️ **順序とプロンプトが重要:** `excluded-address` 行は PC がリースを要求する**前**に設定（さもないとルータが既に配っているかも）、`network`/`default-router`/`dns-server` はプール**内**（`R-CORE(dhcp-config)#`）— `ip dhcp pool …` の*後* — でのみ効くことを忘れずに。

<p align="center">
  <img src="./img/DHCP%26DNS/labC-step2-dhcp.png" alt="3つの DHCP プール定義後のルータCLI" width="760"><br>
  <em>図14 — ルータCLI: 3つの除外アドレス、次に部門ごとのプール（ENG/MKT/MGT）、各々 <code>network</code>・<code>default-router</code>・共有の <code>dns-server 192.168.0.98</code> 付き。</em>
</p>

## ステップC3 — PC を DHCP に切り替え

各 PC で: **Desktop → IP Configuration → DHCP**。各 PC は静的アドレスを捨て、**自部門のプールから1つをリース**し、ゲートウェイと DNS サーバ（`192.168.0.98`）も得ます。

<p align="center">
  <img src="./img/DHCP%26DNS/labC-step3-pcdhcp.png" alt="DHCP に設定した PC の IP Configuration" width="560"><br>
  <em>図15 — PC1 を **DHCP** に設定 — Engineering プールから <code>192.168.0.2</code> / <code>255.255.255.192</code> をリース、入力不要。</em>
</p>

## ステップC4 — 確認：自動アドレスとサーバへ名前で到達

まず PC の **Desktop → Command Prompt** で DHCP が何をくれたか確認:

1. `ipconfig /all` — **DHCP リース**の IPv4 アドレス・**ゲートウェイ**・**DHCP Server**・**DNS Server `192.168.0.98`** がすべて自動で学習されたことを確認。

<p align="center">
  <img src="./img/DHCP%26DNS/labC-IPconfig.png" alt="PC1 の ipconfig /all 出力" width="640"><br>
  <em>図16 — <code>ipconfig /all</code>: IPv4 <code>192.168.0.2</code>、ゲートウェイ <code>192.168.0.1</code>、**DHCP Server** <code>192.168.0.1</code>、**DNS Server** <code>192.168.0.98</code>。</em>
</p>

2. `ping www.lab.local` — **名前**が（DNS 経由で）`192.168.0.98` に解決され応答。（`ftp.lab.local` も同様。）

<p align="center">
  <img src="./img/DHCP%26DNS/labC-ping-www.lab.local.png" alt="ping www.lab.local がサーバに解決" width="540"><br>
  <em>図17 — <code>ping www.lab.local</code> が <code>192.168.0.98</code> に解決し4応答 — DNS が動作。</em>
</p>

## ステップC5 — ブラウザを開いて Web ページを見る

いよいよご褒美 — 実際の Web のように、ブラウザでサイトに**名前で**到達:

1. PC で **Desktop → Web Browser** を開く。
2. アドレスバーに **`http://www.lab.local`** と入力し **Go**。
3. ページが読み込まれる — **IP をどこにも打たずに**。

<p align="center">
  <img src="./img/DHCP%26DNS/labC-step5-browser.png" alt="www.lab.local のページを表示する PC の Web ブラウザ" width="680"><br>
  <em>図18 — PC1 の **Web Browser** が <code>http://www.lab.local</code> を開く — サーバのページが読み込まれ、IP は未入力。</em>
</p>

**核心 — クライアントは常にまず DNS に尋ね、次に Web サーバに尋ねる。** ブラウザは **IP アドレス**にしか接続を開けませんが、あなたは**名前**を与えました。だからその1クリックが**2つの別々の会話**を引き起こしました:

| # | From → To | プロトコル · ポート | 意味 |
|:---:|:---|:---|:---|
| 1 | PC → DNS サーバ | **DNS** · UDP 53 | 「`www.lab.local` の IP は？」 |
| 2 | DNS サーバ → PC | **DNS** 応答 | 「`192.168.0.98` です。」 |
| 3 | PC → Web サーバ | **HTTP** · TCP 80 | 「`/` のページを GET して」 |
| 4 | Web サーバ → PC | **HTTP** `200 OK` | HTML ページ |

> 🔑 **サーバがそこにあるのになぜわざわざ DNS に尋ねる？** *この*ラボでは DNS サービスと Web サービスがたまたま**同じ箱**（`192.168.0.98`）にいます — でも PC は**それを知りません**。持っているのは*名前*だけなので、接続前に **DNS で解決しなければなりません**。電話する前に電話帳で番号を引くのと同じ: 名前 → 番号 → 通話。

**実際に見る（シミュレーションモード）:**

1. **Simulation**（右下）に切り替え → **Edit Filters** → 他が散らからないよう **DNS** と **HTTP** だけ残す。

<p align="center">
  <img src="./img/DHCP%26DNS/labC-fliter-edit.png" alt="DNS と HTTP に設定した Edit Filters" width="700"><br>
  <em>図19 — **Edit Filters** → **DNS** と **HTTP** だけにチェック。</em>
</p>

2. PC の Web Browser で `http://www.lab.local` と入力し **Go**、次に **Play / Capture-Forward** をクリックし順序を観察: **DNS** パケットが**先に** PC を出て、応答が戻り、**その後** **HTTP** パケットが出る。
3. 各色付きパケット → **PDU Information → OSI Model** をクリックして層を読む。**DNS** 応答は **UDP、送信元ポート53**、**HTTP** 応答は **TCP、送信元ポート80** — 同じサーバ IP（`192.168.0.98`）、2つの異なるサービス、そして **DNS が HTTP の開始前に答えた**:

<p align="center">
  <img src="./img/DHCP%26DNS/labC-pdu-dns-53.png" alt="UDP ポート53の DNS 応答を示す PDU" width="440">
  <img src="./img/DHCP%26DNS/labC-pdu-http-80.png" alt="TCP ポート80の HTTP 応答を示す PDU" width="440"><br>
  <em>図20・21 — 左: **DNS** 応答（Layer 4 **UDP src port 53**、Layer 7 DNS）。右: **HTTP** 応答（Layer 4 **TCP src port 80**、Layer 7 HTTP）。両方 <code>192.168.0.98</code> 発 — ルックアップが先、ページが後。</em>
</p>

> 💡 これが DNS の眼目です: 人は覚えやすい**名前**を使い、コンピュータは**数字**で動き続ける — そしてルックアップは常に実接続の*前*に起きます。

## ステップC6 — シミュレーションモードで DORA を見る

1. **Simulation**（右下）をクリックし **Edit Filters** で **DHCP**（と DNS）を残す。
2. PC で IP Configuration を **Static** に切り替えてから **DHCP** に戻し、新しい要求を誘発。
3. **Discover → Offer → Request → ACK** パケットをステップ実行 — [Wireshark ラボA](./WIRESHARK_GUIDE.ja.md#ステップa2--4つの-dora-メッセージを読む) で読んだまさにそのメッセージが、今や*あなたの*ルータで生成されます。

<p align="center">
  <img src="./img/DHCP%26DNS/labC-step6-dora.png" alt="シミュレーションモードをステップ実行する DHCP パケット" width="700"><br>
  <em>図22 — シミュレーションモード: Event List で PC1 → SW-Eng → Router2 を進む DHCP パケット。</em>
</p>

DHCP パケット → **PDU Information** をクリックし、本物の DHCP メッセージであることを確認: **UDP、サーバポート67 → クライアントポート68**、ルータ（`192.168.0.1`）からブロードキャストアドレスへ。

<p align="center">
  <img src="./img/DHCP%26DNS/labC-DORA-ack.png" alt="UDP ポート67と68を示す DHCP パケットの PDU" width="460"><br>
  <em>図23 — DHCP PDU: Layer 7 **DHCP**、Layer 4 **UDP src 67 → dst 68**、`255.255.255.255` へブロードキャスト — まさに Wireshark ラボA の DORA の仕組み。</em>
</p>

---

# ラボC 設問

**まず自分で考えてから「答えを表示」をクリック。**

**Q1.** なぜ3つの**ゲートウェイ**（`.1`・`.65`・`.97`）と**サーバ**（`.98`）を DHCP プールから**除外**する必要がある？

<details>
<summary>💡 答えを表示</summary>

それらは**静的**に割り当てられています。プールがそのアドレスを PC にリースすると **IP 競合** — 2台が1アドレスを主張（ゲートウェイやサーバを失う）— になります。`ip dhcp excluded-address` がそれらを予約し、DHCP が決して配らないようにします。
</details>

**Q2.** なぜこのネットワークは1つでなく**3つ**の DHCP プールが必要？

<details>
<summary>💡 答えを表示</summary>

各プールは**1サブネット**を提供 — その `network`/マスクと `default-router` はサブネット固有です。3部門は3つの異なるサブネット（`/26`・`/27`・`/28`）で3つの異なるゲートウェイなので、各々独自のプールが必要です。ルータは要求が着いたインターフェイスで正しいプールに対応づけます。
</details>

**Q3.** PC は今 `ftp ftp.lab.local` を実行できます。順に**どの2つのルックアップ**が起きる？

<details>
<summary>💡 答えを表示</summary>

まず **DNS** クエリが `ftp.lab.local` → `192.168.0.98` を解決（名前には接続できず、IP にしか接続できない）。次に **FTP** クライアントがその IP に TCP 接続を開きます。名前解決は常にデータ接続に先行します。
</details>

**Q4.** 各 PC は誰も打ち込まずにどうやって DNS サーバのアドレス（`192.168.0.98`）を学んだ？

<details>
<summary>💡 答えを表示</summary>

**DHCP ACK** から — 各プールの `dns-server 192.168.0.98` 行が **DHCP Option 6** として配られます。同じ ACK がアドレス（`network`）とゲートウェイ（`default-router`、Option 3）も運びました。これはラボAの Wireshark ACK で読んだ Option データそのものです。
</details>

---

### 演習問題

ビルドをエンドツーエンドで証明するため、やってみましょう。終えたら**各チェックを付け**、結果のスクリーンショットやメモ（例 `S3/img/`）を保存してください。

**ラボB — SSH と FTP:**
- [ ] **1. 堅牢化＋ログイン。** ルータに SSH を設定し、PC から `ssh -l admin 192.168.0.1`。*記録:* SSH で到達した `R-CORE#` プロンプト、そして `telnet 192.168.0.1` が**拒否**されることを確認。
- [ ] **2. サブネット越しにファイルを移動。** 別部門の PC から `ftp 192.168.0.98`、`student` でログインし、`put` してから `get`。*記録:* 往復成功（`2911` ルータを越えた）。
- [ ] **3. なぜ FTP は安全でないか。** *記録:* 盗聴者がなぜ FTP ログインは読めるが SSH セッションは読めないか、そして代わりに何を使うか（**SFTP/FTPS**）を1行で。

**ラボC — DHCP と DNS:**
- [ ] **4. PC を自動アドレス。** PC を **DHCP** に切り替え、`ipconfig /all` を実行。*記録:* リースされた IP・ゲートウェイ・**DHCP Server**・**DNS Server**、そしてどの**プール**から来たか。
- [ ] **5. サーバに名前で到達。** PC から `ping www.lab.local`、次に Web Browser で **`http://www.lab.local`** を開く。*記録:* **IP をどこにも打たず**にページが読み込まれること。
- [ ] **6. DNS→HTTP を見る。** **シミュレーションモード**（DNS + HTTP でフィルタ）で名前を閲覧。*記録:* **DNS / UDP-53** パケットが **HTTP / TCP-80** パケットの**前**に発火すること。
- [ ] **ストレッチ — わざと配線を壊す。** `SW-Mgt` のアップリンクを**誤った**ルータインターフェイス（`g0/2` でなく `g0/0`）に移し、Management PC を更新。*記録:* 今リースされる**誤った（Engineering）アドレス**、そしてスイッチ → インターフェイス → サブネット → プールの連鎖を使って、なぜ壊れたかを説明。

> [!TIP]
> 各 *記録* 行を成果物として扱いましょう — これらがそろえば、ルータを**保護**し（SSH）、ファイルとページを**提供**し（FTP/HTTP/DNS）、ネットワークを**自己構成**させ（DHCP）、配線が誤ったときに診断できる証明になります。

---

# 次のステップ

- 自分の **FTP** ログインを Wireshark でキャプチャしてユーザ名/パスワードを**平文**で読み、次に **SSH** セッションが読めないことを確認 — ラボBで設定したことの実証。
- **宿題（README より）:** Kurose & Ross の **DNS** Wireshark ラボ + **DNS ポイズニング/スプーフィング**の1段落。
- `.pkt` ファイルを保存 — セッション4がこの同じネットワークを VLAN とルーティングで拡張します。
