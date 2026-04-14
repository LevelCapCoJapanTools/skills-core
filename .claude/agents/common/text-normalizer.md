---
name: text-normalizer
description: |
  テキストの正規化を行う汎用スキル。
  全角・半角変換、空白の統一、改行コード正規化、制御文字除去を決定論的に実行する。
  外部環境に依存せず、同一入力に対して常に同一出力を返す。
---

# text-normalizer スキル

## 概要

テキストを正規化する汎用スキル。以下の変換を決定論的に適用する。

---

## 入力

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `text` | string | ✅ | 正規化対象のテキスト |
| `options.normalize_whitespace` | boolean | ❌（デフォルト: `true`） | 連続する空白を単一スペースに統一する |
| `options.strip_control_chars` | boolean | ❌（デフォルト: `true`） | タブ（U+0009）・改行（U+000A）以外の制御文字（U+0000〜U+001F、U+007F〜U+009F）を除去する |
| `options.normalize_newlines` | boolean | ❌（デフォルト: `true`） | CRLF / CR を LF に統一する |
| `options.fullwidth_to_halfwidth` | boolean | ❌（デフォルト: `false`） | 全角英数字・記号（U+FF01〜U+FF5E）を半角に変換する |
| `options.trim` | boolean | ❌（デフォルト: `true`） | テキスト先頭・末尾の空白文字を除去する |

---

## 出力

| フィールド | 型 | 説明 |
|---|---|---|
| `result` | string | 正規化後のテキスト |
| `changed` | boolean | 入力から変更があった場合 `true` |

---

## 処理内容

以下の順序で変換を適用する。順序を変えてはならない。

1. **改行コード正規化**（`normalize_newlines: true` の場合）
   - `\r\n`（CRLF）を `\n`（LF）に置換する
   - `\r`（CR）単独を `\n`（LF）に置換する

2. **制御文字除去**（`strip_control_chars: true` の場合）
   - タブ（U+0009）・改行（U+000A）を除く U+0000〜U+001F 範囲の文字を除去する（ステップ 1 で CR は LF に変換済みのため対象外）
   - U+007F〜U+009F 範囲の文字を除去する

3. **全角→半角変換**（`fullwidth_to_halfwidth: true` の場合）
   - U+FF01〜U+FF5E の全角 ASCII 互換文字を対応する半角文字（U+0021〜U+007E）に変換する
   - 全角スペース（U+3000）は半角スペース（U+0020）に変換する

4. **連続空白正規化**（`normalize_whitespace: true` の場合）
   - 1 個以上の連続するスペース・タブを単一の半角スペース（U+0020）に置換する
   - 行頭・行末の空白は `trim` の設定に従う（このステップでは除去しない）

5. **トリム**（`trim: true` の場合）
   - テキスト全体の先頭・末尾にある空白文字（スペース・タブ・改行）を除去する

---

## 副作用

なし。入力テキストは変更しない（不変操作）。

---

## エラー条件

| 条件 | エラー内容 |
|---|---|
| `text` が文字列型でない | `TypeError: text must be a string` を返す |
| `text` が `null` または未指定 | `ValueError: text is required` を返す |
| `options` に未知のキーが含まれる | `ValueError: unknown option key: <key>` を返す |
| `options` の値が boolean 型でない | `TypeError: option <key> must be a boolean` を返す |

エラー時は変換結果を返さない。エラーを飲み込まず、呼び出し元に伝播させる。自動リトライは行わない。
