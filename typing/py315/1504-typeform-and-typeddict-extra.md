---
title: "型システム進化（TypeForm と TypedDict extra_items: PEP 747 / PEP 728）"
aliases: [TypeForm, TypedDict closed, PEP 747, PEP 728]
category: typing
python_version: "3.15"
peps: [747, 728]
created: 2026-10-26
updated: 2026-10-26
tags: [py315, typing, typeform, typeddict, metaprogramming]
---

# 型システム進化（TypeForm と TypedDict extra_items: PEP 747 / PEP 728）

## 1. 概要と動機

1. **`TypeForm`（PEP 747）**:
   関数が型引数そのものを値として受け取る場合（`cast(typ, val)` やシリアライザ関数など）、従来の `type[T]` では `int` や `str` 等のクラスしか表現できず、`int | str`（Union）や `Literal["a"]`、`list[int]` などの「型式（Type Form）」を正確に注釈できませんでした。
2. **`TypedDict` の拡張（PEP 728）**:
   `TypedDict` は従来、未定義のキーを禁止するかどうかの柔軟性が不足していました。PEP 728 は `closed=True`（未知のキーを厳格に拒否）および `extra_items=T`（未定義キーを特定型として許可）を導入しました。

---

## 2. 仕様と構文

- **`typing.TypeForm[T]`**:
  - `T` を記述する型式オブジェクトを表現。実行時は `TypeForm(x)` は単に `x` をそのまま返す。
  ```python
  from typing import TypeForm, Any

  def parse_data[T](target_type: TypeForm[T], data: Any) -> T:
      ...
  ```
- **`TypedDict` の `closed` と `extra_items`**:
  ```python
  from typing import TypedDict

  # 未定義キーの混入を静的・実行時に遮断
  class StrictUser(TypedDict, closed=True):
      name: str

  # 未定義キーの値をすべて float として許容
  class Metrics(TypedDict, extra_items=float):
      timestamp: int
  ```

---

## 3. 最小コード例

```python
from typing import TypeForm, TypedDict, Any

# PEP 728: extra_items による動的プロパティの型付け
class ServerMetrics(TypedDict, extra_items=float):
    server_id: str

# PEP 747: 任意の型式を引数に取るシリアライザ
def validate_as[T](target_form: TypeForm[T], raw_val: Any) -> T:
    print(f"Validating against {target_form}")
    return raw_val  # type: ignore

if __name__ == "__main__":
    metrics: ServerMetrics = {
        "server_id": "srv-01",
        "cpu_usage": 78.5,
        "mem_usage": 42.1,
    }
    print("Metrics:", metrics)

    # Union型式そのものを渡して型安全に扱う
    val = validate_as(int | str, "hello")
    print("Validated:", val)
```

### 実行出力
```text
Metrics: {'server_id': 'srv-01', 'cpu_usage': 78.5, 'mem_usage': 42.1}
Validating against int | str
Validated: hello
```

---

## 4. 実務上の指針と落とし穴

- **`type[T]` と `TypeForm[T]` の明確な使い分け**:
  - `type[T]`: `T` のクラスインスタンスを生成するファクトリ関数（`cls()` を呼ぶ場合など）
  - `TypeForm[T]`: `Union`, `Literal`, `Annotated`, `TypedDict` などインスタンス化を前提としない抽象的な型式そのものを渡す場合
  ```python
  # 型式を値として受け取る場合のシグネチャ設計
  def check_type[T](form: TypeForm[T], value: object) -> bool:
      # form に int | str 等の Union も安全に渡せる
      return True
  ```

---

## 5. 関連リンク

- [[versions/py315|Python 3.15 概要]]
- [[typing/py38/403-typing-extensions-standardized|403-typing-extensions-standardized]]
- [[language/py312/801-type-parameter-syntax|801-type-parameter-syntax]]

---

## 6. 参考資料

- [PEP 747 – Annotating Type Forms](https://peps.python.org/pep-0747/)
- [PEP 728 – TypedDict with Typed Extra Items](https://peps.python.org/pep-0728/)
- [What's New In Python 3.15: New Typing Features](https://docs.python.org/3.15/whatsnew/3.15.html#whatsnew315-typeform)
