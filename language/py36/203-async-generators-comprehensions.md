---
title: "非同期ジェネレータと非同期内包表記（PEP 525 / PEP 530）"
aliases: [非同期ジェネレータ, 非同期内包表記, PEP 525, PEP 530]
category: language
python_version: "3.6"
peps: [525, 530]
created: 2026-10-26
updated: 2026-10-26
tags: [py36, language, async, generators, comprehensions]
---

# 非同期ジェネレータと非同期内包表記（PEP 525 / PEP 530）

## 1. 概要と動機

Python 3.5 のネイティブコルーチン（PEP 492）では、`async def` の中で `yield` を使用することは禁止されていました。そのため、ネットワークストリームや定期的なイベントストリームのように、非同期に値が順次届くデータをジェネレータとして簡潔に表現する手段がありませんでした。

PEP 525 は `async def` 内での `yield` を許可して「非同期ジェネレータ」を導入し、PEP 530 はこれらを内包表記内で直感的に扱える「非同期内包表記（`[i async for i in agen()]`）」を追加しました。

---

## 2. 仕様と構文

- **非同期ジェネレータ関数**: `async def` 関数内で `yield` を含むもの。呼び出すと非同期ジェネレータオブジェクトを返す。
- **インターフェース**: `__anext__()` メソッドを実装しており、各要素を待機しながら取得可能。
- **非同期内包表記**:
  - `[x async for x in async_iter]`
  - `{x: y async for x, y in async_iter}`
  - `{x async for x in async_iter}`
  - `(x async for x in async_iter)`（非同期ジェネレータ式）

```python
# 非同期ジェネレータと非同期内包表記の基本構文
async def count_up(n: int):
    for i in range(n):
        await asyncio.sleep(0.01)  # 非同期待機
        yield i                    # 値の逐次生成

# async def 内での非同期内包表記
results = [x * 2 async for x in count_up(3)]
```

---

## 3. 最小コード例

```python
import asyncio

async def ticker(delay: float, to: int):
    for i in range(to):
        await asyncio.sleep(delay)
        yield i

async def main():
    # 非同期ジェネレータを非同期内包表記で収集
    results = [x * 10 async for x in ticker(0.01, 4)]
    print("Collected ticks:", results)

if __name__ == "__main__":
    asyncio.run(main())
```

### 実行出力
```text
Collected ticks: [0, 10, 20, 30]
```

---

## 4. 実務上の指針と落とし穴

- **終了処理とクリーンアップ**:
  非同期ジェネレータが途中で中断された場合、`aclose()` メソッドが正しく呼ばれないと `ResourceWarning` や未完の非同期クリーンアップが生じるリスクがあります。イベントループのシャットダウン処理に注意してください。
- **通常の内包表記との混同**:
  非同期内包表記は `async def` 関数内、またはトップレベルの非同期コンテキストでのみ使用可能です。通常の同期関数内で `[x async for x in ...]` を書くと構文エラーになります。
  ```python
  # 誤り: 同期関数内では async for は使えない
  # def sync_fn():
  #     return [x async for x in async_gen()]  # SyntaxError

  # 正しい対処: async def の中で実行する
  async def async_fn():
      return [x async for x in async_gen()]
  ```

---

## 5. 関連リンク

- [[versions/py36|Python 3.6 概要]]
- [[language/py35/101-async-await|101-async-await]]
- [[language/py315/1503-unpacking-in-comprehensions|1503-unpacking-in-comprehensions]]

---

## 6. 参考資料

- [PEP 525 – Asynchronous Generators](https://peps.python.org/pep-0525/)
- [PEP 530 – Asynchronous Comprehensions](https://peps.python.org/pep-0530/)
