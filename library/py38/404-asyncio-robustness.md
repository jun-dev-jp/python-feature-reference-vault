---
title: "asyncio の堅牢化と asyncio.run() の定着"
aliases: [asyncio堅牢化, asyncio.run, 非同期実行モデル]
category: library
python_version: "3.8"
peps: []
created: 2026-10-26
updated: 2026-10-26
tags: [py38, library, async, concurrency, eventloop]
---

# asyncio の堅牢化と asyncio.run() の定着

## 1. 概要と动機

Python 3.5〜3.6の `asyncio` は、イベントループの初期化・終了処理を手動で行う（`loop = get_event_loop()`, `loop.close()`）必要があり、非同期ジェネレータの終了や保留中タスクのキャンセル処理に漏れが生じやすい構造でした。

Python 3.7 で導入された `asyncio.run()` は、Python 3.8 においてWindows環境でのデフォルトループの改善（`ProactorEventLoop` への切り替え）やタスク名指定、キャンセル処理の堅牢化を経て、モダンな非同期Pythonの標準的な実行エントリーポイントとして完全に定着しました。

---

## 2. 仕様と構文

- **ライフサイクル自動管理**:
  `asyncio.run(coro)` は、新しいイベントループを作成し、渡されたコルーチンを実行完了するまで待機し、残存するすべての保留中タスクをキャンセルしてループを確実にクローズします。
- **タスク名のサポート**:
  `asyncio.create_task(coro, name="worker-1")` によりタスクに名前を付与可能になり、デバッグ性が向上。

```python
# モダンな asyncio エントリーポイント
async def main():
    task = asyncio.create_task(coro(), name="named-worker")
    await task

if __name__ == "__main__":
    asyncio.run(main())  # ループ作成・タスク監視・クリーンアップを自動実行
```

---

## 3. 最小コード例

```python
import asyncio

async def background_worker():
    await asyncio.sleep(0.01)
    return "Finished successfully"

async def main():
    task = asyncio.create_task(background_worker(), name="my_task")
    print("Task created with name:", task.get_name())
    result = await task
    print("Result:", result)

if __name__ == "__main__":
    # ライフサイクルを完全にカプセル化した安全な起動
    asyncio.run(main())
```

### 実行出力
```text
Task created with name: my_task
Result: Finished successfully
```

---

## 4. 実務上の指針と落とし穴

- **入れ子での `asyncio.run()` 呼び出し禁止**:
  既にイベントループが実行されている環境（例: Jupyter Notebookや既存の `async def` 内）で `asyncio.run()` を呼ぶと `RuntimeError: asyncio.run() cannot be called from a running event loop` が発生します。
  ```python
  # 誤り: 実行中ループ内で asyncio.run は呼べない
  async def bad_caller():
      # asyncio.run(sub_coro())  # RuntimeError 送出

      # 正しい対処: そのまま await するか create_task を使う
      await sub_coro()
  ```
- **後方互換性**:
  レガシーな `asyncio.get_event_loop()` を使ったコードは段階的に非推奨化されているため、新規コードでは常に `asyncio.run()` を使用してください。

---

## 5. 関連リンク

- [[versions/py38|Python 3.8 概要]]
- [[language/py35/101-async-await|101-async-await]]
- [[language/py311/701-exception-groups|701-exception-groups]]

---

## 6. 参考資料

- [Python 3.8 What's New: asyncio improvements](https://docs.python.org/3/whatsnew/3.8.html#asyncio)
