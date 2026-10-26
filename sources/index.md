---
title: 一次資料ガイドと参照方針
aliases: [一次資料, PEP参照方針, What's New Guide]
category: sources
python_version: "all"
peps: []
created: 2026-10-26
updated: 2026-10-26
tags: [sources, meta, documentation]
---

# 一次資料ガイドと参照方針

本Vault（Python Feature Reference Vault）は、Python 3.5からPython 3.15までの仕様変更・型ヒント・標準ライブラリの進化を一次資料に基づいて記録・解説するナレッジベースです。

---

## 1. 一次資料の参照基準

本Vaultのすべての記述は、以下の一次資料を根拠として作成されています。

1. **Python Enhancement Proposals (PEPs)**
   - 公式URL規則: `https://peps.python.org/pep-XXXX/`（4桁ゼロ埋め、例: `pep-0492`）
   - 各記事のフロントマター `peps` に該当番号を整数またはリストで記録し、本文末尾の「参考資料」で明示的にリンクします。

2. **What's New in Python**
   - 公式ドキュメントの各バージョンリリースノート: `https://docs.python.org/3/whatsnew/3.X.html`
   - 実装の詳細、C-APIの変更、非推奨化スケジュール（Deprecation timelines）の確認に使用。

3. **CPython ソースコードおよびGitHubリポジトリ**
   - CPython Gitリポジトリ: `https://github.com/python/cpython`
   - Issue / PR、およびコミット履歴（特に3.14/3.15等の開発版や実験的機能の進捗）。

---

## 2. バージョンステータスの分類基準

Pythonのリリースサイクルと本Vaultにおけるステータス定義は以下の通りです。

| ステータス | 対象バージョン（2026年時点） | 特徴・記述方針 |
|---|---|---|
| **EOL (End of Life)** | 3.5, 3.6, 3.7, 3.8 | セキュリティ修正終了。レガシーコード保守・移行観点の記述を中心とする。 |
| **Security Support** | 3.9, 3.10 | セキュリティバグ修正のみ提供。新規導入は不可だが現場で多数稼働。 |
| **Bugfix Support** | 3.11, 3.12 | 通常のバグ修正が提供される現役安定版。 |
| **Latest Stable** | 3.13, 3.14, 3.15 | 3.13〜3.15で導入された新構文・型システム・標準ライブラリ。 |
| **Release / Stable** | 3.15 | PEP 790 で仕様策定・リリースされた安定版。 |

---

## 3. Obsidian標準機能によるナビゲーション方針

外部プラグインを一切導入せずとも、以下の標準機能だけで快適に探索・学習できるように構成しています。

- **双方向リンク (`[[...]]`)**:
  - 相対リンクではなくVault内ルートまたはファイル名を指定し、エイリアス表記 `[[ノート名|表示名]]`（テーブル内では `[[ノート名\|表示名]]`）のルールを統一。
- **YAMLフロントマター**:
  - 各ノートの先頭にメタデータを定義。Obsidianのプロパティ表示およびファイル検索機能で絞り込み可能。
- **タグ構造**:
  - `#py35` 〜 `#py315`: バージョン別タグ
  - `#syntax`, `#library`, `#typing`, `#performance`, `#async`: トピック・関心事別タグ
- **固定連番ID (NNN-slug.md)**:
  - ファイルの並び順が崩れず、時系列かつ論理順に整列。
