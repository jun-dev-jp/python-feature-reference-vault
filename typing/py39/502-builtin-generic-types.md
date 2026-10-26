---
title: "組み込みコレクションの型アノテーションジェネリクス化"
aliases: [組み込みジェネリクス, PEP 585, list[int], dict[str, int]]
category: typing
python_version: "3.9"
peps: [585]
created: 2026-10-26
updated: 2026-10-26
tags: [py39, typing, generics, standard-collections]
---

# 組み込みコレクションの型アノテーションジェネリクス化

## 1. 概要と動機

Python 3.5で導入されたPEP 484では、ジェネリックな型ヒントを記述するために `typing.List`, `typing.Dict`, `typing.Set`, `typing.Tuple` などの専用ラッパー型をインポートする必要がありました。
これにより、以下の二重管理と混乱が生じていました。
- 実行時の型は `list` なのに、型ヒントのためだけに `from typing import List` を書かされる冗長性。
- 初学者が `list[int]` と書いてしまい、`TypeError: 'type' object is not subscriptable` で落ちるトラブル。

PEP 585 は、標準の組み込み型（`list`, `dict`, `set`, `tuple` など）および標準コレクションクラスに対して `__class_getitem__` メソッドを実装し、直接 `list[int]` のように型引数を渡せるようにしました。

---

## 2. 仕様と構文

Python 3.9 以降、以下の標準型が直接添字（Subscript）による型パラメータ指定に対応しました。
- 組み込みコンテナ: `list[T]`, `dict[K, V]`, `set[T]`, `frozenset[T]`, `tuple[T, ...]`
- コレクション抽象基底クラス: `collections.abc.Iterable[T]`, `collections.abc.Sequence[T]`, `collections.abc.Mapping[K, V]`
- その他標準モジュール: `contextlib.AbstractContextManager[T]`, `re.Pattern[str]` 等

```python
# typing モジュールなしでの組み込み型アノテーション
records: dict[str, list[int]] = {"scores": [10, 20]}
items: tuple[int, ...] = (1, 2, 3)
```

---

## 3. 最小コード例

Python 3.9 以降で `typing` からのコンテナインポートなしで動作する型注釈の例です。

```python
# typing からの List, Dict インポートは不要！
def process_registry(
    users: list[str],
    metadata: dict[str, list[int]]
) -> dict[str, tuple[int, int]]:
    result: dict[str, tuple[int, int]] = {}
    for user in users:
        scores = metadata.get(user, [])
        if scores:
            result[user] = (min(scores), max(scores))
    return result

if __name__ == "__main__":
    users = ["alice", "bob"]
    meta = {"alice": [85, 92, 78], "bob": [60, 95]}
    output = process_registry(users, meta)
    print(output)
```

### 実行出力
```text
{'alice': (78, 92), 'bob': (60, 95)}
```

---

## 4. 実務上の指針と落とし穴

- **Python 3.7 / 3.8 との後方互換性**:
  Python 3.7 および 3.8 で `list[int]` 構文を使用したい場合は、ファイルの先頭に `from __future__ import annotations` を記述することで、実行時に評価されず構文エラーを回避できます（静的型チェッカは解釈可能）。
- **`typing.List` 等の非推奨化**:
  PEP 585 により、`typing.List` や `typing.Dict` などのエイリアスは非推奨となり、将来のバージョンで段階的に削除される計画となっています（新しいコードでは必ず小文字の組み込み型を使用してください）。
- **具象型ではなく抽象基底クラスの推奨**:
  引数の型アノテーションには `list[str]` よりも `collections.abc.Sequence[str]` を用いることで、タプル等も受け入れ可能になり、コードの再利用性が高まります。
  ```python
  from collections.abc import Sequence

  # 引数には柔軟な Sequence を使用し、戻り値には具体的な list を指定
  def get_first_item(items: Sequence[str]) -> str:
      return items[0]
  ```

---

## 5. 関連リンク

- [[versions/py39|Python 3.9 概要]]
- [[typing/py35/102-type-hints-basics|102-type-hints-basics]]
- [[typing/py37/302-postponed-annotations|302-postponed-annotations]]
- [[typing/py310/602-union-operator|602-union-operator]]
- [[language/py312/801-type-parameter-syntax|801-type-parameter-syntax]]

---

## 6. 参考資料

- [PEP 585 – Type Hinting Generics In Standard Collections](https://peps.python.org/pep-0585/)
- [Python 3.9 What's New: PEP 585](https://docs.python.org/3/whatsnew/3.9.html#type-hinting-generics-in-standard-collections)
