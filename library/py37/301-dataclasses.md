---
title: "データクラス（dataclasses）の導入"
aliases: [dataclasses, データクラス, PEP 557]
category: library
python_version: "3.7"
peps: [557]
created: 2026-10-26
updated: 2026-10-26
tags: [py37, library, oop, boilerplate]
---

# データクラス（dataclasses）の導入

## 1. 概要と動機

Pythonでデータを保持するためのクラスを定義する際、従来のクラスでは `__init__`, `__repr__`, `__eq__`, `__hash__` などの特殊メソッドを手作業で記述する必要があり、極めて冗長でした。`collections.namedtuple` も存在しましたが、タプルとしてのアンパックやインデックスアクセスが意図しない挙動を生むことや、デフォルト値・型の定義が限定的であるという問題がありました。

PEP 557 は、デコレータ `@dataclass` を付与するだけで、クラス変数型アノテーションを解析し、これらの特殊メソッドを自動生成する `dataclasses` モジュールを導入しました。

---

## 2. 仕様と構文

- **`@dataclass` デコレータ**:
  - `init=True`: `__init__` を自動生成。
  - `repr=True`: 文字列列表現 `__repr__` を自動生成。
  - `eq=True`: 同値性比較 `__eq__` を自動生成。
  - `order=False`: `<, <=, >, >=` 等の順序比較メソッドを生成。
  - `frozen=False`: `True` に設定するとイミュータブル（変更不可）化し、自動で `__hash__` も生成される。
- **`field()` 関数**:
  デフォルトファクトリ（`default_factory=list` 等）や、比較・表現対象から除外する設定を行う。

```python
# データクラス定義の基本形
@dataclass(frozen=True)
class Point:
    x: float
    y: float
    tags: list[str] = field(default_factory=list)  # 可変デフォルト値は factory 指定
```

---

## 3. 最小コード例

Python 3.7 以降で動作する基本的な `@dataclass` の利用例です。

```python
from dataclasses import dataclass, field
from typing import List

@dataclass(frozen=True)
class Item:
    name: str
    price: float
    tags: List[str] = field(default_factory=list)

if __name__ == "__main__":
    item1 = Item(name="Notebook", price=1200.0, tags=["stationery", "office"])
    item2 = Item(name="Notebook", price=1200.0, tags=["stationery", "office"])

    print(item1)
    print("Equal:", item1 == item2)

    # frozen=True なので属性代入は FrozenInstanceError
    try:
        item1.price = 900.0  # type: ignore
    except Exception as e:
        print("Mutation failed as expected:", type(e).__name__)
```

### 実行出力
```text
Item(name='Notebook', price=1200.0, tags=['stationery', 'office'])
Equal: True
Mutation failed as expected: FrozenInstanceError
```

---

## 4. 実務上の指針と落とし穴

- **ミュータブルなデフォルト値の禁止**:
  `tags: List[str] = []` と書くと `ValueError: mutable default <class 'list'> for field tags is not allowed: use default_factory` が発生します。必ず `field(default_factory=list)` を使用してください。
  ```python
  # 誤り: 全インスタンスでリストが共有されるのを防ぐため例外になる
  # tags: list[str] = []

  # 正しい対処: default_factory で毎回新しいリストを生成
  tags: list[str] = field(default_factory=list)
  ```
- **実行時の型検証は行われない**:
  データクラスは型アノテーションに基づいてメソッドを生成しますが、実行時に `price="invalid"` を渡しても自動検証はされません。実行時バリデーションが必要な場合は Pydantic 等を検討するか、`__post_init__` 内で検証します。
- **スロット（Slots）対応（Python 3.10+）**:
  Python 3.10 以降では `@dataclass(slots=True)` を指定することで、メモリ使用量の削減と属性アクセスの高速化が可能です。

---

## 5. 関連リンク

- [[versions/py37|Python 3.7 概要]]
- [[typing/py36/202-variable-annotations|202-variable-annotations]]
- [[typing/py38/403-typing-extensions-standardized|403-typing-extensions-standardized]]

---

## 6. 参考資料

- [PEP 557 – Data Classes](https://peps.python.org/pep-0557/)
- [Python 3.7 What's New: PEP 557](https://docs.python.org/3/whatsnew/3.7.html#pep-557-data-classes)
