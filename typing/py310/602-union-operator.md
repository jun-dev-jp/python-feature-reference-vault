---
title: "型ユニオン演算子（X | Y）"
aliases: [型ユニオン演算子, Union operator, PEP 604, int | str]
category: typing
python_version: "3.10"
peps: [604]
created: 2026-10-26
updated: 2026-10-26
tags: [py310, typing, syntax, union]
---

# 型ユニオン演算子（X | Y）

## 1. 概要と動機

Python 3.5以降、複数の型のいずれかを受け入れる引数を表現するには `typing.Union[int, str]` や `typing.Optional[str]`（実質 `Union[str, None]`）と書く必要がありました。
しかし、この記法は以下の点で不便でした。
- コードが長くなり可読性を損ねる。
- `isinstance(val, (int, str))` のような実行時型チェックと構文上の乖離があった。

PEP 604 は、ビット演算子のパイプシンボル `|` をオーバーロードし、型式として `int | str` や `str | None` と簡潔に記述できるようにしました。

---

## 2. 仕様と構文

- **構文**: `TypeA | TypeB | TypeC`
- **`types.UnionType`**: `int | str` を評価すると、`typing.Union` ではなく軽量な新しい組み込み型 `types.UnionType` のインスタンスが返されます。
- **実行時判定のサポート**:
  `isinstance()` や `issubclass()` の第2引数としても直接利用可能です。
  ```python
  isinstance(val, int | str)  # 従来の isinstance(val, (int, str)) と同等
  ```

---

## 3. 従来記法との比較

Python 3.9 以前の `typing.Union` / `Optional` と、Python 3.10 の型ユニオン演算子（`|`）の比較です。

```python
# --- 従来（Python 3.9以前）: typing モジュールからのインポートが必須 ---
from typing import Optional, Union

def legacy_func(val: Union[int, str], timeout: Optional[float] = None) -> Union[str, None]:
    # 実行時型チェックにはタプルが必要
    if isinstance(val, (int, str)):
        return str(val)
    return None

# --- Python 3.10+: 型ユニオン演算子 | ---
# typing のインポート不要、Optional[T] は T | None に統一、isinstance でも利用可能
def modern_func(val: int | str, timeout: float | None = None) -> str | None:
    # 実行時判定でもタプルの代わりに | を使用可能
    if isinstance(val, int | str):
        return str(val)
    return None

print(legacy_func(100))
print(modern_func(200))
```

---

## 4. 最小コード例

Python 3.10 以降で動作する型ユニオン演算子と `isinstance` の連携例です。

```python
def format_identifier(val: int | str | None) -> str:
    # isinstance でのパイプ演算子の直接利用
    if isinstance(val, int | str):
        return f"ID-{val}"
    return "ID-ANONYMOUS"

if __name__ == "__main__":
    print(format_identifier(101))
    print(format_identifier("ABC"))
    print(format_identifier(None))
    print("Type expression:", int | str)
```

### 実行出力
```text
ID-101
ID-ABC
ID-ANONYMOUS
Type expression: int | str
```

---

## 5. 実務上の指針と落とし穴

- **Python 3.7〜3.9環境への配慮**:
  Python 3.9以前の環境では `int | str` は `TypeError: unsupported operand type(s) for |: 'type' and 'type'` になります。
  ただし、型アノテーション内であれば `from __future__ import annotations` を付与することで Python 3.7+ でも構文エラーにならず記述可能です（ただし `isinstance(val, int | str)` のような実行時式は不可）。
  ```python
  # Python 3.9 以前では TypeError になる実行時式
  # isinstance(x, int | str)

  # Python 3.9 以前での等価コード
  isinstance(x, (int, str))
  ```
- **`Optional` の廃止傾向**:
  `Optional[T]` は実質的に `T | None` の糖衣構文であったため、現代のPythonコードスタイルでは `T | None` への移行が強く推奨されています。

---

## 6. 関連リンク

- [[versions/py310|Python 3.10 概要]]
- [[typing/py35/102-type-hints-basics|102-type-hints-basics]]
- [[typing/py39/502-builtin-generic-types|502-builtin-generic-types]]
- [[language/py312/801-type-parameter-syntax|801-type-parameter-syntax]]

---

## 7. 参考資料

- [PEP 604 – Allow writing union types as X | Y](https://peps.python.org/pep-0604/)
- [Python 3.10 What's New: PEP 604](https://docs.python.org/3/whatsnew/3.10.html#pep-604-new-type-union-operator)
