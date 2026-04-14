---
name: markdown-section-extractor
description: |
  Markdown テキストから指定した見出しのセクション本文を抽出する汎用スキル。
  外部サービスへのアクセスを行わず、決定論的に動作する。
---

# markdown-section-extractor スキル

## 概要

Markdown テキストから指定した見出し（`#` 記法）のセクション本文を抽出する汎用スキル。

---

## 入力

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `markdown` | string | ✅ | 抽出対象の Markdown テキスト |
| `heading` | string | ✅ | 抽出したいセクションの見出しテキスト（`#` 記号を含まない文字列） |
| `options.heading_level` | integer | ❌（デフォルト: `null`） | 見出しレベルを限定する場合に指定（1〜6）。`null` の場合は全レベルを対象とする |
| `options.include_heading` | boolean | ❌（デフォルト: `false`） | 返却テキストに見出し行自体を含めるか |
| `options.include_subsections` | boolean | ❌（デフォルト: `true`） | 該当セクション内の下位見出しセクションも含めるか |

---

## 出力

| フィールド | 型 | 説明 |
|---|---|---|
| `found` | boolean | 指定見出しが見つかった場合 `true` |
| `content` | string / null | 抽出されたセクション本文。`found: false` の場合は `null` |
| `heading_level` | integer / null | 見つかった見出しのレベル（1〜6）。`found: false` の場合は `null` |

---

## 処理内容

以下の手順で処理を行う。

1. **Markdown パース**
   - `markdown` をテキストとして行単位に分割する
   - `#` 始まりの行を見出し行として識別する（`# 〜`、`## 〜`、`### 〜` など、ATX 見出し記法のみ対応）
   - Setext 見出し記法（`=====` / `-----` 下線形式）は対象外とする

2. **見出し検索**
   - 各見出し行について、`heading_level` が指定されていればレベルが一致するもののみを対象とする
   - 見出しテキスト（`#` と先頭空白を除いた部分）が `heading` と完全一致する最初の見出しを対象セクションとする

3. **セクション範囲の決定**
   - 対象見出し行の次の行から、以下の条件を満たす行が出現するまでをセクション本文とする
     - `include_subsections: true` の場合: 対象見出しと**同レベル以上**（レベル数値が小さい、または等しい）の見出し行が出現するまで（下位見出しはセクション本文に含まれる）
     - `include_subsections: false` の場合: **任意レベル**の見出し行が出現するまで（下位見出しの出現でもセクション本文は終了する）
   - 見出しが見つからない場合は `found: false`、`content: null`、`heading_level: null` を返す

4. **本文の組み立て**
   - `include_heading: true` の場合は見出し行を先頭に追加する
   - セクション本文の末尾の空行を除去する

5. **返却**
   - `found: true`、`content` に抽出テキスト、`heading_level` に見出しレベルを設定して返す

---

## 副作用

なし。入力テキストは変更しない（不変操作）。

---

## エラー条件

| 条件 | エラー内容 |
|---|---|
| `markdown` が文字列型でない | `TypeError: markdown must be a string` を返す |
| `markdown` が `null` または未指定 | `ValueError: markdown is required` を返す |
| `heading` が文字列型でない | `TypeError: heading must be a string` を返す |
| `heading` が `null` または未指定 | `ValueError: heading is required` を返す |
| `options.heading_level` が 1〜6 の整数でない | `ValueError: heading_level must be an integer between 1 and 6` を返す |
| `options` に未知のキーが含まれる | `ValueError: unknown option key: <key>` を返す |
| `options` の boolean フィールドが boolean 型でない | `TypeError: option <key> must be a boolean` を返す |

エラー時は `found` / `content` / `heading_level` を返さない。エラーを飲み込まず、呼び出し元に伝播させる。自動リトライは行わない。
