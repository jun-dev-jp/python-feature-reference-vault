---
title: "例外グループと except* 構文"
aliases: [例外グループ, except*, ExceptionGroup, PEP 654]
category: language
python_version: "3.11"
peps: [654]
created: 2026-10-26
updated: 2026-10-26
tags: [py311, language, syntax, exceptions, async]
---

# 例外グループと except* 構文

## 1. 概要と動機

並行処理（`asyncio.gather` やタスクグループ、スレッドプール等）では、複数の独立した並行処理タスクが同時に異なる例外を発生させることがあります。従来のPythonの例外モデルでは一度に1つの例外しか `raise` できず、最初の1つ以外の例外が握りつぶされるか、リストに詰め込んだカスタムオブジェクトを投げるしかありませんでした。

PEP 654 は、無関係な複数の例外を1つにまとめて伝播できる `ExceptionGroup`（および致命的エラー用の `BaseExceptionGroup`）と、それらを個別または型ごとに選択的・部分的に捕捉できる新しい構文 `except*` を導入しました。

---

## 2. 仕様と構文

- **`ExceptionGroup(message, exceptions)`**:
  複数の例外インスタンスをラップして投げるコンテナ型。ツリー構造を持つことができる。
- **`try ... except* Type:` 構文**:
  - `except*` 節は、グループ内に含まれる例外のうち指定した型に一致するものをすべて分離して処理する。
  - 一致しなかった残りの例外は自動的に再構成され、次の `except*` または外側へ再送出（re-raise）される。
  - 通常の `except` と異なり、1つの `try` ブロックに対して複数の `except*` 節がマッチして実行される可能性がある。
  - `except` と `except*` を同一の `try` 文で混在させることは禁止。

```python
# except* による例外の部分捕捉構文
try:
    raise ExceptionGroup("複数エラー", [ValueError("a"), TypeError("b")])
except* ValueError as e:
    # ValueError のみを取り出して処理
    pass
except* TypeError as e:
    # 同一 try から連続してこちらも実行される
    pass
```

---

## 3. 従来のエラーハンドリングとの比較

Python 3.10 以前の単一例外モデルや集約例外クラスの手動管理と、Python 3.11 の `ExceptionGroup` / `except*` 構文の比較です。

```python
# --- 従来（Python 3.10以前）: 複数エラーのリスト保持と手動ループ判定 ---
# 複数の並行エラーを表現する言語構文がなく、独自クラス内でリストを保持して
# 単一の except 節で受け取り、内部で isinstance による手動仕分けが必要だった。
class LegacyMultiError(Exception):
    def __init__(self, errors):
        self.errors = errors

try:
    raise LegacyMultiError([ValueError("invalid val"), TypeError("wrong type")])
except LegacyMultiError as err:
    for e in err.errors:
        if isinstance(e, ValueError):
            print("Legacy handled ValueError:", e)
        elif isinstance(e, TypeError):
            print("Legacy handled TypeError:", e)

# --- Python 3.11+: ExceptionGroup と except* 構文 ---
# 言語組み込みの構文で、一致する型ごとに宣言的かつ個別にブロックを分けて捕捉可能
try:
    raise ExceptionGroup("Batch errors", [ValueError("invalid val"), TypeError("wrong type")])
except* ValueError as eg:
    print("Modern handled ValueError:", eg.exceptions)
except* TypeError as eg:
    print("Modern handled TypeError:", eg.exceptions)
```

---

## 4. 最小コード例

Python 3.11 以降で動作する例外グループの送出と `except*` による分割捕捉の例です。

```python
def trigger_errors():
    errors = [
        ValueError("Invalid configuration value"),
        TypeError("Expected string, got int"),
        ValueError("Number out of bounds"),
    ]
    raise ExceptionGroup("Batch validation failed", errors)

def handle_errors():
    try:
        trigger_errors()
    except* ValueError as eg:
        print(f"Handled {len(eg.exceptions)} ValueError(s):")
        for e in eg.exceptions:
            print(" -", e)
    except* TypeError as eg:
        print(f"Handled {len(eg.exceptions)} TypeError(s):")
        for e in eg.exceptions:
            print(" -", e)

if __name__ == "__main__":
    handle_errors()
```

### 実行出力
```text
Handled 2 ValueError(s):
 - Invalid configuration value
 - Number out of bounds
Handled 1 TypeError(s):
 - Expected string, got int
```

---

## 5. 実務上の指針と落とし穴

- **`asyncio.TaskGroup` との併用**:
  Python 3.11 で追加された `asyncio.TaskGroup` は、管理下のいずれかのタスクが失敗すると残りをキャンセルし、すべての失敗を `ExceptionGroup` にまとめて送出します。タスクグループのエラーハンドリングには `except*` が必須となります。
- **`except Exception` での不完全な捕捉**:
  通常の `except Exception:` で `ExceptionGroup` を受けると、グループ全体が丸ごと捕捉され、中身の個別エラー型に応じたきめ細やかな処理が難しくなります。並行処理エラーには `except*` を使用してください。
  ```python
  # 通常の except と except* は同一 try 内で併用できない
  # try: ...
  # except ValueError: ...   # 併用すると SyntaxError
  # except* TypeError: ...
  ```
- **例外の未処理残余**:
  `except*` で一部の型だけを捕捉し、他の型がグループに残っていた場合、未処理の例外群が再束縛されて上位スコープに自動で浮上します。

---

## 6. 関連リンク

- [[versions/py311|Python 3.11 概要]]
- [[language/py35/101-async-await|101-async-await]]
- [[library/py38/404-asyncio-robustness|404-asyncio-robustness]]

---

## 7. 参考資料

- [PEP 654 – Exception Groups and except*](https://peps.python.org/pep-0654/)
- [Python 3.11 What's New: Exception Groups and except*](https://docs.python.org/3/whatsnew/3.11.html#pep-654-exception-groups-and-except)
