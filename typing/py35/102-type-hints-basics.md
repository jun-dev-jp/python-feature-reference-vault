---
title: "型ヒント（Type Hints）基本システムの導入"
aliases: [型ヒント基礎, PEP 484, typingモジュール導入]
category: typing
python_version: "3.5"
peps: [484, 483]
created: 2026-10-26
updated: 2026-10-26
tags: [py35, typing, static-analysis]
---

# 型ヒント（Type Hints）基本システムの導入

## 1. 概要と動機

動的型付け言語であるPythonの大規模コードベースにおいて、リファクタリングの困難さ、IDE補完の不正確さ、実行時まで型エラーが検出されない問題が顕在化していました。

PEP 484 は、Python 3.0 で追加された関数アノテーション構文（PEP 3107）を基盤として、型チェッカ（mypy等）やIDEが静的解析を行うための統一的な型システムと標準ライブラリ `typing` を導入しました。

---

## 2. 仕様と構文

- **実行時の無影響**: 型アノテーションは実行時の動作やパフォーマンスに原則として影響を与えず、型強制（Type Coercion）も行われません。
- **標準型シンボル**: `typing` モジュールから `Union`, `Optional`, `List`, `Dict`, `Tuple`, `Callable`, `Any` などをインポートして利用。
- **型変数**: ジェネリクスを表現するために `TypeVar` を定義。

```python
from typing import List, Optional, TypeVar

T = TypeVar('T')

def first_or_default(items: List[T], default: Optional[T] = None) -> Optional[T]:
    return items[0] if items else default
```

---

## 3. 最小コード例

Python 3.5 の `typing` モジュールを用いた基本的な関数の型注釈例です。

```python
from typing import Dict, List, Optional

def summarize_scores(scores: List[int]) -> Dict[str, Optional[float]]:
    if not scores:
        return {"average": None, "max": None}
    return {
        "average": sum(scores) / len(scores),
        "max": float(max(scores)),
    }

if __name__ == "__main__":
    result = summarize_scores([80, 95, 70])
    print(result)
```

### 実行出力
```text
{'average': 81.66666666666667, 'max': 95.0}
```

---

## 4. 実務上の指針と落とし穴

- **組み込み型との区別（Python 3.5〜3.8）**:
  Python 3.8までは `list[int]` や `dict[str, int]` のように直接組み込み型へパラメータを渡すことはできず、必ず `from typing import List, Dict` が必要でした（PEP 585 により Python 3.9 で解消）。
- **前方参照（Forward Reference）の文字列クォート**:
  定義前のクラス名を自身のメソッド内で型注釈にする場合、`'MyClass'` のように文字列としてクォートする必要がありました（Python 3.7 の PEP 563 や 3.14 の PEP 649 で改善）。
- **実行時型チェックの誤解**:
  `summarize_scores(["a", "b"])` を渡しても Python インタプリタは例外を出さず実行を試みます。必ず CI や pre-commit で mypy / pyright などの外部静的解析ツールと組み合わせることが不可欠です。
  ```python
  # 型ヒントは実行時には検証されない
  def add(x: int, y: int) -> int:
      return x + y

  # 実行時エラーにならず文字列結合されてしまう
  result = add("foo", "bar")  # "foobar" が返る（型チェッカのみがエラー検出）
  ```

---

## 5. 関連リンク

- [[versions/py35|Python 3.5 概要]]
- [[typing/py36/202-variable-annotations|202-variable-annotations]]
- [[typing/py37/302-postponed-annotations|302-postponed-annotations]]
- [[typing/py39/502-builtin-generic-types|502-builtin-generic-types]]
- [[typing/py310/602-union-operator|602-union-operator]]

---

## 6. 参考資料

- [PEP 484 – Type Hints](https://peps.python.org/pep-0484/)
- [PEP 483 – The Theory of Type Hints](https://peps.python.org/pep-0483/)
