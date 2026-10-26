---
title: "変数アノテーション構文（PEP 526）"
aliases: [変数アノテーション, 変数型注釈, PEP 526]
category: typing
python_version: "3.6"
peps: [526]
created: 2026-10-26
updated: 2026-10-26
tags: [py36, typing, syntax, annotations]
---

# 変数アノテーション構文（PEP 526）

## 1. 概要と動機

Python 3.5 の PEP 484 で関数引数と戻り値のアノテーションが導入されましたが、ローカル変数やクラス変数・インスタンス変数の型を明示する公式な構文がありませんでした。そのため、当時は以下のように型コメント（Type Comments）を用いて型を記述する必要がありました。
```python
# Python 3.5 までの記法（型コメント）
primes = []  # type: list[int]
```
しかし、型コメントは冗長であり、シンタックスハイライトが効かず、静的解析ツールと文法パーサの間で扱いにくさがありました。

PEP 526 は、変数宣言に直接型注釈を付与できる構文 `variable: Type = value` を言語仕様として導入しました。

---

## 2. 仕様と構文

- **基本構文**: `name: Type` または `name: Type = value`
- **クラス属性アノテーション**: クラス定義内で変数アノテーションを行うと、`__annotations__` 辞書に登録される。
- **初期値なし宣言**: 値を代入せずに型だけを宣言可能（例: `header: str`）。ただし値が未代入のまま参照すると通常通り `NameError` となる。

```python
# 変数宣言およびクラス属性アノテーションの基本形
count: int = 0
items: list[str] = []

class UserProfile:
    user_id: int        # 型のみの宣言（__annotations__ に登録される）
    is_active: bool = True  # デフォルト値を伴う属性宣言
```

---

## 3. 最小コード例

Python 3.6 以降で動作する変数型アノテーションと `__annotations__` 属性の確認例です。

```python
from typing import ClassVar, List

class Account:
    # クラス変数とインスタンス変数のアノテーション
    bank_code: ClassVar[str] = "9901"
    account_id: str
    balance: float

    def __init__(self, account_id: str, initial_balance: float) -> None:
        self.account_id = account_id
        self.balance = initial_balance

if __name__ == "__main__":
    acc = Account("A-100", 25000.0)
    print("Account balance:", acc.balance)
    print("Class annotations:", Account.__annotations__)
```

### 実行出力
```text
Account balance: 25000.0
Class annotations: {'bank_code': typing.ClassVar[str], 'account_id': <class 'str'>, 'balance': <class 'float'>}
```

---

## 4. 実務上の指針と落とし穴

- **初期値なし宣言の参照エラー**:
  型アノテーションを記述しただけで初期値を代入していない変数を参照すると、`NameError` が発生します。型宣言は変数の生成（名前の束縛）を行いません。
  ```python
  # 落とし穴: 名前は束縛されない
  total: int
  # print(total)  # NameError: name 'total' is not defined

  # 正しい対処: 必ず初期値を代入するか代入後にアクセス
  total = 0
  print(total)  # 正常に 0 が出力される
  ```
- **dataclasses（Python 3.7+）への影響**:
  PEP 526 の `__annotations__` 辞書メカニズムは、Python 3.7 で導入された `dataclasses`（PEP 557）の基盤となりました。

---

## 5. 関連リンク

- [[versions/py36|Python 3.6 概要]]
- [[typing/py35/102-type-hints-basics|102-type-hints-basics]]
- [[library/py37/301-dataclasses|301-dataclasses]]
- [[language/py312/801-type-parameter-syntax|801-type-parameter-syntax]]

---

## 6. 参考資料

- [PEP 526 – Syntax for Variable Annotations](https://peps.python.org/pep-0526/)
- [Python 3.6 What's New: PEP 526](https://docs.python.org/3/whatsnew/3.6.html#pep-526-syntax-for-variable-annotations)
