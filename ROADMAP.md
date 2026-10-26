---
title: "Python Feature Reference Vault (3.5 〜 3.15) ロードマップ"
aliases: [ROADMAP, ロードマップ, 全体目次]
category: meta
python_version: "all"
peps: []
created: 2026-10-26
updated: 2026-10-26
tags: [roadmap, index, meta]
---

# Python Feature Reference Vault (3.5 〜 3.15) ロードマップ

本ドキュメントでは、Python 3.5から最新安定版 Python 3.15 までの言語仕様・標準ライブラリ・型システムの進化ロードマップ、採択PEP番号、連番固定ID体系、および各トピック記事へのリンクを定義します。

---

## 1. ディレクトリ構造

本VaultはObsidianのコア機能（標準機能）のみで検索・グラフビュー・プロパティ表示が完結するよう、以下のカテゴリ・バージョン階層で構成されています。

```text
python-feature-reference-vault/
├── README.md               # Vaultトップポータル・利用ガイド
├── ROADMAP.md              # 本ドキュメント（トピック一覧・ID体系・リンク集）
├── sources/
│   └── index.md            # 一次資料ガイド・PEP/What's New参照基準
├── versions/               # 各バージョン（3.5〜3.15）の概要・タイムライン・変更まとめ
│   ├── py35.md
│   ├── py36.md
│   ├── ...
│   ├── py314.md
│   └── py315.md
├── language/               # 言語仕様・構文・実行モデル（py35〜py315）
├── library/                # 標準ライブラリの追加・刷新（py35〜py315）
└── typing/                 # 型ヒント・型システムの進化（py35〜py315）
```

---

## 2. 記事の命名規則と固定連番ID体系

- パス形式: `分類/pyXX/NNN-slug.md`
  - 分類: `language`, `library`, `typing`
  - pyXX: `py35` 〜 `py315`
  - NNN: 3桁〜4桁固定連番（100番台: py35, 200番台: py36, ..., 1500番台: py315）
  - slug: 内容を表す簡潔な英数字ケバブケース（ASCIIのみ）
- フロントマター統一スキーマ:
  ```yaml
  ---
  title: タイトル（日本語）
  aliases: [別名1, 別名2]
  category: language | library | typing | versions | sources | meta
  python_version: "3.X" | "all"
  peps: [000]
  created: YYYY-MM-DD
  updated: YYYY-MM-DD
  tags: [py3X, syntax/library/typing, ...]
  ---
  ```

---

## 3. 全バージョン別トピック・ロードマップ一覧

以下の各トピック名は対応するObsidianノートへのリンクです（Markdownテーブルの列崩れを防ぐため `パス|表示名` 形式でパイプをエスケープしています）。

| ID | バージョン | 分類 | トピック名（ノートリンク） | 該当PEP / 仕様 | ステータス |
|:---:|:---:|:---:|:---|:---|:---:|
| `101` | [[versions/py35\|3.5]] | `language` | [[language/py35/101-async-await\|async / await 構文のネイティブ導入]] | PEP 492 | 確定 (EOL) |
| `102` | [[versions/py35\|3.5]] | `typing` | [[typing/py35/102-type-hints-basics\|型ヒント（Type Hints）基本システムの導入]] | PEP 484, PEP 483 | 確定 (EOL) |
| `103` | [[versions/py35\|3.5]] | `language` | [[language/py35/103-unpacking-generalizations\|辞書・リストのアンパック一般化 (* / **)]] | PEP 448 | 確定 (EOL) |
| `104` | [[versions/py35\|3.5]] | `library` | [[library/py35/104-subprocess-run\|subprocess.run() の導入]] | - | 確定 (EOL) |
| `201` | [[versions/py36\|3.6]] | `language` | [[language/py36/201-fstrings\|フォーマット済み文字列リテラル（f-strings）]] | PEP 498 | 確定 (EOL) |
| `202` | [[versions/py36\|3.6]] | `typing` | [[typing/py36/202-variable-annotations\|変数アノテーション構文]] | PEP 526 | 確定 (EOL) |
| `203` | [[versions/py36\|3.6]] | `language` | [[language/py36/203-async-generators-comprehensions\|非同期ジェネレータと内包表記]] | PEP 525, PEP 530 | 確定 (EOL) |
| `204` | [[versions/py36\|3.6]] | `library` | [[library/py36/204-secrets-module\|secrets モジュールの追加]] | PEP 506 | 確定 (EOL) |
| `205` | [[versions/py36\|3.6]] | `library` | [[library/py36/205-pathlib-standardization\|pathlib の実用標準化（ファイルシステムパスプロトコル）]] | PEP 519 | 確定 (EOL) |
| `301` | [[versions/py37\|3.7]] | `library` | [[library/py37/301-dataclasses\|データクラス (dataclasses) の導入]] | PEP 557 | 確定 (EOL) |
| `302` | [[versions/py37\|3.7]] | `typing` | [[typing/py37/302-postponed-annotations\|型アノテーション遅延評価 (from __future__ import annotations)]] | PEP 563 | 確定 (EOL) |
| `303` | [[versions/py37\|3.7]] | `language` | [[language/py37/303-dict-order-guaranteed\|辞書順序の言語仕様としての保証]] | - | 確定 (EOL) |
| `304` | [[versions/py37\|3.7]] | `library` | [[library/py37/304-contextvars\|コンテキスト変数 (contextvars) の導入]] | PEP 567 | 確定 (EOL) |
| `401` | [[versions/py38\|3.8]] | `language` | [[language/py38/401-walrus-operator\|代入式（Walrus Operator :=）]] | PEP 572 | 確定 (EOL) |
| `402` | [[versions/py38\|3.8]] | `language` | [[language/py38/402-positional-only-parameters\|位置専用パラメータ (/)]] | PEP 570 | 確定 (EOL) |
| `403` | [[versions/py38\|3.8]] | `typing` | [[typing/py38/403-typing-extensions-standardized\|Literal, Final, TypedDict, Protocol]] | PEP 586, 591, 589, 544 | 確定 (EOL) |
| `404` | [[versions/py38\|3.8]] | `library` | [[library/py38/404-asyncio-robustness\|asyncio の堅牢化と asyncio.run() 定着]] | - | 確定 (EOL) |
| `405` | [[versions/py38\|3.8]] | `library` | [[library/py38/405-importlib-metadata\|パッケージメタデータ取得 importlib.metadata]] | PEP 566 | 確定 (EOL) |
| `501` | [[versions/py39\|3.9]] | `language` | [[language/py39/501-dict-union-operators\|辞書結合演算子 (&#124;, &#124;=)]] | PEP 584 | セキュリティ保守 |
| `502` | [[versions/py39\|3.9]] | `typing` | [[typing/py39/502-builtin-generic-types\|組み込みコレクションの型アノテーションジェネリクス化]] | PEP 585 | セキュリティ保守 |
| `503` | [[versions/py39\|3.9]] | `library` | [[library/py39/503-zoneinfo\|zoneinfo モジュールの導入（標準IANAタイムゾーン）]] | PEP 615 | セキュリティ保守 |
| `504` | [[versions/py39\|3.9]] | `language` | [[language/py39/504-relaxed-decorator-grammar\|より柔軟なデコレータ文法]] | PEP 614 | セキュリティ保守 |
| `601` | [[versions/py310\|3.10]] | `language` | [[language/py310/601-pattern-matching\|構造的パターンマッチング (match / case)]] | PEP 634, 635, 636 | セキュリティ保守 |
| `602` | [[versions/py310\|3.10]] | `typing` | [[typing/py310/602-union-operator\|型ユニオン演算子 (X &#124; Y)]] | PEP 604 | セキュリティ保守 |
| `603` | [[versions/py310\|3.10]] | `language` | [[language/py310/603-parenthesized-context-managers\|括弧付きコンテキストマネージャ（複数行構文）]] | - | セキュリティ保守 |
| `604` | [[versions/py310\|3.10]] | `typing` | [[typing/py310/604-typeguard-and-typealias\|TypeGuard と明示的型エイリアス (TypeAlias)]] | PEP 647, PEP 613 | セキュリティ保守 |
| `701` | [[versions/py311\|3.11]] | `language` | [[language/py311/701-exception-groups\|例外グループと except* 構文]] | PEP 654 | バグ修正サポート |
| `702` | [[versions/py311\|3.11]] | `library` | [[library/py311/702-tomllib\|tomllib モジュールの標準搭載]] | PEP 680 | バグ修正サポート |
| `703` | [[versions/py311\|3.11]] | `typing` | [[typing/py311/703-self-type-and-typevartuple\|Self 型と可変長ジェネリクス (TypeVarTuple)]] | PEP 673, PEP 646 | バグ修正サポート |
| `704` | [[versions/py311\|3.11]] | `language` | [[language/py311/704-faster-cpython-interpreter\|高速CPython（Faster CPython: 特化適応型インタプリタ）]] | - | バグ修正サポート |
| `705` | [[versions/py311\|3.11]] | `language` | [[language/py311/705-fine-grained-error-locations\|トレースバックでの詳細なエラー箇所下線表示]] | PEP 657 | バグ修正サポート |
| `801` | [[versions/py312\|3.12]] | `language` | [[language/py312/801-type-parameter-syntax\|型パラメータ構文 (type 文, ジェネリック関数/クラス)]] | PEP 695 | バグ修正サポート |
| `802` | [[versions/py312\|3.12]] | `language` | [[language/py312/802-syntactic-formalization-fstrings\|f-strings の文法制限緩和（PEP 701）]] | PEP 701 | バグ修正サポート |
| `803` | [[versions/py312\|3.12]] | `typing` | [[typing/py312/803-override-decorator\|@override デコレータによる安全な派生実装]] | PEP 698 | バグ修正サポート |
| `804` | [[versions/py312\|3.12]] | `language` | [[language/py312/804-per-interpreter-gil\|サブインタプリタごとのGIL分離（Per-Interpreter GIL）]] | PEP 684 | バグ修正サポート |
| `901` | [[versions/py313\|3.13]] | `language` | [[language/py313/901-free-threaded-cpython\|実験的フリースレッドCPython（GIL無効化モード）]] | PEP 703 | 確定・実験的提供 |
| `902` | [[versions/py313\|3.13]] | `language` | [[language/py313/902-experimental-jit\|実験的JITコンパイラ基盤（Copy-and-Patch JIT）]] | PEP 744 | 確定・実験的提供 |
| `903` | [[versions/py313\|3.13]] | `typing` | [[typing/py313/903-typeis-and-typevar-defaults\|TypeIs（狭小化の改善）とデフォルト型引数]] | PEP 742, PEP 696 | バグ修正サポート |
| `904` | [[versions/py313\|3.13]] | `library` | [[library/py313/904-enhanced-interactive-repl\|新しい対話型REPL環境（PyRepl統合・カラー表示）]] | - | バグ修正サポート |
| `1401` | [[versions/py314\|3.14]] | `language` | [[language/py314/1401-template-strings\|テンプレート文字列（t-strings: PEP 750）]] | PEP 750 | 確定 / 開発中 |
| `1402` | [[versions/py314\|3.14]] | `typing` | [[typing/py314/1402-deferred-evaluation-of-annotations\|型注釈のオンデマンド遅延評価（PEP 649 / PEP 749）]] | PEP 649, PEP 749 | 確定 / 開発中 |
| `1403` | [[versions/py314\|3.14]] | `language` | [[language/py314/1403-inline-syntax-improvements\|複数インライン文・構文の刷新]] | PEP 779 等 | 開発中 / 草案 |
| `1404` | [[versions/py314\|3.14]] | `library` | [[library/py314/1404-free-threading-jit-maturation\|フリースレッドとJITエンジンの第2段階強化]] | - | 開発中 |
| `1501` | [[versions/py315\|3.15]] | `language` | [[language/py315/1501-lazy-imports\|明示的遅延インポート（lazy imports: PEP 810）]] | PEP 810 | 確定 (最新安定版) |
| `1502` | [[versions/py315\|3.15]] | `language` | [[language/py315/1502-frozendict-and-sentinel\|不変辞書 frozendict と sentinel 組み込み型]] | PEP 814, PEP 661 | 確定 (最新安定版) |
| `1503` | [[versions/py315\|3.15]] | `language` | [[language/py315/1503-unpacking-in-comprehensions\|内包表記・ジェネレータでのアンパック構文]] | PEP 798 | 確定 (最新安定版) |
| `1504` | [[versions/py315\|3.15]] | `typing` | [[typing/py315/1504-typeform-and-typeddict-extra\|型システム進化 (TypeForm, TypedDict extra_items)]] | PEP 747, PEP 728 | 確定 (最新安定版) |

---

## 4. バージョン別ライフサイクル（EOLタイムライン早見表）

実務における動作保証・最低バージョンの選定基準となるライフサイクル一覧です。

| バージョン | リリース年月 | セキュリティ終了 (EOL) | ステータス（2026年10月時点） | 主な移行留意点 |
| :--- | :--- | :--- | :--- | :--- |
| **3.5** | 2015-09 | 2020-09 | **EOL (サポート終了)** | async/await, typing の原型導入 |
| **3.6** | 2016-12 | 2021-12 | **EOL (サポート終了)** | f-strings, 辞書の挿入順保持 |
| **3.7** | 2018-06 | 2023-06 | **EOL (サポート終了)** | dataclasses, `from __future__ import annotations` |
| **3.8** | 2019-10 | 2024-10 | **EOL (サポート終了)** | 代入式 (`:=`), positional-only 引数 (`/`) |
| **3.9** | 2020-10 | 2025-10 | **EOL (サポート終了)** | 辞書マージ演算子 (`\|`), 標準コレクション型ヒント |
| **3.10** | 2021-10 | 2026-10 | **EOL寸前 / 最終セキュリティ** | 構造的パターンマッチング (`match`/`case`), `X \| Y` |
| **3.11** | 2022-10 | 2027-10 | **セキュリティ修正保守** | ExceptionGroup, `except*`, tomllib, Faster CPython |
| **3.12** | 2023-10 | 2028-10 | **セキュリティ修正保守** | 型パラメータ構文 (`type`), distutils完全削除 |
| **3.13** | 2024-10 | 2029-10 | **バグ修正サポート** | フリースレッド (GIL無効化), JIT (Copy-and-Patch), PyRepl |
| **3.14** | 2025-10 | 2030-10 | **現行安定版** | t-strings (PEP 750), 遅延型注釈評価 (PEP 649) |
| **3.15** | 2026-10 | 2031-10 | **最新安定版** | lazy imports (PEP 810), frozendict (PEP 814), sentinel |

---

## 5. 目的別・横断ラーニングパス

バージョン順の通読だけでなく、関心のあるテーマごとに追跡・学習できる体系的ルートです。

### パス A: 型システム・静的型付けの深化（3.5 〜 3.15）
1. [[typing/py35/102-type-hints-basics|102-type-hints-basics]]: 型ヒントの標準化と `typing` の創世
2. [[typing/py36/202-variable-annotations|202-variable-annotations]]: 変数アノテーション構文
3. [[typing/py38/403-typing-extensions-standardized|403-typing-extensions-standardized]]: 構造的部分型 `Protocol` と `Literal`
4. [[typing/py39/502-builtin-generic-types|502-builtin-generic-types]]: 標準組み込みジェネリクス (`list[int]`)
5. [[typing/py310/602-union-operator|602-union-operator]]: 型ユニオン演算子 (`int | str`)
6. [[typing/py311/703-self-type-and-typevartuple|703-self-type-and-typevartuple]]: `Self` 型と可変長引数 `TypeVarTuple`
7. [[language/py312/801-type-parameter-syntax|801-type-parameter-syntax]]: `type` 文による第一級型パラメータ構文
8. [[typing/py313/903-typeis-and-typevar-defaults|903-typeis-and-typevar-defaults]]: 型の狭小化 `TypeIs` とデフォルト型引数
9. [[typing/py314/1402-deferred-evaluation-of-annotations|1402-deferred-evaluation-of-annotations]]: PEP 649/749 による注釈の遅延評価
10. [[typing/py315/1504-typeform-and-typeddict-extra|1504-typeform-and-typeddict-extra]]: `TypeForm` と閉じた `TypedDict(closed=True)`

### パス B: 非同期処理・並行実行モデルの進化（3.5 〜 3.15）
1. [[language/py35/101-async-await|101-async-await]]: ネイティブコルーチン構文 (`async def` / `await`)
2. [[language/py36/203-async-generators-comprehensions|203-async-generators-comprehensions]]: 非同期ジェネレータ・非同期内包表記
3. [[library/py38/404-asyncio-robustness|404-asyncio-robustness]]: `asyncio` の堅牢化と非同期タスク管理
4. [[language/py311/701-exception-groups|701-exception-groups]]: `TaskGroup` と `except*` 構文
5. [[language/py312/804-per-interpreter-gil|804-per-interpreter-gil]]: サブインタプリタごとのGIL分離
6. [[language/py313/901-free-threaded-cpython|901-free-threaded-cpython]]: フリースレッドCPython（GIL無効化モードの始動）
7. [[library/py314/1404-free-threading-jit-maturation|1404-free-threading-jit-maturation]]: フリースレッドエコシステムの成熟とGIL撤廃移行

### パス C: 実行パフォーマンスとランタイム高速化（3.11 〜 3.15）
1. [[language/py311/704-faster-cpython-interpreter|704-faster-cpython-interpreter]]: 特化適応型インタプリタ（Faster CPython 第1期）
2. [[language/py313/902-experimental-jit|902-experimental-jit]]: Copy-and-Patch 型JITコンパイラの実験導入
3. [[language/py315/1501-lazy-imports|1501-lazy-imports]]: 明示的遅延インポート（起動オーバーヘッドの極小化）

---

## 6. 主な破壊的変更・廃止年表（Breaking Changes）

| バージョン | 削除・変更された主要機能 | 移行先・推奨対応 |
| :--- | :--- | :--- |
| **3.7** | `async` / `await` が予約語化 | 同名の変数や引数名をリネーム |
| **3.8** | `@asyncio.coroutine` や `yield from` の非推奨化 | `async def` / `await` 構文へ統一 |
| **3.10** | `collections` 直下の抽象基底クラス参照廃止 | `collections.abc.*` からインポート |
| **3.11** | `asyncio.get_event_loop()` の暗黙生成禁止 | `asyncio.new_event_loop()` または `asyncio.run()` |
| **3.12** | レガシービルド基盤 **`distutils` の完全削除** | `setuptools` や `flit`, `hatch` への移行必須 |
| **3.13** | 旧 `cgi`, `telnetlib`, `crypt` 等のデッドモジュール一括削除 (PEP 594) | サードパーティ代替ライブラリへ移行 |
| **3.15** | 旧 `__future__.annotations` の仕様整理 | PEP 649/749 遅延評価ランタイムの標準適用 |

---

## 7. Python 3.16 の展望（開発プレビュー・検討事項）

2026年10月のPython 3.15リリースに伴い、次期バージョン **Python 3.16**（2027年10月リリース予定）の開発サイクルがスタートしています。現在コミュニティおよびSteering Councilで議論・検討されている主な方向性は以下の通りです。

1. **フリースレッド（GIL無効化）のデフォルト化ロードマップ（PEP 703 段階3）**:
   - 3.13（実験的オプトイン）→ 3.14/3.15（エコシステム対応）を経て、3.16以降で「GIL無効化ビルドを公式ディストリビューションの推奨／主要ターゲットへ格上げ」することが視野に入っています。
2. **JITコンパイラ（Tier 2 / Copy-and-Patch）の本格最適化**:
   - 3.13で導入されたJIT基盤上で、インライン展開やトレース最適化のさらなる高度化が進められています。
3. **明示的遅延インポート（PEP 810）のエコシステム標準化**:
   - 3.15で導入された `import foo from bar with lazy` 構文を活用した、CLIツールや大規模Webフレームワーク（Django, FastAPI等）における起動速度向上のベストプラクティス確立。

---

## 8. バージョン・ステータス基準

本Vaultでは以下の表記を厳格に適用しています：
- **【確定 (Stable / EOL / Security / Bugfix)】**: CPythonとして正式リリースされ、仕様・構文が固定されたバージョン（3.5〜3.13、および2026年10月に正式リリースされた **3.15**）。
- **【確定・実験的 (Experimental)】**: CPythonにマージされたが、ビルドフラグや実験的オプトインが必要な機能（例: 3.13の `--disable-gil`, JIT）。
- **【採択・開発中 (Accepted / In Development)】**: PEPとして承認され、公式実装が進んでいるが、リリース前の機能（例: 3.14の PEP 750, PEP 649）。
- **【草案・議論中 (Draft / Under Discussion)】**: コミュニティやSteering Councilで議論中の次世代仕様（PEP 779など）。
