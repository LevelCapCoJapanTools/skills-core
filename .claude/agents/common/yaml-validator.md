---
name: yaml-validator
description: |
  YAML 文字列の構文バリデーションを行う汎用スキル。
  外部サービスへのアクセスを行わず、決定論的に動作する。
---

# yaml-validator スキル

## 概要

YAML 文字列の構文チェックを行い、パース済みオブジェクトを返す汎用スキル。

---

## 入力

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `yaml_text` | string | ✅ | バリデーション対象の YAML 文字列 |
| `options.allow_multiple_documents` | boolean | ❌（デフォルト: `false`） | `---` 区切りの複数ドキュメントを許可するか |
| `options.forbid_duplicate_keys` | boolean | ❌（デフォルト: `true`） | 同一スコープ内の重複キーをエラーとするか |

---

## 出力

| フィールド | 型 | 説明 |
|---|---|---|
| `valid` | boolean | バリデーション成功の場合 `true` |
| `errors` | array of string | バリデーション失敗の原因メッセージのリスト（行番号・列番号を含む）。成功時は空配列 |
| `parsed` | object / array / null | 構文チェック成功時のパース済みオブジェクト。`allow_multiple_documents: true` の場合は配列。失敗時は `null` |

---

## 処理内容

以下の順序で処理を行う。

1. **ドキュメント数チェック**
   - `allow_multiple_documents: false`（デフォルト）の場合、`---` マーカーによって区切られたドキュメントが 2 個以上存在するならエラーとして `valid: false`、`errors` にメッセージを設定して処理を終了する

2. **構文チェック**
   - YAML 1.2 仕様に従い `yaml_text` をパースする
   - パース失敗の場合は `valid: false`、`errors` に行番号・列番号付きエラーメッセージ、`parsed: null` を返して処理を終了する
   - パース成功の場合は `parsed` にパース結果を設定する

3. **重複キーチェック**（`forbid_duplicate_keys: true` の場合）
   - パース結果のすべてのマッピングスコープで重複キーを検出する
   - 重複が存在する場合は `valid: false`、`errors` に重複キー名とパスを設定する

4. **成功**
   - すべてのチェックを通過した場合に `valid: true`、`errors: []` を返す

---

## 副作用

なし。入力データは変更しない（不変操作）。

---

## エラー条件

| 条件 | エラー内容 |
|---|---|
| `yaml_text` が文字列型でない | `TypeError: yaml_text must be a string` を返す |
| `yaml_text` が `null` または未指定 | `ValueError: yaml_text is required` を返す |
| `options` に未知のキーが含まれる | `ValueError: unknown option key: <key>` を返す |
| `options` の値が boolean 型でない | `TypeError: option <key> must be a boolean` を返す |

エラー時は `valid` / `errors` / `parsed` を返さない。エラーを飲み込まず、呼び出し元に伝播させる。自動リトライは行わない。
