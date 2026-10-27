---
title: "明示的遅延インポート（lazy imports: PEP 810）"
aliases: [lazy imports, 遅延インポート, PEP 810, lazyキーワード]
category: language
python_version: "3.15"
peps: [810]
created: 2026-10-26
updated: 2026-10-26
tags: [py315, language, syntax, import, startup, performance]
---

# 明示的遅延インポート（lazy imports: PEP 810）

## 1. 概要と動機

大規模なPythonアプリケーションやCLIツールでは、起動時に多数のモジュールを読み込むことで起動時間（Startup Time）が肥大化することが大きな課題でした。従来は関数内でローカルインポートを行う、あるいは `importlib` で自作プロキシを作成する等の対症療法が取られていましたが、コードの散乱や可読性の低下を招いていました。

Python 3.15 は **PEP 810** により、新しいソフトキーワード `lazy` を導入しました。ファイル先頭で `lazy import` と明示的に記述することで、モジュールの実際のロードとバイトコード実行を「**その名前が最初にアクセスされた瞬間**」まで遅延させることが可能になりました。

---

## 2. 仕様と構文

- **構文**:
  ```python
  lazy import json
  lazy from pathlib import Path
  ```
- **動作モデル**:
  - `lazy import` 文の実行時にはモジュール本体はロードされず、軽量なプロキシオブジェクト（`types.LazyImportType`）が束縛される。
  - プロキシの属性参照や関数呼び出しが行われた瞬間に、透過的に完全ロード（Eager import）が実行される。
  - ロード失敗（`ModuleNotFoundError` 等）は、インポート文の位置ではなく**初回の属性アクセス時**に発生し、トレースバックにはインポート元とアクセス元の双方が表示される。
- **制約**:
  - モジュールスコープでのみ記述可能（関数内、クラス内、`try` ブロック内は `SyntaxError`）。
  - スターインポート（`lazy from m import *`）および `__future__` インポートでは使用不可。
- **後方互換対応**:
  - 3.15未満と共存する場合は `__lazy_modules__ = ["json", "pathlib"]` をモジュールに定義することで同様の遅延動作が可能。

---

## 3. 従来手法との比較

Python 3.14 以前の遅延読み込みハック（関数内インポートや importlib）と、Python 3.15 の `lazy import`（PEP 810）の比較です。

```python
# --- 従来（Python 3.14以前）: 関数内インポートによる対症療法 ---
# モジュール先頭で依存関係が一覧できず、関数が呼ばれるたびに import 文のオーバーヘッドが発生
def process_data_legacy(payload: str):
    import json  # 関数ローカルで遅延ロード
    return json.loads(payload)

# --- Python 3.15+: lazy import (PEP 810) ---
# ファイル先頭で宣言的に記述でき、PEP 8 の「インポートは先頭にまとめる」原則を遵守。
# 実際に json の属性にアクセスするまでロードが遅延される。
lazy import json

def process_data_modern(payload: str):
    return json.loads(payload)

print(process_data_legacy('{"mode": "legacy"}'))
print(process_data_modern('{"mode": "modern"}'))
```

---

## 4. 最小コード例

```python
# Python 3.15 ネイティブ遅延インポート構文
lazy import json
lazy from pathlib import Path

def run():
    print("Application startup finished instantly!")
    # ここで初めて json がロードされる
    data = json.loads('{"version": "3.15"}')
    print("Parsed data:", data)

if __name__ == "__main__":
    run()
```

### 実行出力
```text
Application startup finished instantly!
Parsed data: {'version': '3.15'}
```

---

## 5. 実務上の指針と落とし穴

- **グローバル設定**:
  コマンドラインオプション `-X lazy_imports=all` または環境変数 `PYTHON_LAZY_IMPORTS=all` を指定することで、既存コードを書き換えずに全インポートを遅延化できます。
- **副作用を持つモジュールの注意**:
  インポート時にグローバルな登録処理（デコレータ登録やプラグイン初期化）を実行するモジュールは、遅延ロードによって実行順序が変わり予期せぬ挙動になる可能性があるため、Eagerインポートのままにするか `sys.set_lazy_imports_filter()` で除外設定を行います。
  ```python
  # 副作用を伴うモジュールは通常の import を使う
  import plugin_registration  # インポート時のフック登録を即時実行
  lazy import heavy_computation  # 重い計算モジュールのみ遅延
  ```

---

## 6. 関連リンク

- [[versions/py315|Python 3.15 概要]]
- [[language/py315/1502-frozendict-and-sentinel|1502-frozendict-and-sentinel]]
- [[language/py315/1503-unpacking-in-comprehensions|1503-unpacking-in-comprehensions]]

---

## 7. 参考資料

- [PEP 810 – Explicit Lazy Imports](https://peps.python.org/pep-0810/)
- [What's New In Python 3.15: PEP 810](https://docs.python.org/3.15/whatsnew/3.15.html#pep-810-explicit-lazy-imports)
