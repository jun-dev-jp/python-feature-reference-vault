---
title: "型アノテーション遅延評価（PEP 563: __future__.annotations）"
aliases: [__future__.annotations, PEP 563, 注釈文字列化]
category: typing
python_version: "3.7"
peps: [563]
created: 2026-10-26
updated: 2026-10-26
tags: [py37, typing, forward-reference, metaprogramming]
---

# 型アノテーション遅延評価（PEP 563: __future__.annotations）

## 1. 概要と動機

Python 3.5/3.6 において、クラス自身を参照する型（再帰的型参照）や、同ファイル内でまだ定義されていないクラス（前方参照）を型ヒントとして記述する場合、未定義エラーを避けるために `'Node'` のように文字列でクォートする必要がありました。また、モジュールインポート時にすべての型式が即時評価されるため、起動時間の遅延原因にもなっていました。

PEP 563 は、`from __future__ import annotations` を宣言することで、すべての型アノテーションを実行時に評価せず、コンパイラが静的に文字列として `__annotations__` 辞書に格納する仕組みを提供しました。

---

## 2. 仕様と構文

- **宣言**: ファイルの最上部に `from __future__ import annotations` を記述。
- **効果**:
  - 関数やクラスの型注釈がコンパイル時に文字列化される。
  - 前方参照や再帰参照をクォートなしで直接記述可能。
  - Python 3.9 で導入された `list[int]` 構文などを、Python 3.7 / 3.8 上でも実行時エラーを出さずに記述可能。

```python
# ファイル冒頭でインポートして型注釈の文字列化を有効化
from __future__ import annotations

# 未定義クラスでもクォートなしで型アノテーション可能
def process(node: FutureClass) -> None:
    pass
```

---

## 3. 最小コード例

```python
from __future__ import annotations

class TreeNode:
    def __init__(self, value: int, parent: TreeNode | None = None) -> None:
        self.value = value
        # 自身のクラス定義中でもクォートなしで TreeNode を注釈可能
        self.parent: TreeNode | None = parent

if __name__ == "__main__":
    node = TreeNode(10)
    print("Annotations raw repr:", TreeNode.__init__.__annotations__)
```

### 実行出力
```text
Annotations raw repr: {'value': 'int', 'parent': 'TreeNode | None', 'return': 'None'}
```

---

## 4. 実務上の指針と落とし穴

- **実行時型解析ライブラリの課題**:
  Pydantic や FastAPI など、実行時に `get_type_hints()` を使って型を解釈するライブラリにおいて、文字列化された型を `eval()` するコストやスコープ問題が生じる要因となりました。
  ```python
  # 実行時には文字列として保存される
  from __future__ import annotations
  x: int = 1
  print(__annotations__["x"])  # 'int'（型オブジェクトではなく str）
  ```
- **Python 3.14（PEP 649 / 749）への道筋**:
  当初 PEP 563 は将来のデフォルトになる予定でしたが、実行時エコシステムの破壊を懸念してデフォルト化が撤回され、Python 3.14 で「アクセス時オンデマンド評価（PEP 649）」が採択されることになりました。

---

## 5. 関連リンク

- [[versions/py37|Python 3.7 概要]]
- [[typing/py35/102-type-hints-basics|102-type-hints-basics]]
- [[typing/py314/1402-deferred-evaluation-of-annotations|1402-deferred-evaluation-of-annotations]]

---

## 6. 参考資料

- [PEP 563 – Postponed Evaluation of Annotations](https://peps.python.org/pep-0563/)
- [Python 3.7 What's New: PEP 563](https://docs.python.org/3/whatsnew/3.7.html#pep-563-postponed-evaluation-of-annotations)
