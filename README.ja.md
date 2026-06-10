# 🌎ネットワーク解析コース — 1ヶ月プラン

> 🌐 [English](./README.md) | **日本語**

> **期間**: 4週間 | **スケジュール**: 週1回の授業 | **授業時間**: 3時間（10分の休憩を含む）  
> **総セッション数**: 4 | **総対面時間**: 12時間

---

## コース概要

本コースは、ネットワーク解析の基礎を学生に紹介します。データがネットワークをどのように流れるかを理解するところから、実際のトラフィックをキャプチャし、検査し、トラブルシューティングするところまでを扱います。学生は業界標準ツール（Wireshark、Cisco Packet Tracer）を用いた実践経験を積み、ITサポート・ネットワーク管理・サイバーセキュリティのキャリアに応用できる実用スキルを身につけます。

## 前提知識

- 基本的なコンピュータリテラシー（ファイル管理、ソフトウェアのインストール）
- インターネットの概念的な理解
- コマンドラインの知識があると望ましいが**必須ではありません**

## 必要なソフトウェア（無料）

| ツール | 目的 | ダウンロードリンクとインストールガイド |
|------|---------|----------|
| **Wireshark** | ライブパケットキャプチャとプロトコル解析 | [wireshark](https://www.wireshark.org/docs/wsug_html_chunked/ChBuildInstallWinInstall.html) |
| **Cisco Packet Tracer** | ネットワークシミュレーションとトポロジ設計 | [netacad](https://www.netacad.com/skillsforall/files/Cisco_Packet_Tracer_Download_and_Installation_Instructions.pdf) |

## トピック一覧
- [セッション1 — ネットワーク解析入門とOSIモデル](/S1/README.ja.md)
- [セッション2 — プロトコルスタック詳解：Ethernet・IP・TCP・UDP](/S2/README.ja.md)
- [セッション3 — ネットワークサービス：DHCP・DNS・ARP](/S3/README.ja.md)
- [セッション4 — ルーティング・スイッチング・VLAN](/S4/README.md)

## 推奨される補助ツール

| ツール | 目的 | 導入セッション |
|------|---------|-----------------|
| **tcpdump / tshark** | CLIベースのパケットキャプチャ | セッション4 |
| **Nmap** | ネットワークスキャンと探索 | セッション3 |
| **NetworkMiner** | ネットワークフォレンジックとファイル抽出 | セッション3 |
| **Scapy (Python)** | カスタムパケット生成 | 任意 / 上級 |

---

## セッションごとの時間配分（3時間 / 180分）

| セグメント | 時間 | 説明 |
|---------|----------|-------------|
| 🔄 ウォームアップと復習 | 10分 | 前回の復習、Q&A、宿題チェック |
| 📖 講義とライブデモ | 50分 | 実世界・リアルタイムのデモを交えた中核概念 |
| ☕ 休憩 | 10分 | ラボの前に立ち上がってストレッチ・水分補給 |
| 🛠️ ハンズオンラボ | 80分 | Wireshark と Cisco Packet Tracer を用いた詳細なガイド付き演習 |
| 📝 まとめと予告 | 10分 | 要点、宿題の概要、次セッションの予告 |


---

<!-- ## 評価の概要

| 評価項目 | 配点 | セッション |
|------------|--------|---------|
| 授業参加とラボ | 30% | 毎回 |
| 宿題 | 20% | 毎週 |
| ミニCTFの成績 | 15% | セッション3 |
| 総合プロジェクト | 35% | セッション4 |

--- -->

## 推奨リソース
> [!IMPORTANT] 重要
> Wireshark の詳細情報はこちらの [Wireshark User's Guide](https://www.wireshark.org/docs/wsug_html_chunked/) で確認できます
### 練習用 PCAP データセット
| 提供元 | URL | レベル |
|--------|-----|-------|
| Wireshark Sample Captures | `wiki.wireshark.org/SampleCaptures` | 初級 |
| Netresec PCAP Repository | `netresec.com` | 中級 |
| Malware-Traffic-Analysis.net | `malware-traffic-analysis.net` | 上級 |
| Digital Corpora | `digitalcorpora.org` | 中級 |

### ラボリソース
| 提供元 | URL | 説明 |
|--------|-----|-------------|
| Kurose & Ross Wireshark Labs | UMass 教科書サイト | 定番のプロトコルラボ |
| Jeremy's IT Lab | YouTube + GitHub | 76本以上の Packet Tracer ラボ |
| Cisco NetAcad | `netacad.com` | 公式の体系的コース |
| TryHackMe | `tryhackme.com` | インタラクティブなセキュリティ演習 |
| Blue Team Labs Online | `blueteamlabs.online` | 防御側のチャレンジ |

---


> [!NOTE]
> **ラボ環境**: 学生は Wireshark と Packet Tracer をインストールできる管理者権限のある自分のノートPCを用意してください。
