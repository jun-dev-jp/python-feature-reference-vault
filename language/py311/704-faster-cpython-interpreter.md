---
title: "高速CPython（Faster CPython: 特化適応型インタプリタ）"
aliases: [Faster CPython, 特化適応型インタプリタ, インラインキャッシュ, PEP 659]
category: language
python_version: "3.11"
peps: [659]
created: 2026-10-26
updated: 2026-10-26
tags: [py311, language, performance, bytecode, interpreter]
---

# 高速CPython（Faster CPython: 特化適応型インタプリタ）

## 1. 概要と動機

CPythonは歴史的にシンプルで移植性の高いスタック型バイトコードインタプリタとして実装されていましたが、動的言語特有の型チェックや属性検索（`__dict__` の走査等）が実行ループ内で毎回繰り返され、実行速度の遅さが指摘されていました。

Guido van Rossum や Mark Shannon らが主導する「Faster CPython」プロジェクトの第1弾として、Python 3.11 では **特化適応型インタプリタ（Specializing Adaptive Interpreter: PEP 659）** が導入され、既存のPythonコードをそのまま実行するだけで平均10〜60%の高速化を達成しました。

---

## 2. 仕様と構文

- **コード変更不要**: 開発者が構文や書き方を変更する必要は一切ありません。
- **インラインキャッシュ（Inline Caching）**:
  - バイトコード実行時、頻繁に実行されるオペコード（属性アクセス `LOAD_ATTR`、加算 `BINARY_OP`、関数呼び出し等）を監視。
  - 特定の型（例: `int` 同士の加算、特定クラスの固定属性）に特化した高速命令（`LOAD_ATTR_INSTANCE_VALUE` など）へ実行中に自己書き換え。
  - 型が変化した場合は即座に汎用オペコードへフォールバック（Deoptimization）。

```python
# 特化の恩恵を最大化する単一型（Monomorphic）の関数設計
def compute(x: int, y: int) -> int:
    # 常に int 型同士で呼ばれれば特化命令が維持される
    return x * y + 42
```

---

## 3. 最小コード例

Python 3.11 の特化適応型バイトコードの観察例（`dis` モジュール）です。

```python
import dis

def calculate_area(width: int, height: int) -> int:
    return width * height

# 数千回実行してインタプリタに適応・特化させる
for _ in range(5000):
    calculate_area(10, 20)

if __name__ == "__main__":
    print("Bytecode instructions with adaptive specialization:")
    # adaptive=True で特化された命令を表示
    dis.dis(calculate_area, adaptive=True)
```

### 実行出力（Python 3.11 環境）
```text
Bytecode instructions with adaptive specialization:
  ...
  BINARY_OP_MULTIPLY_INT (特化された高速オペコード)
  ...
```

---

## 4. 実務上の指針と落とし穴

- **安定した型の維持**:
  関数の引数やオブジェクトの属性に単一の型（Monomorphic）を渡し続けることで、特化命令が維持され最高のパフォーマンスを発揮します。逆に、同じ変数に頻繁に異なる型の値を代入するポリモーフィックなコードは特化が解除されやすくなります。
  ```python
  # パフォーマンス低下要因: 引数の型が頻繁に変動する
  # compute(1, 2)      -> int に特化
  # compute("a", 2)    -> 特化解除（脱最適化）
  ```
- **後続バージョン（3.12, 3.13 JIT）への発展**:
  このバイトコード最適化基盤が、Python 3.12 のマイクロオペ（Tier 2最適化）や Python 3.13 の JITコンパイラへと接続されていきます。

---

## 5. 関連リンク

- [[versions/py311|Python 3.11 概要]]
- [[language/py313/902-experimental-jit|902-experimental-jit]]

---

## 6. 参考資料

- [PEP 659 – Specializing Adaptive Interpreter](https://peps.python.org/pep-0659/)
- [Python 3.11 What's New: Faster CPython](https://docs.python.org/3/whatsnew/3.11.html#faster-cpython)
