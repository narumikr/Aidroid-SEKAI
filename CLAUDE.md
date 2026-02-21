# プロジェクト全体ガイドライン

## プロジェクト概要

<<<<<<< Updated upstream
プロジェクトではSkillが使用可能です。必要に応じて利用してください。
=======
本プロジェクトでは、XXX を提供します。
このファイルは、プロジェクト全体で共通して参照する運用ガイドです。
>>>>>>> Stashed changes

## ディレクトリ構成

プロジェクト構成は以下です。

```text
sekai-fw/
├── .github/            # CI/CD設定
├── .claude/            # Agentルール・Skills
├── backend/            # バックエンド実装（FastAPI）
├── frontend/           # フロントエンド実装（React）
├── shared/             # 共有資産（API仕様・生成物など）
├── docs/               # 設計・計画ドキュメント
├── README.md           # プロジェクト概要
└── justfile            # 開発用タスク定義
```