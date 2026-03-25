# 多言語対応（i18n）設計書

**対象バージョン：v2.8（予定）**　　**作成日：2026-03-21**

---

## 1. 概要・設計方針

### 1.1 対応言語

| コード | 言語 | デフォルト |
|---|---|---|
| `ja` | 日本語 | ○ |
| `en` | English | — |

### 1.2 基本方針

- **アプリ本体（index.html）は1ファイルのまま維持する**。多言語化のためにサーバーサイドは追加しない。
- UIラベルはすべて `i18n` 辞書オブジェクト（JavaScript）で管理し、`lang` 切替時に `applyI18n()` を呼び出してDOMを一括更新する。
- 質問データ（`questions/*.json`）と識別番号定義（`content/refs_*.json`）はファイル内に `ja`/`en` の両フィールドを持つ多言語形式に移行する。
- **利用者入力値（メモ・総合評価コメント・組織情報）は言語切替の影響を受けない。**
- 言語設定は `localStorage` に保存し、次回起動時に引き継ぐ。

---

## 2. UIラベルの多言語化

### 2.1 i18n 辞書の構造

`index.html` 内の `<script>` ブロックに以下の辞書を定義する。

```javascript
const I18N = {
  ja: {
    // ── メニューバー ──────────────────────────────────────
    menu_guide:        '📖 解説書',
    menu_save:         '💾 保存',
    menu_load:         '📂 読み込み',
    menu_version:      'ℹ️ バージョン情報',
    menu_restart:      '🔄 最初からやり直す',

    // ── Step 0：スタート画面 ──────────────────────────────
    hero_tag:          'Security Assessment',
    hero_title:        '企業セキュリティ\nリスクアセスメント',
    hero_subtitle:     '評価するフレームワーク・カテゴリを選択してください',
    org_section_title: 'アセスメント情報',
    org_company:       '企業名',
    org_dept:          '組織名 / 部署名',
    org_admin:         '入力管理者',
    org_date:          '評価実施日',
    ph_company:        '例：株式会社〇〇',
    ph_dept:           '例：情報システム部',
    ph_admin:          '例：山田 太郎',
    loading_fw:        '質問セットを読み込み中...',
    btn_start:         'アセスメントを開始する →',
    upload_label:      '以前の回答ファイルを読み込む',
    upload_hint:       'クリックまたはドラッグ＆ドロップ（.json）',

    // ── Step 1：回答入力画面 ──────────────────────────────
    bulk_select_all:   '全て選択',
    bulk_select_none:  '選択解除',
    bulk_apply:        '一括適用',
    ans_yes:           'はい',
    ans_most:          'ほとんど',
    ans_part:          '一部',
    ans_no:            'いいえ',
    ans_clear:         'クリア',
    memo_placeholder:  'メモ（任意）',
    btn_prev:          '← 前のセクション',
    btn_next:          '次のセクション →',
    btn_show_result:   '📊 評価結果を見る',
    btn_finish:        '結果を見る →',

    // ── Step 2：評価結果画面 ──────────────────────────────
    result_title:      'セキュリティ評価結果',
    result_updating:   '結果を更新中...',
    btn_export_pdf:    '📄 PDFレポート出力',
    card_total_score:  '総合スコア',
    card_maturity:     '成熟度レベル',
    card_answered:     '回答済み',
    card_all_items:    '全 {n} 項目の平均',
    radar_title:       'レーダーチャート',
    section_scores:    'セクション別スコア',
    critical_title:    '要対応事項',
    warn_title:        '改善推奨事項',
    no_critical:       '未対応項目はありません ✓',
    no_warn:           '部分対応項目はありません ✓',
    summary_title:     '📝 総合評価コメント',
    summary_hint:      '評価結果の所見・推奨事項・経営層向けコメントなどを自由に記述できます（Markdown 形式）。',
    btn_save_result:   '💾 回答をファイル保存',
    btn_back_q:        '← 回答ページに戻る',
    btn_restart:       '🔄 最初からやり直す',

    // ── 情報バー ──────────────────────────────────────────
    info_company:      '企業名',
    info_dept:         '組織名 / 部署名',
    info_admin:        '入力管理者',
    info_env:          '対象環境',
    info_date:         '評価実施日',
    btn_edit_info:     '✏️ 編集',
    info_empty:        '未入力',
    info_auto_date:    '（自動）',

    // ── 各種モーダル・ダイアログ ──────────────────────────
    modal_close:       '✕ 閉じる',
    modal_cancel:      'キャンセル',
    modal_confirm:     '変更する',
    modal_back:        '戻る',
    modal_restart:     'やり直す',
    edit_modal_title:  '✏️ アセスメント情報の編集',
    confirm_modal_title: '✏️ アセスメント情報の変更確認',
    confirm_modal_msg: 'アセスメント情報を以下の内容に変更します。',
    confirm_ok:        'よろしいですか？',
    restart_modal_title: '🔄 最初からやり直す',
    restart_modal_msg: 'これまでの回答内容・メモ・組織情報がすべてクリアされます。よろしいですか？',
    load_modal_title:  '📂 回答ファイルの読み込み',
    load_drop_hint:    'ファイルを選択するか、ここにドラッグ＆ドロップしてください。',
    load_restore_note: '回答内容・アセスメント情報・メモがすべて復元されます。',
    load_overwrite_warn: '※ 現在の回答内容は読み込んだファイルの内容で上書きされます',
    load_btn_select:   'クリックしてファイルを選択',
    load_btn_drop:     'または .json ファイルをドロップ',

    // ── 成熟度ラベル ──────────────────────────────────────
    maturity_5:        '最適化',
    maturity_4:        '管理',
    maturity_3:        '定義',
    maturity_2:        '管理的',
    maturity_1:        '初期',

    // ── 識別番号ポップオーバー ────────────────────────────
    pop_copy:          'コピー',
    pop_close:         '✕ 閉じる',
    pop_loading:       '定義を取得中...',
    pop_not_found:     'の定義情報が見つかりませんでした。',

    // ── トースト・エラーメッセージ ────────────────────────
    toast_saved:       '💾 保存しました：',
    toast_loaded:      '📂 読み込み完了',
    toast_updated:     '✏️ アセスメント情報を更新しました',
    toast_copied:      '📋 コピーしました',
    err_fw_required:   'フレームワークを選択してください',
    err_invalid_file:  '対応していないファイル形式です',
    err_no_fw:         'フレームワーク情報が見つかりません',

    // ── PDFレポート ───────────────────────────────────────
    pdf_title:         '企業セキュリティ リスクアセスメント報告書',
    pdf_org_info:      'アセスメント情報',
    pdf_summary_card:  '評価サマリー',
    pdf_radar:         'レーダーチャート',
    pdf_section_scores:'セクション別スコア',
    pdf_critical:      '要対応事項',
    pdf_critical_sub:  '回答が「いいえ」の項目です。優先的な対応が必要です。',
    pdf_warn:          '改善推奨事項',
    pdf_warn_sub:      '「一部対応」または「ほとんど対応」の項目です。さらなる強化を推奨します。',
    pdf_summary_comment: '総合評価コメント',
    pdf_no_critical:   '✓ 要対応事項はありません',
    pdf_no_warn:       '✓ 改善推奨事項はありません',
    pdf_generated_at:  '生成日時：',
    pdf_print_btn:     '📄 PDFとして保存',
    pdf_print_hint:    '「送信先」で「PDFに保存」を選択してください。',
    pdf_col_section:   'セクション',
    pdf_col_score:     'スコア',
    pdf_col_maturity:  '成熟度',
    pdf_col_bar:       'スコアバー（各5%）',
    pdf_col_no:        'No.',
    pdf_col_question:  '質問内容',
    pdf_col_answer:    '回答',
  },

  en: {
    // ── Menu bar ──────────────────────────────────────────
    menu_guide:        '📖 Guide',
    menu_save:         '💾 Save',
    menu_load:         '📂 Load',
    menu_version:      'ℹ️ Version',
    menu_restart:      '🔄 Start Over',

    // ── Step 0: Start screen ─────────────────────────────
    hero_tag:          'Security Assessment',
    hero_title:        'Enterprise Security\nRisk Assessment',
    hero_subtitle:     'Select a framework to begin your assessment',
    org_section_title: 'Assessment Info',
    org_company:       'Organization',
    org_dept:          'Department / Division',
    org_admin:         'Assessor',
    org_date:          'Assessment Date',
    ph_company:        'e.g. Acme Corp.',
    ph_dept:           'e.g. Information Security Dept.',
    ph_admin:          'e.g. John Smith',
    loading_fw:        'Loading question sets...',
    btn_start:         'Start Assessment →',
    upload_label:      'Load a previously saved file',
    upload_hint:       'Click or drag & drop (.json)',

    // ── Step 1: Question input ────────────────────────────
    bulk_select_all:   'Select All',
    bulk_select_none:  'Deselect All',
    bulk_apply:        'Apply',
    ans_yes:           'Yes',
    ans_most:          'Mostly',
    ans_part:          'Partial',
    ans_no:            'No',
    ans_clear:         'Clear',
    memo_placeholder:  'Notes (optional)',
    btn_prev:          '← Previous',
    btn_next:          'Next →',
    btn_show_result:   '📊 View Results',
    btn_finish:        'View Results →',

    // ── Step 2: Results ───────────────────────────────────
    result_title:      'Security Assessment Results',
    result_updating:   'Updating results...',
    btn_export_pdf:    '📄 Export PDF Report',
    card_total_score:  'Overall Score',
    card_maturity:     'Maturity Level',
    card_answered:     'Answered',
    card_all_items:    'Avg. of {n} items',
    radar_title:       'Radar Chart',
    section_scores:    'Section Scores',
    critical_title:    'Action Required',
    warn_title:        'Improvement Recommended',
    no_critical:       'No action-required items ✓',
    no_warn:           'No partial-compliance items ✓',
    summary_title:     '📝 Overall Assessment Comment',
    summary_hint:      'Write your findings, recommendations, or executive summary here (Markdown supported).',
    btn_save_result:   '💾 Save Answers',
    btn_back_q:        '← Back to Questions',
    btn_restart:       '🔄 Start Over',

    // ── Info bar ──────────────────────────────────────────
    info_company:      'Organization',
    info_dept:         'Department',
    info_admin:        'Assessor',
    info_env:          'Framework',
    info_date:         'Assessment Date',
    btn_edit_info:     '✏️ Edit',
    info_empty:        '(not set)',
    info_auto_date:    '(auto)',

    // ── Modals / Dialogs ──────────────────────────────────
    modal_close:       '✕ Close',
    modal_cancel:      'Cancel',
    modal_confirm:     'Confirm',
    modal_back:        'Back',
    modal_restart:     'Start Over',
    edit_modal_title:  '✏️ Edit Assessment Info',
    confirm_modal_title: '✏️ Confirm Changes',
    confirm_modal_msg: 'The following changes will be applied.',
    confirm_ok:        'Are you sure?',
    restart_modal_title: '🔄 Start Over',
    restart_modal_msg: 'All answers, notes, and assessment info will be cleared. Continue?',
    load_modal_title:  '📂 Load Answer File',
    load_drop_hint:    'Select a file or drag & drop it here.',
    load_restore_note: 'Answers, assessment info, and notes will all be restored.',
    load_overwrite_warn: '※ Current answers will be overwritten by the loaded file.',
    load_btn_select:   'Click to select a file',
    load_btn_drop:     'or drop a .json file here',

    // ── Maturity labels ───────────────────────────────────
    maturity_5:        'Optimized',
    maturity_4:        'Managed',
    maturity_3:        'Defined',
    maturity_2:        'Repeatable',
    maturity_1:        'Initial',

    // ── Ref popover ───────────────────────────────────────
    pop_copy:          'Copy',
    pop_close:         '✕ Close',
    pop_loading:       'Loading definition...',
    pop_not_found:     'definition not found.',

    // ── Toast / Error messages ────────────────────────────
    toast_saved:       '💾 Saved: ',
    toast_loaded:      '📂 File loaded',
    toast_updated:     '✏️ Assessment info updated',
    toast_copied:      '📋 Copied',
    err_fw_required:   'Please select a framework',
    err_invalid_file:  'Unsupported file format',
    err_no_fw:         'Framework information not found',

    // ── PDF Report ────────────────────────────────────────
    pdf_title:         'Enterprise Security Risk Assessment Report',
    pdf_org_info:      'Assessment Information',
    pdf_summary_card:  'Assessment Summary',
    pdf_radar:         'Radar Chart',
    pdf_section_scores:'Section Scores',
    pdf_critical:      'Action Required',
    pdf_critical_sub:  'Items answered "No" — immediate action needed.',
    pdf_warn:          'Improvement Recommended',
    pdf_warn_sub:      'Items answered "Partial" or "Mostly" — further improvement recommended.',
    pdf_summary_comment: 'Overall Assessment Comment',
    pdf_no_critical:   '✓ No action-required items',
    pdf_no_warn:       '✓ No improvement-recommended items',
    pdf_generated_at:  'Generated: ',
    pdf_print_btn:     '📄 Save as PDF',
    pdf_print_hint:    'Select "Save as PDF" from the print destination.',
    pdf_col_section:   'Section',
    pdf_col_score:     'Score',
    pdf_col_maturity:  'Maturity',
    pdf_col_bar:       'Score Bar (per 5%)',
    pdf_col_no:        'No.',
    pdf_col_question:  'Question',
    pdf_col_answer:    'Answer',
  }
};

let currentLang = localStorage.getItem('lang') || 'ja';

function t(key, vars = {}) {
  let text = (I18N[currentLang] || I18N.ja)[key] || I18N.ja[key] || key;
  Object.entries(vars).forEach(([k, v]) => {
    text = text.replace(`{${k}}`, v);
  });
  return text;
}
```

### 2.2 DOM へのキー付け方式

HTMLの各ラベル要素に `data-i18n` 属性を付与する。

```html
<!-- テキストコンテンツの置き換え -->
<button data-i18n="menu_save">💾 保存</button>

<!-- placeholder の置き換え -->
<input data-i18n-placeholder="ph_company" placeholder="例：株式会社〇〇">

<!-- title（ツールチップ）の置き換え -->
<button data-i18n-title="btn_show_result" title="途中でも結果ページへ移動できます">

<!-- HTML を含む場合（innerHTML） -->
<p data-i18n-html="restart_modal_msg">これまでの...</p>
```

### 2.3 `applyI18n()` 関数

言語切替時に全DOMを一括更新する。

```javascript
function applyI18n() {
  // テキストコンテンツ
  document.querySelectorAll('[data-i18n]').forEach(el => {
    el.textContent = t(el.dataset.i18n);
  });
  // placeholder
  document.querySelectorAll('[data-i18n-placeholder]').forEach(el => {
    el.placeholder = t(el.dataset.i18nPlaceholder);
  });
  // title 属性
  document.querySelectorAll('[data-i18n-title]').forEach(el => {
    el.title = t(el.dataset.i18nTitle);
  });
  // innerHTML（HTMLタグを含む場合）
  document.querySelectorAll('[data-i18n-html]').forEach(el => {
    el.innerHTML = t(el.dataset.i18nHtml);
  });
  // <html lang=""> の更新
  document.documentElement.lang = currentLang;
}

function switchLang(lang) {
  currentLang = lang;
  localStorage.setItem('lang', lang);

  // 言語バーのアクティブ状態更新
  document.querySelectorAll('.lang-btn').forEach(btn => {
    btn.classList.toggle('active', btn.dataset.lang === lang);
  });

  applyI18n();

  // 動的生成済みコンテンツを再描画
  if (currentStep === 1) {
    renderSection(currentSectionIdx); // 質問・セクション名を再描画
    renderOrgInfoStrip('orgInfoStripQ');
    updateNav();
    updateTabs();
  } else if (currentStep === 2) {
    showResults(); // 評価結果ページを再描画
  }
}
```

---

## 3. メニューバーの言語切替UI

### 3.1 HTML 構造

```html
<nav class="menu-bar">
  <!-- 既存メニュー項目 -->
  <button class="menu-item" data-i18n="menu_guide" onclick="openGuideModal()">...</button>
  ...

  <!-- 言語切替バー（右端に配置） -->
  <div class="lang-switcher" style="margin-left:auto;">
    <button class="lang-btn active" data-lang="ja" onclick="switchLang('ja')">日本語</button>
    <span class="lang-sep">|</span>
    <button class="lang-btn" data-lang="en" onclick="switchLang('en')">ENGLISH</button>
  </div>
</nav>
```

### 3.2 CSS

```css
.lang-switcher {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 0 12px;
  border-left: 1px solid var(--border);
}
.lang-btn {
  background: none;
  border: none;
  font-size: 12px;
  color: var(--text-muted);
  cursor: pointer;
  padding: 2px 6px;
  border-radius: 3px;
  transition: all .15s;
}
.lang-btn:hover { color: var(--text); }
.lang-btn.active {
  color: var(--accent);
  font-weight: 700;
  background: rgba(59,130,246,0.1);
}
.lang-sep { color: var(--border); font-size: 11px; }
```

---

## 4. 質問JSONの多言語対応

### 4.1 変更後のスキーマ

現在の単一言語フィールドを、`ja`/`en` オブジェクトで包む形式に変更する。

```json
{
  "id": "sim3",
  "label":    { "ja": "SIM3",    "en": "SIM3" },
  "sublabel": { "ja": "CSIRTインシデント管理成熟度モデル", "en": "CSIRT Incident Management Maturity Model" },
  "frameworks": {
    "ja": ["SIM3 v1.1"],
    "en": ["SIM3 v1.1"]
  },
  "sections": [
    {
      "id": "sim3_o",
      "label": { "ja": "O. 組織（Organisation）", "en": "O. Organisation" },
      "ref": "ORGANISATION",
      "subsections": [
        {
          "title": {
            "ja": "O1〜O5 — ミッション・体制・リソース・マネジメント",
            "en": "O1–O5 — Mission, Structure, Resources & Management"
          },
          "questions": [
            {
              "no": "O1",
              "text": {
                "ja": "CSIRTのミッション・権限の範囲・提供するサービスの種類が文書化されており、コンスティチュエンシー（対応対象組織）に公表されている",
                "en": "The CSIRT's mission, scope of authority, and service catalog are documented and published to its constituency."
              },
              "refs": ["SIM3 O1"]
            }
          ]
        }
      ]
    }
  ]
}
```

### 4.2 manifest.json の多言語対応

```json
{
  "id": "sim3",
  "label":    { "ja": "SIM3", "en": "SIM3" },
  "sublabel": { "ja": "CSIRTインシデント管理成熟度モデル", "en": "CSIRT Incident Mgmt Maturity Model" },
  "icon":     "🚨",
  "color":    "#7c3aed",
  "description": {
    "ja": "ENISA・TF-CSIRTが策定したCSIRT向けセキュリティインシデント管理成熟度モデル。...",
    "en": "The CSIRT Security Incident Management Maturity Model developed by ENISA and TF-CSIRT. ..."
  },
  "file":     "questions/sim3.json",
  "env":      null,
  "tags": {
    "ja": ["SIM3 v1.1"],
    "en": ["SIM3 v1.1"]
  },
  "refsFiles": ["content/refs_sim3.json"]
}
```

### 4.3 質問データのアクセスヘルパー

```javascript
// 現在の言語で文字列を取得（文字列 or {ja,en} オブジェクトに対応）
function loc(value) {
  if (typeof value === 'string') return value;
  return value[currentLang] || value.ja || '';
}

// 使用例
section.label         → loc(section.label)
subsection.title      → loc(subsection.title)
question.text         → loc(question.text)
fw.label              → loc(fw.label)
fw.description        → loc(fw.description)
```

---

## 5. refs_*.json の多言語対応

### 5.1 変更後のスキーマ

```json
{
  "_meta": {
    "version": "2.0",
    "framework": "SIM3 v1.1",
    "description": {
      "ja": "SIM3 v1.1 の識別番号定義。44件収録。",
      "en": "SIM3 v1.1 reference definitions. 44 entries."
    },
    "count": 44
  },
  "SIM3 O1": {
    "summary": {
      "ja": "SIM3 O1 — CSIRTのミッション・権限範囲・提供サービスの文書化と公表",
      "en": "SIM3 O1 — Document and publish the CSIRT's mission, mandate and service catalog"
    },
    "detail": {
      "ja": "【正式名称】O1: Mission, Constituency and Mandate\n【目的・概要】\nCSIRTのミッション...\n【主な要件】\n・...\n【実装例】\n...",
      "en": "【Official Name】O1: Mission, Constituency and Mandate\n【Purpose & Overview】\nDefine and publish...\n【Key Requirements】\n・...\n【Implementation Examples】\n..."
    }
  }
}
```

### 5.2 ツールチップ・ポップオーバーの言語切替

```javascript
function showRefTooltip(event, ref) {
  const entry = REF_DEFINITIONS[ref];
  if (!entry) return;
  const summary = loc(entry.summary);  // loc() で現在言語を選択
  // ...ツールチップ表示
}

async function fetchRefDefinition(ref) {
  const entry = REF_DEFINITIONS[ref];
  if (entry) {
    const detail = loc(entry.detail);  // 現在言語の detail を使用
    // ...ポップオーバー表示
  }
}
```

---

## 6. 動的コンテンツの再描画戦略

言語切替時に再描画が必要な箇所を整理する。

| コンポーネント | 再描画方法 | 備考 |
|---|---|---|
| メニューバーラベル | `applyI18n()` でDOMを一括更新 | `data-i18n` 属性で管理 |
| フレームワークカード | `renderFrameworkGrid()` を再呼び出し | manifest の `loc()` で表示 |
| セクションタブ | `buildTabs()` を再呼び出し | sections の `loc()` で表示 |
| 質問テキスト | `renderSection(currentSectionIdx)` を再呼び出し | questions の `loc()` で表示 |
| 識別番号ツールチップ | 次回ホバー時に `loc()` で自動更新 | キャッシュ（`_defCache`）は**言語切替時にクリア** |
| 識別番号ポップオーバー | 次回クリック時に `loc()` で自動更新 | 同上 |
| 情報バー | `renderOrgInfoStrip()` を再呼び出し | ラベルは `t()` で、入力値はそのまま |
| 成熟度ラベル | `showResults()` を再呼び出し | `t('maturity_N')` で表示 |
| 評価結果ページ全体 | `showResults()` を再呼び出し | |
| 回答値ボタン（はい等） | `renderSection()` を再呼び出し | `t('ans_yes')` 等で表示 |
| モーダルタイトル・本文 | `applyI18n()` でDOMを一括更新 | `data-i18n` 属性で管理 |
| PDFレポート | 出力時の `currentLang` で生成 | 出力時点の言語で PDF を生成 |
| **メモ・総合評価コメント** | **変更しない** | 利用者入力値は言語に依存しない |
| **組織情報の入力値** | **変更しない** | 利用者入力値は言語に依存しない |

### 6.1 `_defCache` の言語対応

ポップオーバーの整形済みキャッシュは言語ごとに分ける。

```javascript
// 変更前
const _defCache = {};

// 変更後（言語をキーの一部に含める）
const _defCache = {};
// キー形式: "SIM3 O1:ja" / "SIM3 O1:en"
function defCacheKey(ref) { return `${ref}:${currentLang}`; }
```

---

## 7. 解説書（guide.json）の多言語対応

```json
{
  "title": {
    "ja": "解説書 — 使い方ガイド",
    "en": "User Guide"
  },
  "sections": [
    {
      "icon": "🎯",
      "heading": { "ja": "このシステムについて", "en": "About This System" },
      "content": [
        {
          "type": "p",
          "text": {
            "ja": "本システムは...",
            "en": "This system is a..."
          }
        }
      ]
    }
  ]
}
```

解説書モーダルのレンダリング関数も `loc()` を通じて現在言語のテキストを表示する。

---

## 8. 保存ファイルへの言語情報の記録

保存ファイルに `lang` フィールドを追加し、読み込み時に言語を復元する（任意）。

```json
{
  "version": "2.8",
  "lang": "en",
  "savedAt": "2026-03-21T10:30:00.000Z",
  "orgInfo": { ... },
  ...
}
```

> ただし、読み込み後に現在の言語設定を上書きするかどうかはUXの観点から要検討。  
> **推奨：** 読み込んでも言語は上書きしない（ユーザーが選んだ言語を維持する）。

---

## 9. 実装ステップ（推奨順序）

| フェーズ | 内容 | 概算作業量 |
|---|---|---|
| Phase 1 | I18N辞書の定義・`t()` / `applyI18n()` の実装・UIラベルの `data-i18n` 化 | 大（HTML全体の属性付与） |
| Phase 2 | 言語切替バーのUI実装・`switchLang()` 実装 | 小 |
| Phase 3 | manifest.json と全質問JSONの多言語形式への移行 | 大（全10フレームワーク） |
| Phase 4 | 全 refs_*.json の多言語形式への移行 | 大（601件の英訳） |
| Phase 5 | `loc()` の全描画関数への適用・`_defCache` のキー変更 | 中 |
| Phase 6 | guide.json の多言語対応 | 中 |
| Phase 7 | PDFレポートの多言語対応 | 小 |
| Phase 8 | 統合テスト・言語切替時の全画面確認 | 中 |

> **最大の工数は Phase 3・4（JSONの英訳）** であり、AIによる自動生成が有効。  
> `prompt_fw_addition.md` に倣い、既存JSONの英訳生成用プロンプトを別途作成することを推奨。

---

## 10. 後方互換性

- 既存の保存ファイル（`version: "2.6"` 等）は `lang` フィールドがないため、読み込み時はデフォルト（`ja`）で扱う。
- 現行の文字列フォーマット（`label: "string"` 形式）と多言語フォーマット（`label: {ja, en}` 形式）の両方を `loc()` が透過的に処理するため、移行中の混在状態でも動作する。
- 未翻訳キーは `I18N.ja` の値にフォールバックするため、英語コンテンツが未定義の状態でも日本語で動作する。

---

## 11. 設計上の注意点・リスク

| 項目 | 内容 |
|---|---|
| 翻訳品質 | 技術用語の英訳は専門用語の統一が必要。フレームワーク固有の英語正式名称（SIM3のパラメータ名等）はオリジナル英語を使用する。 |
| フォント | 英語・日本語を同一CSSフォントスタックで処理可能（Noto Sans JPは英日両対応）。 |
| 日付フォーマット | `fmtDate()` を言語に応じて切り替える（ja: `2026年3月21日` / en: `March 21, 2026`）。 |
| 数字フォーマット | スコア表示（`%`）は言語依存なし。 |
| PDFのフォント | Noto Sans JPで英日両対応のため変更不要。 |
| 文字列の長さ変化 | 英語ラベルは日本語より長くなる場合がある。UIのレイアウトが崩れないよう `text-overflow: ellipsis` や `flex-wrap` の調整が必要。 |
