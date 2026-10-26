---
title: "コンテキスト変数（contextvarsモジュール: PEP 567）"
aliases: [contextvars, コンテキスト変数, PEP 567]
category: library
python_version: "3.7"
peps: [567]
created: 2026-10-26
updated: 2026-10-26
tags: [py37, library, async, concurrency, thread-local]
---

# コンテキスト変数（contextvarsモジュール: PEP 567）

## 1. 概要と動機

スレッドベースのプログラミングでは、リクエストIDやユーザ認証情報などのリクエストスコープのコンテキストを保持するために `threading.local()` が使われてきました。しかし、`asyncio` による非同期プログラミングでは、単一のスレッド上で複数のタスク（コルーチン）が協調的に切り替わりながら実行されるため、スレッドローカル変数ではタスクごとの状態を分離できず、データ競合や混ざり合いが発生していました。

PEP 567 は、非同期コルーチン間でも安全にコンテキストを独立して伝播・分離できる `contextvars` モジュールを導入しました。

---

## 2. 仕様と構文

- **`ContextVar(name, *, default)`**: コンテキスト変数を宣言。
- **`var.get(default=None)`**: 現在のコンテキストにおける値を取得。
- **`var.set(value)`**: 現在のコンテキストに値をセットし、復元用トークン（`Token`）を返す。
- **`var.reset(token)`**: トークンを用いて変更前の値に復元。
- **自動伝播**: `asyncio.create_task()` で新しいタスクを起動した際、親タスクのコンテキスト変数のスナップショットが子タスクに自動コピーされます。

```python
# コンテキスト変数の宣言と値の退避・復元
user_ctx = contextvars.ContextVar("user", default="guest")

token = user_ctx.set("admin")  # 値を設定してトークンを保持
try:
    current = user_ctx.get()   # "admin" を取得
finally:
    user_ctx.reset(token)      # 確実に元の状態へ復元
```

---

## 3. 最小コード例

```python
import asyncio
import contextvars

# リクエストIDを保持するコンテキスト変数
request_id_var: contextvars.ContextVar[str] = contextvars.ContextVar("request_id", default="none")

async def worker(task_name: str, req_id: str):
    # タスクごとに独立したコンテキストを設定
    request_id_var.set(req_id)
    await asyncio.sleep(0.01)
    print(f"[{task_name}] Processed request: {request_id_var.get()}")

async def main():
    # 2つの並行タスクが互いに干渉せず独自のリクエストIDを保持
    await asyncio.gather(
        worker("Task-A", "REQ-1001"),
        worker("Task-B", "REQ-1002")
    )

if __name__ == "__main__":
    asyncio.run(main())
```

### 実行出力
```text
[Task-A] Processed request: REQ-1001
[Task-B] Processed request: REQ-1002
```

---

## 4. 実務上の指針と落とし穴

- **Webフレームワークでの不可欠性**:
  FastAPI、Starlette、Loguru などのモダンな非同期エコシステムにおいて、ロギング、トレーシング、DBセッション管理の心臓部として利用されています。
- **ミュータブルオブジェクトの共有**:
  コンテキスト変数に辞書やリストなどのミュータブルなオブジェクトを格納し、子タスクがそのオブジェクト自体を変更した場合、同一オブジェクトであるため他のタスクにも変更が見えてしまいます。不変（イミュータブル）な値を格納するのが原則です。
  ```python
  # 落とし穴: 可変オブジェクトの直接変更は別タスクにも影響する
  ctx_data = contextvars.ContextVar("data", default={})
  # ctx_data.get()["count"] = 1  # 共有辞書が破壊される

  # 正しい対処: 辞書全体を新しいインスタンスに差し替えて set する
  new_state = {**ctx_data.get(), "count": 1}
  ctx_data.set(new_state)
  ```

---

## 5. 関連リンク

- [[versions/py37|Python 3.7 概要]]
- [[language/py35/101-async-await|101-async-await]]
- [[library/py38/404-asyncio-robustness|404-asyncio-robustness]]

---

## 6. 参考資料

- [PEP 567 – Context Variables](https://peps.python.org/pep-0567/)
- [Python 公式ドキュメント: contextvars](https://docs.python.org/3/library/contextvars.html)
