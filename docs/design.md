# 企業セキュリティ リスクアセスメントシステム 設計書

**バージョン：v2.6**　　**最終更新：2026-03-21**

---

## 目次

1. [システム概要](#1-システム概要)
2. [アーキテクチャ](#2-アーキテクチャ)
3. [ファイル構成](#3-ファイル構成)
4. [対応フレームワーク](#4-対応フレームワーク)
5. [機能仕様](#5-機能仕様)
6. [データ設計](#6-データ設計)
7. [識別番号定義ファイル設計](#7-識別番号定義ファイル設計)
8. [変更履歴](#8-変更履歴)

---

## 1. システム概要

本システムは、複数の国際セキュリティフレームワークに対応した企業向けセキュリティリスクアセスメントツールです。

| 項目 | 内容 |
|---|---|
| システム名 | 企業セキュリティ リスクアセスメントシステム |
| バージョン | 2.6 |
| 動作環境 | Webサーバー経由（fetch API 使用のため file:// 不可） |
| AI機能 | なし（オフライン完結） |
| PDF出力 | ブラウザ印刷方式（4ページ構成・日本語対応） |
| データ保存 | JSON 直接ダウンロード方式 |
| 拡張方式 | manifest.json + 質問JSON + refs_*.json の追加のみ |

---

## 2. アーキテクチャ

```
index.html                    ← アプリ本体（エンジン）
questions/
├── manifest.json             ← フレームワーク一覧（refsFiles フィールド付き）
├── nist_onprem.json          ← NIST SP800-53 オンプレミス（90問）
├── nist_cloud.json           ← NIST SP800-53 クラウド（80問）
├── nist_hybrid.json          ← NIST SP800-53 ハイブリッド（79問）
├── cis_v8.json               ← CIS Controls v8.1（98問）
├── iso27001.json             ← ISO/IEC 27001:2022（93問）
├── nist_csf.json             ← NIST CSF v2.0（63問）
└── pci_dss.json              ← PCI DSS v4.0（67問）
content/
├── guide.json                ← 解説書コンテンツ
├── version.json              ← バージョン情報
├── refs_nist.json            ← NIST SP 800-53 定義（99件）
├── refs_cis.json             ← CIS Controls v8.1 定義（113件）
├── refs_csa.json             ← CSA CCM v4 定義（16件）
├── refs_iso27001.json        ← ISO/IEC 27001:2022 定義（93件）
├── refs_csf.json             ← NIST CSF v2.0 定義（63件）
└── refs_pci.json             ← PCI DSS v4.0 定義（104件）
docs/
├── design.md                 ← 本設計書
├── test_spec.md              ← テスト仕様書
└── prompt_fw_addition.md     ← フレームワーク追加用プロンプト
```

**動作要件：** Webサーバー経由必須（fetch API 使用のため `file://` 不可）

---

## 3. ファイル構成

### 3.1 index.html

アプリ本体。質問データ・コンテンツを持たない純粋なエンジン。  
全機能を1ファイルに収録（HTML / CSS / JavaScript）。

**読み込むCDNライブラリ：**
- Chart.js 4.4.1（レーダーチャート）
- marked.js 9.1.6（Markdown→HTML変換）

### 3.2 manifest.json

フレームワークカタログ。スタート画面のカード一覧を動的生成する。

```json
{
  "version": "1.0",
  "frameworks": [
    {
      "id": "pci_dss",
      "label": "PCI DSS v4.0",
      "sublabel": "全12要件",
      "icon": "💳",
      "color": "#0052a5",
      "description": "説明文",
      "file": "questions/pci_dss.json",
      "env": null,
      "tags": ["PCI DSS v4.0"],
      "refsFiles": ["content/refs_pci.json"]
    }
  ]
}
```

**refsFiles フィールド：** フレームワーク開始時に並列 fetch する識別番号定義ファイルの一覧。

### 3.3 質問 JSON ファイル

```json
{
  "id": "フレームワークID",
  "label": "表示名",
  "sublabel": "副題",
  "frameworks": ["フレームワーク名"],
  "sections": [
    {
      "id": "セクションID",
      "label": "セクション表示名",
      "ref": "英語参照名",
      "subsections": [
        {
          "title": "サブセクション名",
          "questions": [
            {
              "no": "質問番号",
              "text": "質問文（体言止め）",
              "refs": ["識別番号1", "識別番号2"]
            }
          ]
        }
      ]
    }
  ]
}
```

### 3.4 識別番号定義ファイル（refs_*.json）

```json
{
  "_meta": {
    "version": "1.0",
    "framework": "フレームワーク名",
    "description": "説明",
    "count": 件数
  },
  "識別番号": {
    "summary": "識別番号 — 30〜50字の概要",
    "detail": "【正式名称】...
【目的・概要】...
【主な要件】
・...
【実装例】..."
  }
}
```

---

## 4. 対応フレームワーク

| ID | フレームワーク | 問数 | 識別番号 |
|---|---|---|---|
| nist_onprem | NIST SP 800-53 / CIS Controls（オンプレ） | 90問 | refs_nist.json + refs_csa.json + refs_cis.json |
| nist_cloud | NIST SP 800-53 / CIS Controls（クラウド） | 80問 | refs_nist.json + refs_csa.json + refs_cis.json |
| nist_hybrid | NIST SP 800-53 / CIS Controls（ハイブリッド） | 79問 | refs_nist.json + refs_csa.json + refs_cis.json |
| cis_v8 | CIS Controls v8.1（全18カテゴリ） | 98問 | refs_cis.json |
| iso27001 | ISO/IEC 27001:2022（全10ドメイン） | 93問 | refs_iso27001.json |
| nist_csf | NIST CSF v2.0（全6機能） | 63問 | refs_csf.json |
| pci_dss | PCI DSS v4.0（全12要件） | 67問 | refs_pci.json |

**識別番号辞書：合計 488件**

| ファイル | フレームワーク | 件数 |
|---|---|---|
| refs_nist.json | NIST SP 800-53 Rev.5 | 99件 |
| refs_cis.json | CIS Controls v8.1 | 113件 |
| refs_csa.json | CSA CCM v4 | 16件 |
| refs_iso27001.json | ISO/IEC 27001:2022 | 93件 |
| refs_csf.json | NIST CSF v2.0 | 63件 |
| refs_pci.json | PCI DSS v4.0 | 104件 |

---

## 5. 機能仕様

### 5.1 画面構成

| Step | 画面名 | 概要 |
|---|---|---|
| Step 0 | スタート画面 | 組織情報入力・フレームワーク選択・ファイル読み込み |
| Step 1 | 回答入力画面 | セクションタブ・4択回答・メモ・一括操作・アセスメント情報バー |
| Step 2 | 評価結果画面 | スコア・レーダーチャート・要対応/改善・総合評価コメント |

### 5.2 回答方式

| 回答値 | 表示 | スコア |
|---|---|---|
| yes | はい | 100% |
| most | ほとんど | 75% |
| part | 一部 | 50% |
| no | いいえ | 0% |

### 5.3 成熟度レベル

| レベル | ラベル | スコア範囲 |
|---|---|---|
| Lv.5 | 最適化 | 85%以上 |
| Lv.4 | 管理 | 70〜84% |
| Lv.3 | 定義 | 55〜69% |
| Lv.2 | 管理的 | 35〜54% |
| Lv.1 | 初期 | 34%以下 |

### 5.4 識別番号ポップオーバー機能

- **ホバー：** `REF_DEFINITIONS[ref].summary` をツールチップで表示
- **クリック：** `REF_DEFINITIONS[ref].detail` を4セクション構成で整形表示
- **ロード：** フレームワーク開始時に `refsFiles` を Promise.all で並列 fetch
- **キャッシュ：** `_defCache` にセッション内キャッシュ（フレームワーク切替時にリセット）
- **フォールバック：** `REF_DEFINITIONS` が空の場合は `fetchRefDefinition` 内で再ロードを試みる

### 5.5 アセスメント情報バー

- Step 1（回答入力）・Step 2（評価結果）の両方に表示
- 企業名・組織名/部署名・入力管理者・対象環境・評価実施日を一覧表示
- 「✏️ 編集」ボタンで編集モーダルを開き、変更確認ダイアログを経て更新
- キャンセル時は `_orgEditPrev` により編集前の値に復元

### 5.6 総合評価コメント（Markdown エディタ）

- 評価結果画面の Findings セクション下に配置
- ツールバー：太字・斜体・見出し・リスト・引用・リンク・コード・区切り線
- 「編集」「プレビュー」タブ切替（marked.js によるレンダリング）
- `{ breaks: true }` オプションにより Enter 1回で改行
- `_mdSummary` にグローバル保持・保存ファイルの `summaryComment` フィールドに含まれる

### 5.7 データの保存と読み込み

**保存：** Blob + `<a download>` による直接ダウンロード  
**ファイル名：** `security_assessment_(企業名)_(フレームワーク名)_(日付).json`  
**保存データ構造：**

```json
{
  "version": "2.6",
  "savedAt": "ISO 8601 形式",
  "orgInfo": { "company": "", "dept": "", "admin": "", "date": "" },
  "env": "フレームワークID or null",
  "frameworkId": "フレームワークID",
  "answeredCount": 件数,
  "totalCount": 件数,
  "answers": { "質問番号": "yes|most|part|no" },
  "memos": { "質問番号": "メモテキスト" },
  "summaryComment": "Markdown テキスト",
  "sections": [ ... ]
}
```

**読み込み：** `frameworkId` を主キーとしてフレームワークを特定。全フレームワーク対応。読み込み完了後に Step 1 へ自動遷移。

### 5.8 PDF レポート出力

- `window.print()` によるブラウザ印刷方式
- 新規ウィンドウで印刷 HTML を生成・印刷ダイアログを自動表示
- Noto Sans JP（Google Fonts CDN）で日本語完全対応

**ページ構成：**

| ページ | 内容 |
|---|---|
| 1（表紙） | 組織情報・総合スコア・成熟度・回答率・レーダーチャート |
| 2（総合評価コメント） | Markdown→HTML変換済みコメント（入力時のみ出力） |
| 3（セクション別スコア） | 全セクションのスコア・成熟度・スコアバー |
| 4（要対応事項） | 回答「いいえ」の全項目 |
| 5（改善推奨事項） | 回答「一部対応」「ほとんど対応」の全項目 |

### 5.9 やり直し確認ダイアログ

回答が1件以上ある場合に確認ダイアログを表示。「やり直す」でリセット実行。  
リセット時に `answers`・`memos`・`orgInfo`・`REF_DEFINITIONS`・`_defCache`・`_mdSummary` を全クリア。

---

## 6. データ設計

### 6.1 グローバル状態変数

| 変数名 | 型 | 用途 |
|---|---|---|
| `currentFrameworkId` | string | 選択中フレームワークID |
| `currentFrameworkMeta` | object | manifest.json のフレームワークエントリ |
| `currentEnv` | string\|null | 環境種別（レガシー互換） |
| `sections` | array | 現在の質問データ |
| `answers` | object | 回答データ `{no: "yes\|most\|part\|no"}` |
| `memos` | object | メモデータ `{no: "テキスト"}` |
| `orgInfo` | object | 組織情報 |
| `REF_DEFINITIONS` | object | 識別番号定義（フレームワーク切替時にリセット） |
| `_defCache` | object | 整形済み定義のセッションキャッシュ |
| `_mdSummary` | string | 総合評価コメント（Markdown） |
| `_orgEditPrev` | object | 編集前の orgInfo（キャンセル用） |
| `radarChart` | Chart | Chart.js インスタンス |
| `frameworkCatalog` | array | manifest.json の frameworks 配列 |

---

## 7. 識別番号定義ファイル設計

### 7.1 ファイル命名規則

`content/refs_{フレームワーク略称}.json`

### 7.2 detail フィールドのフォーマット

```
【正式名称】英語の正式名称
【目的・概要】
2〜4文で目的と重要性を説明
【主な要件】
・要件1
・要件2
・要件3
【実装例】
具体的なツール名・手順を含む実装例
```

### 7.3 識別番号のキー形式

| フレームワーク | 形式 | 例 |
|---|---|---|
| NIST SP 800-53 | `NIST XX-N` | `NIST AC-2` |
| CIS Controls | `CIS N.N` | `CIS 5.1` |
| CSA CCM | `CSA CCM XXX-NN` | `CSA CCM IAM-06` |
| ISO/IEC 27001 | `ISO 27001 AN.N` | `ISO 27001 A5.1` |
| NIST CSF | `CSF XX.XX-NN` | `CSF GV.OC-01` |
| PCI DSS | `PCI N.N.N` | `PCI 1.1.1` |

---

## 8. 変更履歴

| バージョン | 日付 | 変更内容 |
|---|---|---|
| v1.0 | 2026-03-21 | 初版リリース。NIST SP800-53 オンプレミス対応。 |
| v2.0 | 2026-03-21 | マルチフレームワーク対応。識別番号辞書内蔵。ISO/IEC 27001:2022 追加。 |
| v2.1 | 2026-03-21 | 識別番号定義ファイルをフレームワーク別外部JSONに分割（refs_*.json）。全321件の詳細定義を整備。 |
| v2.2 | 2026-03-21 | PDF出力機能追加（ブラウザ印刷方式）。やり直し確認ダイアログ追加。保存ボタン初期無効化。回答画面にアセスメント情報バー追加。 |
| v2.3 | 2026-03-21 | NIST CSF v2.0 対応追加（6機能・22カテゴリ・63問・refs_csf.json 63件）。識別番号辞書計384件に拡張。 |
| v2.4 | 2026-03-21 | 保存方式をダウンロード方式に変更（saveModal廃止）。NIST CSF 保存・読み込みバグ修正。ファイル名にフレームワーク名を追加。アセスメント情報バー編集機能追加。評価結果ページのハードコード表記削除。 |
| v2.5 | 2026-03-21 | 総合評価コメント欄追加（Markdownエディタ・プレビュー切替・PDFレポートへの反映・保存復元対応）。 |
| v2.6 | 2026-03-21 | PCI DSS v4.0 対応追加（6カテゴリ・12要件・67問・refs_pci.json 104件）。識別番号辞書計488件に拡張。ドキュメントを Markdown 形式に移行。 |
