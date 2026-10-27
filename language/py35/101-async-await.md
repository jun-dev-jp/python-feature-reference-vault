---
title: "async / await 構文のネイティブ導入"
aliases: [async/await, ネイティブコルーチン, PEP 492]
category: language
python_version: "3.5"
peps: [492]
created: 2026-10-26
updated: 2026-10-26
tags: [py35, language, async, concurrency]
---

# async / await 構文のネイティブ導入

## 1. 概要と動機

Python 3.4 までは、非同期処理を行うために `@asyncio.coroutine` デコレータと `yield from` 構文が用いられていました。しかし、この方式には以下の課題がありました。
- **ジェネレータとの混同**: 通常のイテレータを生成するジェネレータと、非同期処理を行うコルーチンが構文上区別できず、可読性と静的解析の難度が高かった。
- **意図しないミス**: コルーチン関数の呼び出し忘れや `yield from` の記述漏れが実行時に捕捉しづらかった。

PEP 492 は `async def` および `await` を専用のキーワードとして定義し、言語仕様レベルで「ネイティブコルーチン」を導入しました。

---

## 2. 仕様と構文

- **`async def`**: コルーチン関数を定義する。呼び出すとコルーチンオブジェクトを返す（即時実行はされない）。
- **`await expression`**: 式が awaitable（`__await__()` メソッドを持つオブジェクト、コルーチン、タスク、Future）である場合に実行を一時停止し、完了を待つ。通常の同期関数内では使用不可。
- **非同期イテレータ (`async for`)**: `__aiter__()` と `__anext__()` を用いて非同期ストリームをループ処理する。
- **非同期コンテキストマネージャ (`async with`)**: `__aenter__()` と `__aexit__()` を用いて非同期リソースを安全に管理する。

```python
# 非同期構文の基本シグネチャとコンテキスト制御
async def process_item(item: str) -> None:
    # await は async def の内部でのみ使用可能
    await asyncio.sleep(0.01)

# 非同期コンテキストマネージャの利用
async with lock:
    await process_item("data")
```

---

## 3. 従来記法との比較

Python 3.4 以前の `@asyncio.coroutine` / `yield from` 方式と、Python 3.5 のネイティブ `async` / `await` 構文の比較です。

```python
import asyncio

# --- 従来（Python 3.4以前）: ジェネレータベースのコルーチン ---
# @asyncio.coroutine デコレータと yield from を使用。
# 通常のジェネレータ関数と見分けがつかず、構文上の保護がなかった。
@asyncio.coroutine
def legacy_fetch(data_id):
    yield from asyncio.sleep(0.01)
    return {"id": data_id, "mode": "legacy"}

# --- Python 3.5+: ネイティブコルーチン構文 ---
# 専用キーワード async def と await により、コルーチンであることが言語仕様上明確。
async def modern_fetch(data_id: int) -> dict:
    await asyncio.sleep(0.01)
    return {"id": data_id, "mode": "modern"}

# 実行検証
async def main():
    res1 = yield from legacy_fetch(1)  # Python 3.4互換（3.8で非推奨、3.11で削除）
    res2 = await modern_fetch(2)
    print("Legacy:", res1)
    print("Modern:", res2)
```

---

## 4. 最小コード例

Python 3.5 以降で動作する独立した非同期タスク実行の例です。

```python
import asyncio

async def fetch_data(item_id: int) -> dict:
    await asyncio.sleep(0.01)  # 非同期I/O待ちのシミュレーション
    return {"id": item_id, "status": "ok"}

async def main():
    print("Start fetching...")
    result = await fetch_data(42)
    print("Result:", result)

if __name__ == "__main__":
    loop = asyncio.get_event_loop()
    loop.run_until_complete(main())
```

### 実行出力
```text
Start fetching...
Result: {'id': 42, 'status': 'ok'}
```

---

## 5. 実務上の指針と落とし穴

- **コルーチンオブジェクトの放置**:
  `fetch_data(42)` を単体で呼び出し、`await` を忘れると実行されず、`RuntimeWarning: coroutine 'fetch_data' was never awaited` が発生します。
  ```python
  # 典型的なアンチパターン（awaitの欠落）
  fetch_data(42)  # 警告発生: 処理は実際には実行されない

  # 正しい記述
  await fetch_data(42)  # イベントループに処理がスケジュールされる
  ```
- **CPUバウンドな処理の混入**:
  `async def` 内で重い計算や同期I/O（`time.sleep()` や同期HTTPクライアント）を実行すると、イベントループ全体をブロックして並行性が失われます。
- **Python 3.7+ 移行時の注意**:
  Python 3.7 から `asyncio.run(main())` が推奨され、`get_event_loop()` の直接呼び出しは非推奨化へ向かいました。

---

## 6. 関連リンク

- [[versions/py35|Python 3.5 概要]]
- [[language/py36/203-async-generators-comprehensions|203-async-generators-comprehensions]]
- [[library/py38/404-asyncio-robustness|404-asyncio-robustness]]
- [[library/py37/304-contextvars|304-contextvars]]

---

## 7. 参考資料

- [PEP 492 – Coroutines with async and await syntax](https://peps.python.org/pep-0492/)
- [Python 3.5 What's New: PEP 492](https://docs.python.org/3/whatsnew/3.5.html#pep-492-coroutines-with-async-and-await-syntax)
