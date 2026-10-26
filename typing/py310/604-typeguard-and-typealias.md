---
title: "ユーザー定義型絞り込み（TypeGuard）と明示的型エイリアス（TypeAlias）"
aliases: [TypeGuard, TypeAlias, PEP 647, PEP 613]
category: typing
python_version: "3.10"
peps: [647, 613]
created: 2026-10-26
updated: 2026-10-26
tags: [py310, typing, typeguard, typealias]
---

# ユーザー定義型絞り込み（TypeGuard）と明示的型エイリアス（TypeAlias）

## 1. 概要と動機

1. **`TypeAlias`（PEP 613）**:
   従来の型エイリアスは `Point = tuple[float, float]` のように通常の変数代入と構文が同一であったため、静的型チェッカが「グローバル変数の宣言」なのか「新しい型エイリアスの定義」なのか判定に苦慮する場合がありました。
2. **`TypeGuard`（PEP 647）**:
   ユーザーが定義した述語関数（`is_str_list(val)` 等）を使って `if` 文で判定した際、型チェッカは関数の戻り値が `bool` であることしか分からず、ブロック内部で引数の型を絞り込むことができませんでした。

PEP 613 と PEP 647 は、これらの静的解析のギャップを埋める標準構文を導入しました。

---

## 2. 仕様と構文

- **`TypeAlias`**:
  `Coordinate: TypeAlias = tuple[int, int]` のように型注釈として付与（Python 3.12の `type` 文の前身）。
- **`TypeGuard[T]`**:
  関数の戻り値型として指定。その関数が `True` を返した場合、検証対象の引数の型が `T` であると型チェッカに通知。

```python
# TypeAlias と TypeGuard の宣言構文
UserID: TypeAlias = int | str

def is_string(val: object) -> TypeGuard[str]:
    return isinstance(val, str)
```

---

## 3. 最小コード例

```python
from typing import TypeGuard, Any

def is_str_list(val: list[Any]) -> TypeGuard[list[str]]:
    return all(isinstance(x, str) for x in val)

def process_items(items: list[Any]):
    if is_str_list(items):
        # ここでは items は list[str] として静的に安全に扱える
        print("Joined:", ", ".join(items))
    else:
        print("Mixed list detected")

if __name__ == "__main__":
    process_items(["alpha", "beta", "gamma"])
    process_items([1, 2, 3])
```

### 実行出力
```text
Joined: alpha, beta, gamma
Mixed list detected
```

---

## 4. 実務上の指針と落とし穴

- **`TypeGuard` の片方向性の欠陥**:
  `TypeGuard` は `False` 分岐で型を除外（絞り込み）できない制限がありました。この欠点は Python 3.13 で導入された `TypeIs`（PEP 742）によって解決されています。
  ```python
  # TypeGuard は False のときに元の型から絞り込まない
  def check(val: int | str):
      if is_str(val):
          reveal_type(val)  # str
      else:
          reveal_type(val)  # int | str（int に絞り込まれない欠点がある）
  ```
- **`TypeAlias` から `type` 文への移行**:
  Python 3.12 以降では、より直感的で強力な `type Alias = ...` 構文（PEP 695）への移行が推奨されます。

---

## 5. 関連リンク

- [[versions/py310|Python 3.10 概要]]
- [[language/py312/801-type-parameter-syntax|801-type-parameter-syntax]]
- [[typing/py313/903-typeis-and-typevar-defaults|903-typeis-and-typevar-defaults]]

---

## 6. 参考資料

- [PEP 647 – User-Defined Type Guards](https://peps.python.org/pep-0647/)
- [PEP 613 – Explicit Type Aliases](https://peps.python.org/pep-0613/)
