## Cisco Packet Tracer — はじめに＆ラボ2

> 🌐 [English](./PACKET_TRACER_GUIDE.md) | **日本語**

> [セッション1 — ネットワーク解析入門とOSIモデル](./README.ja.md) のコンパニオンガイド。
> [README](./README.ja.md#ハンズオンラボ) の **ラボB — Cisco Packet Tracer：はじめてのネットワーク構築** に取り組む**前**に読んでください。
> [Wireshark — はじめに＆ラボ1 ガイド](./WIRESHARK_GUIDE.ja.md) と対になります。

---

- [Cisco Packet Tracer — はじめに＆ラボ2](#cisco-packet-tracer--はじめにラボ2)
  - [Cisco Packet Tracer とは？](#cisco-packet-tracer-とは)
  - [Packet Tracer のインストール](#packet-tracer-のインストール)
  - [Packet Tracer ウィンドウの案内](#packet-tracer-ウィンドウの案内)
  - [使用するデバイスアイコン](#使用するデバイスアイコン)
  - [デバイスやケーブルを削除する](#デバイスやケーブルを削除する)
  - [構築するもの](#構築するもの)
  - [ラボ2 — ルーティングされた2サブネットのネットワークを構築](#ラボ2--ルーティングされた2サブネットのネットワークを構築)
    - [ステップ1 — デバイスを配置する](#ステップ1--デバイスを配置する)
    - [ステップ2 — デバイスをケーブルで接続](#ステップ2--デバイスをケーブルで接続)
    - [ステップ3 — PC0 を設定](#ステップ3--pc0-を設定)
    - [ステップ4 — PC1 を設定](#ステップ4--pc1-を設定)
    - [ステップ5 — ルータを設定](#ステップ5--ルータを設定)
    - [ステップ6 — アドレスを確認](#ステップ6--アドレスを確認)
    - [ステップ7 — `ping` で疎通テスト](#ステップ7--ping-で疎通テスト)
  - [ラボ2 設問](#ラボ2-設問)

---

### Cisco Packet Tracer とは？

**Cisco Packet Tracer** は Cisco Networking Academy が提供する無料のネットワーク**シミュレータ**です。ルータ・スイッチ・PC・サーバをキャンバスにドラッグ＆ドロップし、ケーブルでつなぎ、設定し、本物そっくりのトラフィックを流せます — すべてソフトウェア上で、物理ハードウェア不要です。

**Wireshark** が実ネットワークのトラフィックを*観察する*のに対し、**Packet Tracer** はネットワークを*構築して*パケットが流れる様子を見られます。**シミュレーションモード**では1つのパケットをホップごとにアニメーション表示し、各機器がOSIの各層でどうカプセル化/非カプセル化するかを示します — [セッション1のOSI講義](./README.ja.md#講義) の絶好の相棒です。

---

### Packet Tracer のインストール

Packet Tracer は **Cisco Networking Academy の「Resource Hub」** から無料で配布されています。ダウンロードには（無料の）Cisco アカウントが必要です。

1. **サインイン／アカウント作成。** Cisco SkillsForAll / Networking Academy のサイトでログイン。**Google アカウントがあればそのまま直接サインインできます** — これが最短です。

   <p align="center">
     <img src="./img/cisco_package_tracer/login_page.png" alt="Cisco ログインページ" width="420"><br>
     <em>図1 — サインイン。Google アカウント利用が最短。</em>
   </p>

2. **Resource Hub を開き、自分のOS向け（Windows / macOS / Linux）のインストーラをダウンロード。** プラットフォームに合うビルドを選びます。

   <p align="center">
     <img src="./img/cisco_package_tracer/cisco_pkg_install_page.png" alt="Cisco Resource Hub ダウンロードページ" width="420"><br>
     <em>図2 — Resource Hub から自分のOS用インストーラをダウンロード。</em>
   </p>

3. **インストールして起動。** 初回起動時に**同じアカウントで再度サインイン**（または *Skills for All* / *Networking Academy* を選択）。**"Keep me logged in"** にチェックすれば毎回聞かれません。

   <p align="center">
     <img src="./img/cisco_package_tracer/login_sill_forall.png" alt="起動時の Packet Tracer サインイン" width="600"><br>
     <em>図3 — 初回起動で再度サインイン（"Keep me logged in" にチェック）。</em>
   </p>

---

### Packet Tracer ウィンドウの案内

ログインすると、メインのワークスペースが表示されます:

<p align="center">
  <img src="./img/cisco_package_tracer/cisco_package_tracer_window.png" alt="Cisco Packet Tracer メインウィンドウ" width="700"><br>
  <em>図4 — Packet Tracer のワークスペースと主要エリア。</em>
</p>

| エリア | 場所 | 役割 |
| :--- | :--- | :--- |
| **メニュー＆上部ツールバー** | 上部 | File/Edit/Options、ツール: 選択・移動・**削除**・ズームなど。 |
| **Logical / Physical タブ** | 左上 | このラボでは **Logical**（抽象トポロジ表示）のまま。 |
| **ワークスペース** | 中央（白） | デバイスを配置・配線するキャンバス。 |
| **デバイス種別セレクタ** | 左下 | ハードのカテゴリ: Routers・Switches・End Devices・Connections… |
| **デバイス一覧** | 下部、セレクタの隣 | 選んだカテゴリの具体的な機種（例 `2901` ルータ）。 |
| **Realtime / Simulation 切替** | 右下 | **Realtime** は即時にトラフィックを流し、**Simulation** はホップごとに進められる。 |

> 📖 **公式リファレンス:** **Menu Bar**・**Main Tool Bar**・**Common Tools Bar**（Select・Inspect・Delete・Place Note・PDU ツール）・**Logical/Physical** のワークスペースナビ・**Realtime/Simulation Bar**・**Network Component Box**（デバイス＆接続セレクタ）といった*すべて*のパネルのインタラクティブな解説は、Cisco 公式の [**Packet Tracer Interface Overview**](https://tutorials.ptnetacad.net/help/default/interfaceOverview.htm) を参照。各エリアが Packet Tracer のラベル通りの名前で説明されています。

---

### 使用するデバイスアイコン

Packet Tracer は Cisco 標準のネットワークアイコンを使います。このラボに関係するのは **PC**・**Router**・**Cable (Various)** コネクタです。

<p align="center">
  <img src="./img/cisco_package_tracer/defualt_icon.png" alt="Cisco 標準デバイスアイコン" width="520"><br>
  <em>図5 — Cisco 標準デバイスアイコン。使うのは PC・Router・Cable。</em>
</p>

---

### デバイスやケーブルを削除する

間違えたら、上部ツールバーの **Delete** ツール（✕の付いた箱、ショートカット **Del**）をクリックし、削除したいデバイスやケーブルをクリックします。

<p align="center">
  <img src="./img/cisco_package_tracer/remove_device_method.png" alt="ツールバーの Delete ツール" width="560"><br>
  <em>図6 — Delete ツール（ショートカット <code>Del</code>）でデバイスやケーブルを削除。</em>
</p>

---

### 構築するもの

**2つの*異なる*サブネット上の2台のPC**を**ルータ**でつなぎます。2台のPCは別ネットワークにいるためルータが必須で、スイッチだけではトラフィックを移せません。

<p align="center">
  <img src="./img/cisco_package_tracer/netwrok_structure.png" alt="目標トポロジ: PC0 — Router0 (2901) — PC1" width="480"><br>
  <em>図7 — 2台のPCを Cisco <strong>2901</strong> ルータに接続。インターフェイス <strong>g0/0</strong> が PC0 のネットワーク、<strong>g0/1</strong> が PC1 のネットワークに面します。緑の三角はインターフェイスが up の意味。</em>
</p>

| デバイス | インターフェイス | IPアドレス | サブネットマスク | デフォルトゲートウェイ |
| :--- | :--- | :--- | :--- | :--- |
| **PC0** | FastEthernet0 | `192.168.1.100` | `255.255.255.0` | `192.168.1.1` |
| **PC1** | FastEthernet0 | `10.1.1.100` | `255.255.255.0` | `10.1.1.1` |
| **Router0** | g0/0 | `192.168.1.1` | `255.255.255.0` | — |
| **Router0** | g0/1 | `10.1.1.1` | `255.255.255.0` | — |

> 各PCの**デフォルトゲートウェイは、自分のサブネット上のルータインターフェイス**です。宛先が自分のローカルネットワークに*ない*ときにホストがパケットを送る先のアドレスです。

---

### ラボ2 — ルーティングされた2サブネットのネットワークを構築

#### ステップ1 — デバイスを配置する

* 左下のセレクタで **End Devices** を選び、**2台の `PC`** アイコンをワークスペースにドラッグ。`PC0`・`PC1` と名付けられます。
* **Routers** を選び、**`2901`** ルータを中央にドラッグ。`Router0` になります。

#### ステップ2 — デバイスをケーブルで接続

* **Connections**（稲妻アイコンのカテゴリ）を選び、**Copper Straight-Through**（無地の実線、*Cable (Various)*）を選択。
* **`PC0`** をクリック → **`FastEthernet0`** を選択。続いて **`Router0`** → **`GigabitEthernet0/0`** を選択。
* **`PC1`** → **`FastEthernet0`**、続いて **`Router0`** → **`GigabitEthernet0/1`**。
* リンクの点は最初**赤/橙**で、インターフェイスが up になると**緑**に変わります（ルータのインターフェイスはステップ5で起動）。

#### ステップ3 — PC0 を設定

**`PC0`** → **Config** タブ（または **Desktop → IP Configuration**）で設定:

* **IPv4 Address:** `192.168.1.100`
* **Subnet Mask:** `255.255.255.0`
* **Default Gateway:** `192.168.1.1`

<p align="center">
  <img src="./img/cisco_package_tracer/PC0_setting.png" alt="PC0 のIPアドレスとゲートウェイを設定" width="660"><br>
  <em>図8 — PC0: IP <code>192.168.1.100</code>、マスク <code>255.255.255.0</code>、ゲートウェイ <code>192.168.1.1</code>。</em>
</p>

#### ステップ4 — PC1 を設定

**`PC1`** → **Config** タブ（または **Desktop → IP Configuration**）で設定:

* **IPv4 Address:** `10.1.1.100`
* **Subnet Mask:** `255.255.255.0`
* **Default Gateway:** `10.1.1.1`

<p align="center">
  <img src="./img/cisco_package_tracer/PC1_setting.png" alt="PC1 のIPアドレスとゲートウェイを設定" width="660"><br>
  <em>図9 — PC1: IP <code>10.1.1.100</code>、マスク <code>255.255.255.0</code>、ゲートウェイ <code>10.1.1.1</code>。</em>
</p>

#### ステップ5 — ルータを設定

ルータのインターフェイスは**既定でシャットダウンされIPもありません**。**`Router0`** → **CLI** タブで以下を入力します。各インターフェイスにゲートウェイIPを割り当て、`no shutdown` で起動します:

```ios
Router> enable
Router# configure terminal
Router(config)# interface g0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# no shutdown
Router(config-if)# exit
Router(config)# interface g0/1
Router(config-if)# ip address 10.1.1.1 255.255.255.0
Router(config-if)# no shutdown
```

**CLI プロンプトの理解（Cisco IOS のコマンドモード）。** Cisco ルータはどこからでも何でも変更させてはくれません — 入れ子の*モード*を移動し、**プロンプト記号が今どのモードかを示します**。最初の2コマンドは「見るだけ」から「設定できる」へ昇る手順です:

| コマンド | 表示されるプロンプト | モード | 意味 |
| :--- | :--- | :--- | :--- |
| *（接続直後）* | `Router>` | **User EXEC** | 最下位の読み取り専用モード。基本ステータスは見られるが**何も変更できない**。`>` が目印。 |
| **`enable`** | `Router#` | **Privileged EXEC** | 強力なコマンドを「有効化」。`#`（イネーブルモード）で全設定の表示・保存・再起動が可能 — ただしまだ設定の*編集*はできない。*管理者*権限のイメージ。 |
| **`configure terminal`** | `Router(config)#` | **Global Configuration** | *端末から*設定編集モードに入る。機器全体に影響する設定を変更できる。`(config)` が編集中の印。 |
| `interface g0/0` | `Router(config-if)#` | **Interface Config** | 1つのインターフェイスに入る。`(config-if)` は、今打つコマンド（`ip address` や `no shutdown` など）が**そのインターフェイスにのみ**適用される意味。 |

つまり流れは: **`enable`**（管理者権限を得る）→ **`configure terminal`**（設定編集を開始）→ **`interface g0/0`**（1つのポートに入る）→ IPを設定して起動。**`exit`** で一段ずつ戻り（例 `(config-if)` から `(config)`）、**`end`** / **`Ctrl+Z`** で一気に privileged EXEC（`Router#`）へ戻ります。

> 💡 詰まったり何を打てるか分からないときは、どのプロンプトでも **`?`** を入力すれば IOS がそのモードで使えるコマンドを一覧表示します。

**`ip address 192.168.1.1 255.255.255.0` の理解。** この1コマンドがインターフェイスにネットワーク上の身元を与えます。**2つの部分**から成ります:

```
ip address   192.168.1.1        255.255.255.0
             └─ IPアドレス       └─ サブネットマスク
```

* **`192.168.1.1` — インターフェイスのIPアドレス。** これがルータの **g0/0** ポートのアドレスになり、まさに PC0 に入力した**デフォルトゲートウェイ**（ステップ3）です。PC0 が別ネットワークに届けたいとき、このアドレスにパケットを渡します。
* **`255.255.255.0` — サブネットマスク。** マスクはIPアドレスを**ネットワーク部**と**ホスト部**に分けます。`255` のバイトは「このバイトはネットワーク」、`0` のバイトは「このバイトは個々のホストを識別」を意味します。つまり `255.255.255.0` は*最初の3つの数字がネットワーク、最後がホスト*:

  | | 1番目 | 2番目 | 3番目 | 4番目 |
  | :--- | :---: | :---: | :---: | :---: |
  | **IP** `192.168.1.1` | 192 | 168 | 1 | **1** |
  | **マスク** `255.255.255.0` | 255 | 255 | 255 | **0** |
  | **意味** | ネットワーク | ネットワーク | ネットワーク | ホスト |

  これはネットワーク **`192.168.1.0/24`** を定義します — `192.168.1.` で始まる（`.1`〜`.254`）すべてのホストが*同じ*ローカルネットワークにいます。`/24` は「マスクの先頭24ビットが1」（8+8+8）の略記で、`255.255.255.0` と同じ意味です。

> 🔑 これが**このラボにルータが要る理由**です: PC0（`192.168.1.x`）と PC1（`10.1.1.x`）はこのマスクの下で**ネットワーク部が異なる**ため**別ネットワーク**にいて、ルータを*通して*しか互いに届きません。ルータの2つのインターフェイスがその2つの異なるネットワーク上にあることに注目 — g0/0 に `192.168.1.1`、g0/1 に `10.1.1.1`。

<p align="center">
  <img src="./img/cisco_package_tracer/router_setting.png" alt="ルータ CLI 設定" width="600"><br>
  <em>図10 — 各インターフェイスにIPを割り当て、<code>no shutdown</code> で起動。</em>
</p>

`%LINK-5-CHANGED ... changed state to up` メッセージに注目 — 各インターフェイスがオンラインになった合図です（リンクの点も緑に）。

#### ステップ6 — アドレスを確認

**`PC0`** → **Desktop** タブ → **Command Prompt** を開き、`ipconfig` を実行。IPv4アドレス・サブネットマスク・デフォルトゲートウェイが設定通りか確認します。

```text
PC> ipconfig
   IPv4 Address.........: 192.168.1.100
   Subnet Mask..........: 255.255.255.0
   Default Gateway......: 192.168.1.1
```

<p align="center">
  <img src="./img/cisco_package_tracer/PC0_Ipconfig.png" alt="PC0 の ipconfig 出力" width="660"><br>
  <em>図11 — <strong>Desktop → Command Prompt</strong> を開き <code>ipconfig</code> でアドレスを確認。</em>
</p>

#### ステップ7 — `ping` で疎通テスト

引き続き PC0 の **Command Prompt** で:

* **自分／自分のサブネットへ ping:** `ping 192.168.1.100`
* **ルータをまたいで PC1 のサブネットへ ping:** `ping 10.1.1.100`

どちらも `TTL=127` で **4回の成功応答**が返るはずです。`10.1.1.100` に届くことは、**ルータが2つのサブネット間でパケットを転送している**証拠です。

<p align="center">
  <img src="./img/cisco_package_tracer/IP0_ping_test.png" alt="PC0 から両サブネットへの ping テスト" width="640"><br>
  <em>図12 — どちらの ping も4応答。ルータが2サブネット間でトラフィックを転送。</em>
</p>

> 🔬 **シミュレーションモードを試す:** **Simulation**（右下）に切り替え、再度 `ping` を実行し、各ホップで移動する封筒をクリック。**In Layers / Out Layers** タブに、PC0 → Router0 → PC1 でパケットがどうカプセル化／非カプセル化されるかが正確に表示されます — 動くOSIモデルです。

---

### ラボ2 設問

**まず自分で考えてから「答えを表示」をクリック。**

**Q1.** PC0（`192.168.1.x`）と PC1（`10.1.1.x`）が**別サブネット**なのに、**なぜ `ping 10.1.1.100` は成功**したのか？ それを可能にしたデバイスは？

<details>
<summary>💡 答えを表示</summary>

**ルータ（Router0）** が2つのサブネットの間にいて、一方から他方へパケットを**転送（ルーティング）**するからです。PC0 は `10.1.1.100` が自分の `192.168.1.0/24` ネットワークに*ない*と判断し、**デフォルトゲートウェイ**（`192.168.1.1`、ルータの g0/0）へ送ります。ルータはそれを **g0/1** から `10.1.1.0/24` ネットワークの PC1 へ転送します。ただのスイッチではこれはできません — 単一サブネット*内*でしかフレームを動かせないからです。
</details>

**Q2.** PC0 の設定から**デフォルトゲートウェイを削除**すると `ping 10.1.1.100` はどうなるか？ なぜ？

<details>
<summary>💡 答えを表示</summary>

ping は**失敗**します（"Destination host unreachable" / request timed out）。デフォルトゲートウェイがないと、PC0 は*別*サブネット宛のパケットの送り先がありません — 自分の `192.168.1.0/24` 内のホストへの届け方しか知らないからです。ゲートウェイはサブネット外すべてへの「出口」です。
</details>

**Q3.** `ping` 出力の応答に **`TTL=127`** とあります。送信側は 128 から始めました — この減少は、パケットが何**ホップ**したかについて何を示すか？ *(README の [ping 詳解](./README.ja.md#詳解ping-コマンドとオプション) 参照。)*

<details>
<summary>💡 答えを表示</summary>

**1ホップ。** TTL（Time To Live）はパケットが通る**ルータごとに1**減ります。128 から始まり 127 で届いた = ちょうど**1台のルータ**（Router0）を通過。（TTL はパケットの永久ループを防ぎます: 0 になると破棄。）
</details>

**Q4.** ルータインターフェイスの **`no shutdown`** コマンドの役割は？ 実行前のインターフェイスはどの状態？

<details>
<summary>💡 答えを表示</summary>

既定でルータのインターフェイスは**管理上ダウン**（シャットダウン）です。**`no shutdown`** がインターフェイスを**オン**にします — だから `%LINK-5-CHANGED ... changed state to up` メッセージが出て、リンクの点が**緑**になります。IPアドレスの割り当てだけでは*不十分*で、インターフェイスを有効化する必要があります。
</details>

**Q5.** **シミュレーションモード**に切り替えてルータをまたいで ping します。**Router0** で **In Layers** と **Out Layers** タブに現れるOSI層は？ そして**第2層（MAC）ヘッダが変わる**のに**第3層（IP）アドレスは同じ**ままなのはなぜ？

<details>
<summary>💡 答えを表示</summary>

ルータでパケットは**入って**きて **第3層（IP）まで上って**処理され（宛先IPを読んでどこへ送るか決める）、**出て**いくとき第2層へ再構築されます。**第3層の送信元/宛先IPアドレスはエンドツーエンドで同じ**（PC0 → PC1）— *元の送信者と最終受信者*を識別するからです。一方**第2層のMACアドレスはホップごとに書き換えられます**: 出ていく際、送信元MACはルータの **g0/1** のMAC、宛先MACは **PC1** のMACになります — MACアドレスは*現在の*物理リンク上の機器しか識別しないからです。この「IPは不変、MACはホップごとに変わる」がルーティングの核心です。
</details>

---
