---
name: path-sanitizer
description: |
  ファイルパス文字列の検証とサニタイズを行う汎用スキル。
  ディレクトリトラバーサル・絶対パス・制御文字などの危険なパターンを検出し、
  安全なパスを返す。外部ファイルシステムへのアクセスを行わず、決定論的に動作する。
---

# path-sanitizer スキル

## 概要

ファイルパス文字列を検証・サニタイズする汎用スキル。実際のファイルシステムへのアクセスは行わない。

---

## 入力

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `path` | string | ✅ | 検証・サニタイズ対象のファイルパス文字列 |
| `options.allow_absolute` | boolean | ❌（デフォルト: `false`） | 絶対パス（`/` 始まり、または Windows の `C:\` 形式）を許可するか |
| `options.allow_traversal` | boolean | ❌（デフォルト: `false`） | `..` によるディレクトリトラバーサルを許可するか |
| `options.separator` | string | ❌（デフォルト: `/`） | パス区切り文字。`/` または `\\` のいずれかのみ指定可能 |

---

## 出力

| フィールド | 型 | 説明 |
|---|---|---|
| `valid` | boolean | 検証成功（危険パターンが含まれない）の場合 `true` |
| `errors` | array of string | 検証失敗の原因メッセージのリスト。成功時は空配列 |
| `sanitized` | string / null | サニタイズ後のパス文字列。`valid: true` の場合のみ設定。`valid: false` の場合は `null` |

---

## 処理内容

以下の順序でチェックを適用する。いずれか 1 つでも失敗した時点で `valid: false` を返す。

1. **制御文字チェック**
   - U+0000〜U+001F および U+007F〜U+009F の制御文字が含まれる場合はエラー

2. **Null バイトチェック**
   - `\0`（U+0000）が含まれる場合はエラー

3. **絶対パスチェック**（`allow_absolute: false` の場合）
   - `/` 始まりのパスはエラー
   - `[A-Za-z]:\` または `[A-Za-z]:/` 形式の Windows 絶対パスはエラー
   - `\\server\share` 形式の UNC パスはエラー

4. **ディレクトリトラバーサルチェック**（`allow_traversal: false` の場合）
   - パスセグメントに `..` が含まれる場合はエラー
   - URL エンコードされた `..`（`%2e` と `.` のすべての大文字小文字の組み合わせ、例: `%2e%2e`、`%2E%2E`、`%2e%2E`、`%2E%2e`、`%2e.`、`.%2e`、`%2E.`、`.%2E`）はエラー

5. **区切り文字正規化**
   - `separator` の指定に従い、`/` と `\\` を統一する

6. **重複区切り文字の除去**
   - 連続する区切り文字（例: `//`、`\\\\`）を単一の区切り文字に正規化する

7. **先頭・末尾の区切り文字除去**
   - `allow_absolute: false` の場合、先頭の区切り文字を除去する
   - 末尾の区切り文字を除去する

すべてのチェックを通過した場合に `valid: true`、`sanitized` に正規化済みパスを設定して返す。

---

## 副作用

なし。実際のファイルシステムへのアクセスは行わない（不変操作）。

---

## エラー条件

| 条件 | エラー内容 |
|---|---|
| `path` が文字列型でない | `TypeError: path must be a string` を返す |
| `path` が `null` または未指定 | `ValueError: path is required` を返す |
| `options.separator` が `/` でも `\\` でもない | `ValueError: separator must be "/" or "\\"` を返す |
| `options` に未知のキーが含まれる | `ValueError: unknown option key: <key>` を返す |
| `options` の boolean フィールドが boolean 型でない | `TypeError: option <key> must be a boolean` を返す |

エラー時は `valid` / `errors` / `sanitized` を返さない。エラーを飲み込まず、呼び出し元に伝播させる。自動リトライは行わない。
