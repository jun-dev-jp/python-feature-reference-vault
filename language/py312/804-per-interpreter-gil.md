---
title: "サブインタプリタごとのGIL分離（Per-Interpreter GIL: PEP 684）"
aliases: [Per-Interpreter GIL, PEP 684, サブインタプリタGIL分離]
category: language
python_version: "3.12"
peps: [684]
created: 2026-10-26
updated: 2026-10-26
tags: [py312, language, gil, concurrency, interpreters]
---

# サブインタプリタごとのGIL分離（Per-Interpreter GIL: PEP 684）

## 1. 概要と動機

CPythonは古くからC-APIを通じて「サブインタプリタ（Sub-interpreters）」を作成する機能を備えていましたが、すべてのインタプリタが単一のグローバルなGILを共有していたため、別々のインタプリタであっても真のマルチコア並列実行は不可能でした。

PEP 684 は、CPythonの内部グローバル状態（メモリアロケータや型オブジェクト等）をインタプリタごとにカプセル化し、**サブインタプリタごとに独立したGIL（Per-Interpreter GIL）**を持たせることに成功しました。これにより、1つのOSプロセス内で真のマルチコア並列実行が実現可能になりました。

---

## 2. 仕様と構文

- **C-API**: `Py_NewInterpreterFromConfig()` 等で独立したGILを持つサブインタプリタを起動可能。
- **PythonレベルのAPI**: Python 3.12時点では低水準な内部モジュール（`_xxsubinterpreters`）として試験提供され、Python 3.13 / 3.14 に向けて高水準な `interpreters` モジュールの標準化（PEP 554 / PEP 734）が進められています。

```python
# サブインタプリタの生成と文字列実行モデル
interp_id = subinterpreters.create()
subinterpreters.run_string(interp_id, "x = 42")  # 独立したGIL下で実行
```

---

## 3. 最小コード例

Python 3.12+ の内部サブインタプリタAPIを用いた並列実行の概念例です。

```python
import _xxsubinterpreters as subinterpreters
import threading

def run_in_subinterpreter(interp_id):
    subinterpreters.run_string(
        interp_id,
        "import sys; print(f'Running in isolated subinterpreter {sys.version_info.major}.{sys.version_info.minor}')"
    )

if __name__ == "__main__":
    interp_id = subinterpreters.create()
    t = threading.Thread(target=run_in_subinterpreter, args=(interp_id,))
    t.start()
    t.join()
    subinterpreters.destroy(interp_id)
```

### 実行出力
```text
Running in isolated subinterpreter 3.12
```

---

## 4. 実務上の指針と落とし穴

- **PEP 703（フリースレッド）との違い**:
  - **PEP 684（Per-Interpreter GIL）**: インタプリタ単位でメモリ空間が分離されており、データ共有はチャネル通信等で行う（ErlangやGoのGoroutineに近いモデル）。
  - **PEP 703（Free-threaded）**: 単一のインタプリタ内でスレッドが同一のオブジェクトメモリを直接共有するモデル。
- 双方のアプローチが並行して成熟に向かっています。
- **オブジェクト共有の制限**:
  サブインタプリタ間では直接Pythonオブジェクトへの参照を渡すことができず、バイト列やイミュータブルなデータへの変換が必要です。
  ```python
  # 共有データはチャネル経由でシリアライズして渡す必要がある
  # subinterpreters.set_shared_channel(...)
  ```

---

## 5. 関連リンク

- [[versions/py312|Python 3.12 概要]]
- [[language/py313/901-free-threaded-cpython|901-free-threaded-cpython]]

---

## 6. 参考資料

- [PEP 684 – A Per-Interpreter GIL](https://peps.python.org/pep-0684/)
- [Python 3.12 What's New: PEP 684](https://docs.python.org/3/whatsnew/3.12.html#pep-684-a-per-interpreter-gil)
