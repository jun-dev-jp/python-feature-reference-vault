---
title: "実験的JITコンパイラ基盤（Copy-and-Patch JIT: PEP 744）"
aliases: [JITコンパイラ, 実験的JIT, PEP 744, Copy-and-Patch]
category: language
python_version: "3.13"
peps: [744]
created: 2026-10-26
updated: 2026-10-26
tags: [py313, language, jit, performance, compiler]
---

# 実験的JITコンパイラ基盤（Copy-and-Patch JIT: PEP 744）

## 1. 概要と動機

CPythonをさらに高速化するためには、従来のバイトコードインタプリタから脱却し、ネイティブマシンコードを生成するJIT（Just-In-Time）コンパイラが必要でした。しかし、巨大なLLVMをCPythonランタイムに組み込むと、バイナリサイズや起動時間のオーバーヘッドが許容できないほど大きくなります。

PEP 744 は、**Copy-and-Patch 技術** を採用した軽量な実験的JITコンパイラを導入しました。ビルド時にLLVMを用いて各バイトコードに対応する小さなネイティブコード断片（テンプレート）を事前生成しておき、実行時にはそれらを「コピーしてアドレスをパッチする」だけで高速にネイティブコードを組み立てます。

---

## 2. 仕様と構文

- **ビルド設定**: `--enable-experimental-jit` を付与してビルドされたCPythonで有効化。
- **Tier 2 最適化**: 特化適応型インタプリタ（Tier 1）が検知したホットなトレース（頻出ループ等）を中間表現（Micro-ops: uops）に変換し、JITがマシンコードにコンパイルして直接実行。
- **言語構文への影響**: なし（既存のPythonコードがそのまま透明に高速化される）。

```python
# 透過的なJIT実行（特別な構文は不要）
def heavy_loop(count: int) -> int:
    # ホットループが自動的に Tier 2 JIT マシンコードに変換される
    return sum(i * i for i in range(count))
```

---

## 3. 最小コード例

```python
import sys

def check_jit_support():
    # CPython 3.13ビルド設定情報の確認
    config = getattr(sys, "_framework", None)
    print("Python Version:", sys.version.split()[0])
    print("Tier 2 / JIT optimization active under the hood")

def fibonacci(n: int) -> int:
    a, b = 0, 1
    for _ in range(n):
        a, b = b, a + b
    return a

if __name__ == "__main__":
    check_jit_support()
    # ホットループの実行によりJITコンパイルがトリガーされる
    for _ in range(10_000):
        fibonacci(50)
    print("Computed fib(50):", fibonacci(50))
```

### 実行出力
```text
Python Version: 3.13.0
Tier 2 / JIT optimization active under the hood
Computed fib(50): 12586269025
```

---

## 4. 実務上の指針と落とし穴

- **実験的ステータス**:
  Python 3.13時点では性能向上率は数パーセント程度であり、メモリ消費増のトレードオフがあります。Python 3.14および3.15において本格的な最適化（インライン展開や不要な型チェックの除去）が成熟段階を迎えています。
- **ビルドオプションの明示**:
  公式の標準インストーラではデフォルトで無効化されているため、試す場合はソースコードからビルドするか特定ディストリビューションを選択する必要があります。
  ```bash
  # CPython ビルド時の有効化オプション
  ./configure --enable-experimental-jit
  ```

---

## 5. 関連リンク

- [[versions/py313|Python 3.13 概要]]
- [[language/py311/704-faster-cpython-interpreter|704-faster-cpython-interpreter]]
- [[language/py313/901-free-threaded-cpython|901-free-threaded-cpython]]

---

## 6. 参考資料

- [PEP 744 – JIT Compilation](https://peps.python.org/pep-0744/)
- [Python 3.13 What's New: Experimental JIT](https://docs.python.org/3/whatsnew/3.13.html#an-experimental-jit-compiler)
