# skills-core

SSOT における「汎用スキル層」を担うリポジトリ。

Skill とは「再利用可能な操作単位」であり、複数のプロジェクトで共通に利用できる処理を指す。本リポジトリでは環境依存を排除し、決定論的に動作するスキルのみを定義する。

## 管理対象パス

| パス | 内容 |
|---|---|
| `.claude/agents/common/**` | 汎用スキル定義ファイル（Claude サブエージェント形式） |
| `sets/**` | セット定義 YAML ファイル |

## 汎用スキル一覧

| スキル | 概要 |
|---|---|
| `text-normalizer` | テキスト正規化（全角/半角変換、空白・改行コード統一、制御文字除去） |
| `json-validator` | JSON 構文チェックおよび JSON Schema バリデーション |
| `yaml-validator` | YAML 構文チェックおよび重複キー検出 |
| `path-sanitizer` | ファイルパスの検証・サニタイズ（トラバーサル・絶対パス・制御文字検出） |
| `markdown-section-extractor` | Markdown テキストから指定見出しのセクション本文を抽出 |

## セット定義一覧

| セット | 概要 |
|---|---|
| `sets/common-all.yml` | 全汎用スキルを含むセット |
| `sets/common-validation.yml` | バリデーション系スキル（JSON / YAML / パス）のセット |

## 基本原則

- **汎用性**: 特定プロジェクト依存は禁止
- **再利用性**: 複数案件で使えること
- **決定論**: 同じ入力 → 同じ出力
- **副作用の明示**: 何を変更するか明確にする

## 分離ルール

- このリポジトリは汎用スキルのみを扱う
- provider 依存のスキル → `skills-provider`
- ドメイン依存のスキル → `skills-domain`

## 参照ドキュメント

- [SSOT 配付システム仕様](docs/ssot-distribution-system-spec-final-2026-04-06.md)
- [カタログパス担当表](docs/catalog-path-ownership-draft.md)
