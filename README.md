---
title: "Python Feature Reference Vault (3.5 - 3.15)"
aliases: [Vault Home, ホーム, インデックス]
category: meta
python_version: "all"
peps: []
created: 2026-10-26
updated: 2026-10-26
tags: [index, portal, meta]
---

# Python Feature Reference Vault (3.5 〜 3.15)

Python 3.5 から最新・次期仕様の Python 3.15 までの言語仕様・標準ライブラリ・型システムの進化を日本語で体系的に学べるナレッジベースです。

外部コミュニティプラグインに依存せず、**Obsidian標準のコア機能（フロントマター、タグ、ファイルリンク、プロパティ）のみ**で快適にブラウジングおよび全文検索ができるように設計されています。

---

## 📚 クイックナビゲーション

- 🗺️ **全体マップ**: [[ROADMAP|ROADMAP（トピック一覧・ID体系・設計書）]]
- 📖 **一次資料・参照方針**: [[sources/index|一次資料ガイド（PEPs & What's New）]]

### 🚀 バージョン別ポータル（3.5 〜 3.15）

|      バージョン      |  ステータス   | 概要ノート                              | 主な注目トピック                                                                                                                                                                                                      |
| :-------------: | :------: | :--------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Python 3.5**  |   EOL    | [[versions/py35\|Python 3.5 概要]]   | [[language/py35/101-async-await\|async/await 構文]], [[typing/py35/102-type-hints-basics\|型ヒント基礎]]                                                                                                              |
| **Python 3.6**  |   EOL    | [[versions/py36\|Python 3.6 概要]]   | [[language/py36/201-fstrings\|f-strings]], 変数型注釈                                                                                                                                                              |
| **Python 3.7**  |   EOL    | [[versions/py37\|Python 3.7 概要]]   | [[library/py37/301-dataclasses\|dataclasses]], 辞書順序の言語保証                                                                                                                                                      |
| **Python 3.8**  |   EOL    | [[versions/py38\|Python 3.8 概要]]   | [[language/py38/401-walrus-operator\|代入式 :=]], 位置専用引数 `/`                                                                                                                                                     |
| **Python 3.9**  | Security | [[versions/py39\|Python 3.9 概要]]   | [[typing/py39/502-builtin-generic-types\|組み込みジェネリクス]], [[language/py39/501-dict-union-operators\|辞書結合 \|]]                                                                                                    |
| **Python 3.10** | Security | [[versions/py310\|Python 3.10 概要]] | [[language/py310/601-pattern-matching\|match/case]], [[typing/py310/602-union-operator\|型ユニオン \|]]                                                                                                            |
| **Python 3.11** |  Bugfix  | [[versions/py311\|Python 3.11 概要]] | [[language/py311/701-exception-groups\|例外グループ except*]], [[library/py311/702-tomllib\|tomllib]]                                                                                                               |
| **Python 3.12** |  Bugfix  | [[versions/py312\|Python 3.12 概要]] | [[language/py312/801-type-parameter-syntax\|type文・型パラメータ構文]]                                                                                                                                                  |
| **Python 3.13** |  Latest  | [[versions/py313\|Python 3.13 概要]] | [[language/py313/901-free-threaded-cpython\|フリースレッドCPython (nogil)]], [[typing/py313/903-typeis-and-typevar-defaults\|TypeIs]]                                                                                |
| **Python 3.14** |  In Dev  | [[versions/py314\|Python 3.14 概要]] | [[language/py314/1401-template-strings\|t-strings (PEP 750)]], [[typing/py314/1402-deferred-evaluation-of-annotations\|型注釈遅延評価 (PEP 649)]]                                                                    |
| **Python 3.15** |  Stable  | [[versions/py315\|Python 3.15 概要]] | [[language/py315/1501-lazy-imports\|lazy imports (PEP 810)]], [[language/py315/1502-frozendict-and-sentinel\|frozendict (PEP 814)]], [[language/py315/1503-unpacking-in-comprehensions\|内包表記アンパック (PEP 798)]] |

---

## 🏷️ タグ体系

- バージョン: `#py35`, `#py36`, `#py37`, `#py38`, `#py39`, `#py310`, `#py311`, `#py312`, `#py313`, `#py314`, `#py315`
- カテゴリ: `#language`（構文・実行モデル）, `#library`（標準ライブラリ）, `#typing`（型システム）
- テーマ: `#async`, `#syntax`, `#gil`, `#concurrency`, `#experimental`, `#performance`

