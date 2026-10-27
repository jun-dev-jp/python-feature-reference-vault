---
title: "構造的パターンマッチング（match / case）"
aliases: [パターンマッチング, match case, PEP 634, PEP 635, PEP 636]
category: language
python_version: "3.10"
peps: [634, 635, 636]
created: 2026-10-26
updated: 2026-10-26
tags: [py310, language, syntax, pattern-matching]
---

# 構造的パターンマッチング（match / case）

## 1. 概要と動機

複雑なネストされたデータ構造（JSON、AST、タプル、オブジェクトなど）を処理する際、従来の `if / elif / else` や `isinstance()` による判定は、条件式が肥大化し可読性が低下する要因でした。

PEP 634 / 635 / 636 は、他言語（Scala、Rust等）で強力な機能として知られる「構造的パターンマッチング」を導入しました。単なる値の分岐（switch文）にとどまらず、**データの形状（構造）の一致検証、要素のアンパック、ガード条件**をワンステップで記述できます。

---

## 2. 仕様と構文

- **基本構造**:
  ```python
  match subject:
      case pattern_1:
          ...
      case pattern_2 if condition:  # ガード節
          ...
      case _:                       # ワイルドカード（デフォルト）
          ...
  ```
- **主なパターン種別**:
  - リテラルパターン: `case 404:` や `case "admin":`
  - キャプチャパターン: 変数名に束縛 `case [x, y]:`
  - シーケンスパターン: リストやタプル `case [1, *rest]:`
  - マッピングパターン: 辞書の特定キー `case {"type": "event", "id": event_id}:`
  - クラスパターン: クラス型と属性 `case Point(x=0, y=y):`
  - ORパターン: パイプで結合 `case 200 | 201:`
  - ASパターン: サブパターンを変数に束縛 `case [x, y] as point:`

---

## 3. 従来記法との比較

Python 3.9 以前の `if / isinstance / dict.get` による複雑な分岐と、Python 3.10 の構造的パターンマッチングの比較です。

```python
# --- 従来（Python 3.9以前）: 型チェック、キー存在確認、値の取り出しが分散 ---
def handle_event_legacy(event):
    if isinstance(event, dict) and event.get("type") == "click":
        pos = event.get("position")
        if isinstance(pos, (list, tuple)) and len(pos) == 2:
            x, y = pos
            return f"Legacy: Clicked at ({x}, {y})"
    elif isinstance(event, dict) and event.get("type") == "quit":
        return "Legacy: Quit"
    return "Legacy: Unknown"

# --- Python 3.10+: 構造的パターンマッチング ---
# 形状の検証、特定キーの抽出、要素のアンパックを1ステップで宣言
def handle_event_modern(event):
    match event:
        case {"type": "click", "position": [x, y]}:
            return f"Modern: Clicked at ({x}, {y})"
        case {"type": "quit"}:
            return "Modern: Quit"
        case _:
            return "Modern: Unknown"

sample = {"type": "click", "position": [10, 20]}
print(handle_event_legacy(sample))
print(handle_event_modern(sample))
```

---

## 4. 最小コード例

Python 3.10 以降で動作する、コマンド解釈処理のパターンマッチ例です。

```python
from dataclasses import dataclass

@dataclass
class Command:
    action: str
    target: str

def execute_instruction(payload: object) -> str:
    match payload:
        case {"type": "quit"}:
            return "Terminating application."
        case {"type": "move", "direction": ("up" | "down" | "left" | "right") as d}:
            return f"Moving {d}."
        case Command(action="create", target=name):
            return f"Creating resource: {name}"
        case [first, *rest] if len(rest) > 0:
            return f"Batch processing: {first} and {len(rest)} others"
        case _:
            return "Unrecognized instruction."

if __name__ == "__main__":
    print(execute_instruction({"type": "move", "direction": "up"}))
    print(execute_instruction(Command(action="create", target="document.pdf")))
    print(execute_instruction(["cmd1", "cmd2", "cmd3"]))
```

### 実行出力
```text
Moving up.
Creating resource: document.pdf
Batch processing: cmd1 and 2 others
```

---

## 5. 実務上の指針と落とし穴

- **単純な値の比較における濫用**:
  単なる数値や文字列の完全一致分岐であれば、従来の辞書ディスパッチ（`handlers.get(key)()`）や単純な `if/elif` の方が軽量で直感的な場合があります。複雑なデータ形状の分解やアンパックを伴う場面で最も威力を発揮します。
- **定数マッチの落とし穴**:
  `STATUS_OK = 200` を定義して `case STATUS_OK:` と書くと、値の比較ではなく「値を変数 `STATUS_OK` にキャプチャ（再束縛）」してしまいます。定数と比較したい場合は `case status.OK:` のようにドット修飾された名前空間を使うか、ガード節を使用します。
  ```python
  # 誤り: NOT_FOUND 変数に任意の値がキャプチャされてしまう
  NOT_FOUND = 404
  # case NOT_FOUND: ...  # 常にマッチしてしまう！

  # 正しい対処: 列挙型やドットアクセスを使う
  from http import HTTPStatus
  # case HTTPStatus.NOT_FOUND: ...
  ```
- **マッピングパターンの部分一致**:
  辞書パターン `case {"id": x}:` は、対象の辞書に他のキーが含まれていてもマッチします（完全一致ではなく部分一致の仕様）。

---

## 6. 関連リンク

- [[versions/py310|Python 3.10 概要]]
- [[language/py38/401-walrus-operator|401-walrus-operator]]
- [[typing/py310/602-union-operator|602-union-operator]]

---

## 7. 参考資料

- [PEP 634 – Structural Pattern Matching: Specification](https://peps.python.org/pep-0634/)
- [PEP 636 – Structural Pattern Matching: Tutorial](https://peps.python.org/pep-0636/)
- [Python 3.10 What's New: Structural Pattern Matching](https://docs.python.org/3/whatsnew/3.10.html#pep-634-structural-pattern-matching)
