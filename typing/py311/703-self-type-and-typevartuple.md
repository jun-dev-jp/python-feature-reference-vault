---
title: "Self型と可変長ジェネリクス（PEP 673 / PEP 646）"
aliases: [Self型, TypeVarTuple, PEP 673, PEP 646]
category: typing
python_version: "3.11"
peps: [673, 646]
created: 2026-10-26
updated: 2026-10-26
tags: [py311, typing, self, generics, tensor]
---

# Self型と可変長ジェネリクス（PEP 673 / PEP 646）

## 1. 概要と動機

1. **`Self` 型（PEP 673）**:
   フルエントインターフェース（メソッドチェーン）やコピーメソッドにおいて、サブクラスで呼び出された際に「その派生クラス自身のインスタンス」を戻り値として型付けするには、`TypeVar` を使った複雑なバインド記法が必要でした。
2. **`TypeVarTuple`（PEP 646）**:
   NumPy配列やPyTorchテンソルなどの多次元配列のシェイプ（Shape）を型付けする際、次元数が可変であるタプルを表現するジェネリクスが存在しませんでした。

Python 3.11 はこれらの課題を解決する2大型機能を導入しました。

---

## 2. 仕様と構文

- **`typing.Self`**:
  メソッドの戻り値型に `-> Self` と書くだけで、実行時の呼び出し元クラス（派生クラス含む）の型を正確に表現。
- **`typing.TypeVarTuple`**:
  任意の数の型パラメータをひとまとめにアンパック（`*Ts`）して受け取れる可変長型変数。

```python
# Self 型によるメソッドチェーン定義
class Builder:
    def set_name(self, name: str) -> Self:
        self.name = name
        return self
```

---

## 3. 最小コード例

```python
from typing import Self

class QueryBuilder:
    def __init__(self) -> None:
        self.clauses: list[str] = []

    def select(self, field: str) -> Self:
        self.clauses.append(f"SELECT {field}")
        return self

    def where(self, condition: str) -> Self:
        self.clauses.append(f"WHERE {condition}")
        return self

class AdvancedQueryBuilder(QueryBuilder):
    def optimize(self) -> Self:
        self.clauses.append("OPTIMIZE")
        return self

if __name__ == "__main__":
    builder = AdvancedQueryBuilder()
    # メソッドチェーン後も正確に AdvancedQueryBuilder 型が維持される
    query = builder.select("id").where("active = 1").optimize()
    print("Clauses:", query.clauses)
```

### 実行出力
```text
Clauses: ['SELECT id', 'WHERE active = 1', 'OPTIMIZE']
```

---

## 4. 実務上の指針と落とし穴

- **`Self` の簡潔さ**:
  `T = TypeVar('T', bound='Base')` のような定型句を完全に排除できるため、ビルダーパターンやクラスファクトリで積極的に活用してください。
- **後方互換性**:
  Python 3.10以前では `from typing_extensions import Self, TypeVarTuple` を使用します。
  ```python
  # クラスメソッドで新しいインスタンスを返す際にも Self が有用
  @classmethod
  def create_default(cls) -> Self:
      return cls()
  ```

---

## 5. 関連リンク

- [[versions/py311|Python 3.11 概要]]
- [[language/py312/801-type-parameter-syntax|801-type-parameter-syntax]]

---

## 6. 参考資料

- [PEP 673 – Self Type](https://peps.python.org/pep-0673/)
- [PEP 646 – Variadic Generics](https://peps.python.org/pep-0646/)
