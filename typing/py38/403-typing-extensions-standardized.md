---
title: "型ヒント拡張の標準統合（Literal, Final, TypedDict, Protocol）"
aliases: [Protocol, TypedDict, Literal, Final, PEP 544, PEP 586, PEP 589, PEP 591]
category: typing
python_version: "3.8"
peps: [544, 586, 589, 591]
created: 2026-10-26
updated: 2026-10-26
tags: [py38, typing, static-analysis, protocol, typeddict]
---

# 型ヒント拡張の標準統合（Literal, Final, TypedDict, Protocol）

## 1. 概要と動機

Python 3.5〜3.7の型システムは、名目的サブタイピング（明示的なクラス継承）を中心としていたため、Pythonの代名詞である「ダックタイピング」や辞書の構造的型付けを表現できませんでした。
Python 3.8 では、以前はサードパーティの `typing_extensions` にあった4つの画期的な型機能が標準ライブラリ `typing` に統合されました。

---

## 2. 仕様と構文

1. **`Protocol`（PEP 544 - 構造的サブタイピング）**:
   明示的な継承なしで、特定のメソッドや属性を持つオブジェクトを型として受け入れる静的ダックタイピング。
2. **`TypedDict`（PEP 589 - 型付き辞書）**:
   各キーと値の型が決まっている辞書の構造を定義。
3. **`Literal`（PEP 586 - 固定リテラル値型）**:
   文字列や数値などの特定の値そのものを型として指定（例: `Literal["r", "w"]`）。
4. **`Final`（PEP 591 - 再代入・オーバーライド禁止）**:
   定数の再代入禁止や、クラス・メソッドのオーバーライド禁止を宣言。

```python
# 各構文の宣言例
TIMEOUT: Final[int] = 10                  # 定数保護
AccessMode = Literal["read", "write"]     # 特定リテラルのみ許容

class ItemDict(TypedDict):                 # 辞書スキーマ
    id: int
    name: str
```

---

## 3. 最小コード例

```python
from typing import Protocol, TypedDict, Literal, Final

# 1. Final
MAX_RETRIES: Final = 3

# 2. Literal
Mode = Literal["read", "write"]

# 3. TypedDict
class UserConfig(TypedDict):
    theme: str
    auto_save: bool

# 4. Protocol（静的ダックタイピング）
class Renderable(Protocol):
    def render(self) -> str: ...

class Component:
    def render(self) -> str:
        return "<div>Rendered</div>"

def display(item: Renderable) -> None:
    print("Output:", item.render())

if __name__ == "__main__":
    display(Component())  # ComponentはRenderableを継承していないが型安全！
```

### 実行出力
```text
Output: <div>Rendered</div>
```

---

## 4. 実務上の指針と落とし穴

- **`Protocol` の実行時判定**:
  `Protocol` クラスは標準では `isinstance()` で判定できません。実行時にもチェックしたい場合は `@typing.runtime_checkable` デコレータを付与します。
  ```python
  from typing import Protocol, runtime_checkable

  @runtime_checkable
  class Closable(Protocol):
      def close(self) -> None: ...

  # runtime_checkable があれば isinstance で判定可能
  print(isinstance(open("test.txt", "w"), Closable))  # True
  ```
- **後方互換性**:
  Python 3.7以前の環境をサポートする場合は `from typing_extensions import Protocol, TypedDict, Literal, Final` で対応可能です。

---

## 5. 関連リンク

- [[versions/py38|Python 3.8 概要]]
- [[typing/py35/102-type-hints-basics|102-type-hints-basics]]
- [[typing/py312/803-override-decorator|803-override-decorator]]
- [[typing/py315/1504-typeform-and-typeddict-extra|1504-typeform-and-typeddict-extra]]

---

## 6. 参考資料

- [PEP 544 – Protocols: Structural subtyping (static duck typing)](https://peps.python.org/pep-0544/)
- [PEP 586 – Literal Types](https://peps.python.org/pep-0586/)
- [PEP 589 – TypedDict: Type Hints for Dictionaries with a Fixed Schema](https://peps.python.org/pep-0589/)
- [PEP 591 – Adding a final qualifier to typing](https://peps.python.org/pep-0591/)
