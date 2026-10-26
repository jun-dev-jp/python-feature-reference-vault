---
title: "型絞り込みの厳格化（TypeIs: PEP 742）"
aliases: [TypeIs, 型絞り込み, PEP 742, TypeGuard改善]
category: typing
python_version: "3.13"
peps: [742]
created: 2026-10-26
updated: 2026-10-26
tags: [py313, typing, narrowing, type-safety]
---

# 型絞り込みの厳格化（TypeIs: PEP 742）

## 1. 概要と動機

Python 3.10で導入された `TypeGuard`（PEP 647）は、関数を利用して型を絞り込む仕組みを提供しました。しかし、`TypeGuard` には「**False分岐において型が絞り込まれない（狭小化が片方向のみ）**」「**基底型以外への不当な型の置き換えを許してしまう**」という型安全上の欠陥がありました。

PEP 742 は、真偽値の双方で直感的な双方向の型絞り込みを実現する新しいシンボル `typing.TypeIs` を導入しました。

---

## 2. 仕様と構文

- **構文**: `def is_str_list(val: list[object]) -> TypeIs[list[str]]: ...`
- **True分岐**: 引数の型を `TypeIs[T]` の `T` との共通部分（交差）へ絞り込む。
- **False分岐**: 引数の型から `T` を除外した差分型として正しく絞り込む。
- **制約**: `TypeIs[T]` の `T` は、対象引数の型のサブタイプまたは互換性のある型でなければならず、まったく無関係な型へのキャストは型チェッカによって拒否される。

```python
# TypeIs の双方向型絞り込みシグネチャ
def is_str(val: object) -> TypeIs[str]:
    return isinstance(val, str)
```

---

## 3. 最小コード例

```python
from typing import TypeIs

def is_int(val: object) -> TypeIs[int]:
    return isinstance(val, int)

def process_value(val: int | str) -> str:
    if is_int(val):
        # ここでは val は int
        return f"Integer doubled: {val * 2}"
    else:
        # TypeIs により、False分岐では自動的に str に絞り込まれる！
        return f"String upper: {val.upper()}"

if __name__ == "__main__":
    print(process_value(10))
    print(process_value("python"))
```

### 実行出力
```text
Integer doubled: 20
String upper: PYTHON
```

---

## 4. 実務上の指針と落とし穴

- **`TypeGuard` との使い分け**:
  - `TypeIs` は常に厳格な型の絞り込み（述語）に使用します。
  - `TypeGuard` は「任意の型から特定の狭い型への緩やかな安全装置（Narrowing）」として残されますが、大半のユースケースでは `TypeIs` が推奨されます。
- **Python 3.12以前での利用**:
  `typing_extensions >= 4.10.0` から `from typing_extensions import TypeIs` で利用可能です。
  ```python
  # False 分岐でも型が正しく排除される
  def example(x: int | str):
      if is_str(x):
          pass  # x は str
      else:
          pass  # x は確実に int に絞り込まれる！
  ```

---

## 5. 関連リンク

- [[versions/py313|Python 3.13 概要]]
- [[typing/py310/604-typeguard-and-typealias|604-typeguard-and-typealias]]
- [[language/py312/801-type-parameter-syntax|801-type-parameter-syntax]]

---

## 6. 参考資料

- [PEP 742 – Narrowing types with TypeIs](https://peps.python.org/pep-0742/)
- [Python 3.13 What's New: PEP 742](https://docs.python.org/3/whatsnew/3.13.html#pep-742-narrowing-types-with-typeis)
