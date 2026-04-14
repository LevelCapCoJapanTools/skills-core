---
name: json-validator
description: |
  JSON 文字列の構文バリデーションおよび JSON Schema に基づく構造バリデーションを行う汎用スキル。
  外部サービスへのアクセスを行わず、決定論的に動作する。
---

# json-validator スキル

## 概要

JSON 文字列の構文チェックおよびスキーマ適合チェックを行う汎用スキル。

---

## 入力

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `json_text` | string | ✅ | バリデーション対象の JSON 文字列 |
| `schema` | object | ❌ | パース済み JSON Schema オブジェクト（Draft 7 準拠）。指定しない場合は構文チェックのみ行う |

---

## 出力

| フィールド | 型 | 説明 |
|---|---|---|
| `valid` | boolean | バリデーション成功の場合 `true` |
| `errors` | array of string | バリデーション失敗の原因メッセージのリスト。成功時は空配列 |
| `parsed` | object / array / null | 構文チェック成功時のパース済みオブジェクト。失敗時は `null` |

---

## 処理内容

以下の順序で処理を行う。

1. **構文チェック**
   - `json_text` を JSON パースする
   - パース失敗の場合は `valid: false`、`errors` にエラーメッセージ、`parsed: null` を返して処理を終了する
   - パース成功の場合は `parsed` にパース結果を設定する

2. **スキーマバリデーション**（`schema` が指定された場合）
   - JSON Schema Draft 7 の仕様に従い、パース結果に対してスキーマバリデーションを行う
   - バリデーション失敗の場合は `valid: false` とし、`errors` に各違反のパスとメッセージを設定する
   - バリデーション成功の場合は `valid: true`、`errors: []` を返す

3. **スキーマ未指定の場合**
   - 構文チェック成功 → `valid: true`、`errors: []` を返す

---

## 副作用

なし。入力データは変更しない（不変操作）。

---

## エラー条件

| 条件 | エラー内容 |
|---|---|
| `json_text` が文字列型でない | `TypeError: json_text must be a string` を返す |
| `json_text` が `null` または未指定 | `ValueError: json_text is required` を返す |
| `schema` が指定されているが object 型でない | `TypeError: schema must be an object` を返す |
| `schema` 自体が有効な JSON Schema でない | `ValueError: schema is not a valid JSON Schema` を返す |

エラー時は `valid` / `errors` / `parsed` を返さない。エラーを飲み込まず、呼び出し元に伝播させる。自動リトライは行わない。
