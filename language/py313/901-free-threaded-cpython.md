---
title: "実験的フリースレッドCPython（GIL無効化ビルド）"
aliases: [フリースレッド, Free-threaded Python, GIL無効化, PEP 703, nogil]
category: language
python_version: "3.13"
peps: [703]
created: 2026-10-26
updated: 2026-10-26
tags: [py313, language, gil, concurrency, experimental, performance]
---

# 実験的フリースレッドCPython（GIL無効化ビルド）

## 1. 概要と動機

CPythonは歴史的に「GIL（グローバルインタプリタロック）」を保持しており、マルチスレッド環境下でも同時に1つのOSスレッドしかPythonバイトコードを実行できませんでした。これにより、マルチコアCPUの計算リソースを活用するには `multiprocessing` によるプロセス間通信（IPC）が必須となり、共有メモリオブジェクトのシリアライズやIPCのオーバーヘッドが大きな課題でした。

PEP 703 は、参照カウントの最適化（Biased Reference Counting）やメモリアロケータ（mimalloc）の刷新により、**GILを完全に無効化してマルチスレッドで真の並列実行を可能にするフリースレッドビルド（Free-threaded build）**をCPython 3.13に実験的機能として導入しました。

---

## 2. 仕様と構文

- **ビルド構成**:
  Python 3.13 では通常のGIL付きバイナリとは別に、`--disable-gil` オプションでビルドされたフリースレッドバイナリ（`python3.13t`）として提供される。
- **実行時フラグ**:
  環境変数 `PYTHON_GIL=0` または起動オプション `-X gil=0` により、GILの有効/無効を切り替え可能。
- **スレッドセーフティ**:
  言語レベルの辞書・リストなどの組み込みオブジェクトアクセスは内部ロックおよび不変スナップショット等で保護されるが、ユーザー定義の共有データ構造の競合状態（Race condition）はプログラマが明示的に同期（Lock等）する必要がある。

```python
# フリースレッド環境でのGIL有効状態の確認
import sys
gil_active = sys._is_gil_enabled()  # False の場合フリースレッド動作中
```

---

## 3. 最小コード例

Python 3.13 (フリースレッドビルド) でGILの状態を確認し、マルチスレッドで並列計算を行うコード例です。

```python
import sys
import threading
import time

def check_gil_status():
    # Python 3.13で追加されたGIL状態確認API
    status = getattr(sys, "_is_gil_enabled", None)
    if status is not None:
        print("GIL is enabled:", status())
    else:
        print("Standard build (GIL present)")

def cpu_heavy_work(count: int):
    total = 0
    for i in range(count):
        total += (i % 7) * (i % 3)
    return total

if __name__ == "__main__":
    check_gil_status()

    # フリースレッド環境では2つのスレッドが別コアで真に同時並列実行される
    t1 = threading.Thread(target=cpu_heavy_work, args=(1_000_000,))
    t2 = threading.Thread(target=cpu_heavy_work, args=(1_000_000,))

    start = time.perf_counter()
    t1.start(); t2.start()
    t1.join(); t2.join()
    print(f"Elapsed: {time.perf_counter() - start:.3f}s")
```

### 実行出力（python3.13t 環境）
```text
GIL is enabled: False
Elapsed: 0.084s
```

---

## 4. 実務上の指針と落とし穴

- **実験的（Experimental）ステータス**:
  Python 3.13時点では明確に実験的ステータスです。シングルスレッド性能は通常のGIL有効版と比べて約5〜15%程度オーバーヘッド（Biased Reference Countingのコスト）があります。
- **C拡張モジュール（C-API）の互換性**:
  NumPy、PyTorchなどのC拡張ライブラリがフリースレッドビルドに対応（Thread-safe化）している必要があります。未対応の拡張を読み込むと自動的にGILが再有効化される仕組みがあります。
  ```python
  # 未対応C拡張のインポート時にGILが自動再有効化される場合の警告
  # RuntimeWarning: The global interpreter lock has been re-enabled
  ```
- **将来のロードマップ（PEP 703フェーズ）**:
  コミュニティの移行状況に応じて、Python 3.14で安定化を進め、3.15以降にフリースレッド版をデフォルトビルドにするかどうかが議論されています。

---

## 5. 関連リンク

- [[versions/py313|Python 3.13 概要]]
- [[language/py312/804-per-interpreter-gil|804-per-interpreter-gil]]
- [[language/py313/902-experimental-jit|902-experimental-jit]]
- [[versions/py315|Python 3.15 概要]]

---

## 6. 参考資料

- [PEP 703 – Making the Global Interpreter Lock Optional in CPython](https://peps.python.org/pep-0703/)
- [Python 3.13 What's New: Free-threaded CPython](https://docs.python.org/3/whatsnew/3.13.html#free-threaded-cpython)
