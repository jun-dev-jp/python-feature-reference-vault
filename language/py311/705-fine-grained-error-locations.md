---
title: "トレースバックでの詳細なエラー箇所表示 (Fine-grained Error Locations, PEP 657)"
aliases: [PEP 657, 高度トレースバック, エラー位置下線表示, Traceback下線]
category: language
python_version: "3.11"
peps: [657]
created: 2026-10-26
updated: 2026-10-26
tags: [py311, language, traceback, debugging, pep657]
---

# トレースバックでの詳細なエラー箇所表示 (Fine-grained Error Locations, PEP 657)

---

## 1. 概要と動機

Python 3.10 までのトレースバックは、例外が発生した「ソースコードの行番号」のみを表示していました。そのため、以下のような1行に複数の操作が含まれる式でエラーが起きた場合、どの操作が原因か即座に判別できませんでした：
- `user["profile"]["address"]["zipcode"]`（どのキーが存在しなかったのか？）
- `a.b.c.method()`（どの属性が `None` だったのか？）
- `(x + y) / (z - w)`（どちらの括弧または演算でゼロ除算や型エラーが起きたのか？）

Python 3.11 の **PEP 657** により、コンパイラがバイトコードとソースコードの「正確な開始列・終了列（Column offset）」を対応付けるテーブルを生成し、トレースバック上で問題の式そのものを下線（`^^^^^^^^`）でピンポイントに指し示すようになりました。

---

## 2. 仕様と構文

- **列情報の追跡**:
  コードオブジェクトの `.co_positions()` メソッドにより、各バイトコード命令に対応する `(start_line, end_line, start_col, end_col)` のタプルが提供されます。
- **トレースバック出力の強化**:
  例外発生時、対象の式全体の下にキャレット記号（`^` や `~`）が自動描画されます。
- **無効化オプション**:
  コードオブジェクトのサイズを削減したい特殊な制約環境では、環境変数 `PYTHONNODEBUGRANGES=1` またはコマンドライン引数 `-X no_debug_ranges` で列情報記録を無効化できます。

```python
# 例: 複雑なインデックスアクセス
data = {"users": [{"name": "Alice"}]}

# Python 3.11+ では "age" にピンポイントで下線が表示される
# data["users"][0]["age"]
# ~~~~~~~~~~~~~~~~~^^^^^^^
# KeyError: 'age'
```

---

## 3. 最小コード例

```python
def calculate_metric(payload: dict) -> float:
    # 複数キーアクセスと計算が1行に密集した処理
    return payload["metrics"]["primary"]["score"] / payload["metrics"]["secondary"]["weight"]

if __name__ == "__main__":
    sample = {
        "metrics": {
            "primary": {"score": 100},
            "secondary": None  # ここで NoneType has no attribute '__getitem__' が発生
        }
    }
    try:
        calculate_metric(sample)
    except TypeError as exc:
        import traceback
        print("Captured traceback with PEP 657 column indicators:")
        traceback.print_exc(limit=1)
```

### 実行出力
```text
Captured traceback with PEP 657 column indicators:
Traceback (most recent call last):
  File "example.py", line 3, in calculate_metric
    return payload["metrics"]["primary"]["score"] / payload["metrics"]["secondary"]["weight"]
                                                     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^
TypeError: 'NoneType' object is not subscriptable
```

---

## 4. 実務上の指針と落とし穴

- **デバッグ作業の大幅な効率化**:
  原因特定のために「1行を複数行に分解して一時変数を置く」という旧来のデバッグ手順が不要になりました。
- **メモリ消費量への影響**:
  列位置テーブルの保持により、コンパイル済みバイトコード（`.pyc`）のサイズはわずかに（約22%程度）増加しますが、実行時の実行速度オーバーヘッドはゼロです。メモリが極度に制約された組み込み環境以外では無効化する必要はありません。

```python
# ✕ かつて原因特定のために行われていた冗長な一時変数分割
m = payload["metrics"]
sec = m["secondary"]  # どこで落ちたか特定するためだけに代入していた
w = sec["weight"]

# ◯ Python 3.11以降: 自然な式記述のままで正確に例外位置が判明する
w = payload["metrics"]["secondary"]["weight"]
```

---

## 5. 関連リンク

- [[versions/py311|Python 3.11 概要]]
- [[language/py311/701-exception-groups|701-exception-groups]]
- [[ROADMAP|全体ロードマップ]]

---

## 6. 参考資料

- [PEP 657 – Include Fine-Grained Error Locations in Tracebacks](https://peps.python.org/pep-0657/)
- [Python 3.11 What's New: Better error locations in tracebacks](https://docs.python.org/ja/3/whatsnew/3.11.html#enhanced-error-locations-in-tracebacks)
