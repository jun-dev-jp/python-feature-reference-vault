---
title: "型パラメータ構文（PEP 695: type文と新ジェネリクス）"
aliases: [型パラメータ構文, type文, PEP 695, 新型記法]
category: language
python_version: "3.12"
peps: [695]
created: 2026-10-26
updated: 2026-10-26
tags: [py312, language, typing, syntax, generics]
---

# 型パラメータ構文（PEP 695: type文と新ジェネリクス）

## 1. 概要と動機

Python 3.5〜3.11までのジェネリクス定義は、`T = TypeVar('T')` や `Generic[T]` を記述する必要があり、極めて不自然で冗長でした。また、型エイリアスを定義する際も単なる変数代入に見えるため、`TypeAlias` アノテーション（PEP 613）を追加するなどパッチ当てが繰り返されてきました。

PEP 695 は、Pythonの文法そのものを拡張し、モダンな言語（TypeScriptやRust等）と同様の角括弧による型パラメータ構文 `[T]` と、専用の `type` 文を導入しました。

---

## 2. 仕様と構文

- **型エイリアス定義 (`type` 文)**:
  ```python
  type Point = tuple[float, float]
  type CoordinateList[T] = list[tuple[T, T]]
  ```
- **ジェネリック関数**:
  `TypeVar` を事前に作らずに関数定義で直接パラメータ化。
  ```python
  def first[T](items: list[T]) -> T:
      return items[0]
  ```
- **ジェネリッククラス**:
  `Generic[T]` を継承せず直接クラス名に角括弧を付与。
  ```python
  class Box[T]:
      def __init__(self, value: T) -> None:
          self.value = value
  ```
- **スコープ付き型変数**:
  型パラメータは自動的に新しいスコープで作成され、大域的な `TypeVar` オブジェクトを汚染しません。

---

## 3. 最小コード例

Python 3.12 以降で動作する、PEP 695 構文によるジェネリックスタックと型エイリアスの例です。

```python
# TypeVar や Generic の import は一切不要
type NumberOrStr = int | float | str

class Stack[T]:
    def __init__(self) -> None:
        self._items: list[T] = []

    def push(self, item: T) -> None:
        self._items.append(item)

    def pop(self) -> T:
        return self._items.pop()

def duplicate_item[T](val: T) -> tuple[T, T]:
    return (val, val)

if __name__ == "__main__":
    stack: Stack[NumberOrStr] = Stack()
    stack.push("hello")
    stack.push(42)
    print("Popped:", stack.pop())
    print("Duplicate:", duplicate_item(3.14))
```

### 実行出力
```text
Popped: 42
Duplicate: (3.14, 3.14)
```

---

## 4. 実務上の指針と落とし穴

- **後方互換性（Python 3.11以下）**:
  PEP 695 はPythonのパーサ・文法レベルの変更であるため、Python 3.11以下の環境では `SyntaxError` となります。`from __future__ import annotations` を付けても回避できません。ライブラリ開発等で 3.11以前をサポートする場合は従来の `TypeVar` / `typing.TypeAlias` 記法を維持する必要があります。
  ```python
  # Python 3.11 以前との比較
  # レガシー: T = TypeVar('T'); def f(x: T) -> T: ...
  # Python 3.12+: def f[T](x: T) -> T: ...
  ```
- **遅延評価の恩恵**:
  `type` 文で定義された型エイリアスは遅延評価（Lazy Evaluation）されるため、定義時点で未定義のクラス名（前方参照）をクォートなしでそのまま記述できます。

---

## 5. 関連リンク

- [[versions/py312|Python 3.12 概要]]
- [[typing/py35/102-type-hints-basics|102-type-hints-basics]]
- [[typing/py39/502-builtin-generic-types|502-builtin-generic-types]]
- [[typing/py310/602-union-operator|602-union-operator]]
- [[typing/py314/1402-deferred-evaluation-of-annotations|1402-deferred-evaluation-of-annotations]]

---

## 6. 参考資料

- [PEP 695 – Type Parameter Syntax](https://peps.python.org/pep-0695/)
- [Python 3.12 What's New: Type Parameter Syntax](https://docs.python.org/3/whatsnew/3.12.html#pep-695-type-parameter-syntax)
