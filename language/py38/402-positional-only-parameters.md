---
title: "位置専用パラメータ構文（/ : PEP 570）"
aliases: [位置専用引数, PEP 570, positional-only parameters]
category: language
python_version: "3.8"
peps: [570]
created: 2026-10-26
updated: 2026-10-26
tags: [py38, language, syntax, functions, api-design]
---

# 位置専用パラメータ構文（/ : PEP 570）

## 1. 概要と動機

C言語で実装されたPython組み込み関数（例: `len(obj)`, `pow(x, y)`）は、引数をキーワードとして渡すこと（`len(obj=...)`）を許可せず、位置のみで受け付けていました。一方で、純粋なPython関数ではすべての引数が自動的にキーワード引数としても指定可能でした。
これにより、以下の課題がありました。
- ライブラリ作者が引数名をリファクタリング（改名）しただけで、利用者がキーワード指定していた場合に後方互換性が破壊される。
- `**kwargs` を受け取る関数で、引数名が `kwargs` 内のキーと衝突する。

PEP 570 は、スラッシュ `/` を用いて「位置専用パラメータ（Positional-Only Parameters）」を純粋なPython関数でも定義できるようにしました。

---

## 2. 仕様と構文

- **構文**: `def func(pos_only1, pos_only2, /, pos_or_kwd, *, kwd_only): ...`
- **スラッシュ `/` の左側**: 位置引数としてのみ渡すことができ、キーワード指定すると `TypeError`。
- **アスタリスク `*` の右側**: キーワード引数としてのみ渡すことができる（PEP 3102）。
- **中間**: 位置またはキーワードのどちらでも指定可能。

```python
# 引数種別の境界指定構文
def api_func(x, y, /, standard_arg, *, option=True):
    # x, y は位置専用、option はキーワード専用
    return (x + y, standard_arg, option)
```

---

## 3. 最小コード例

```python
def configure_user(user_id, /, name, *, is_admin=False):
    return {"id": user_id, "name": name, "admin": is_admin}

if __name__ == "__main__":
    # 正しい呼び出し
    print(configure_user(101, name="Alice", is_admin=True))
    print(configure_user(102, "Bob"))

    # キーワードとして渡すと例外
    try:
        configure_user(user_id=103, name="Charlie")
    except TypeError as e:
        print("Error as expected:", e)
```

### 実行出力
```text
{'id': 101, 'name': 'Alice', 'admin': True}
{'id': 102, 'name': 'Bob', 'admin': False}
Error as expected: configure_user() got some positional-only arguments passed as keyword arguments: 'user_id'
```

---

## 4. 実務上の指針と落とし穴

- **APIの保守性向上**:
  引数名に意味がなく単なる入力値である場合（例: `math.sin(x)` やシリアライザの入力値）、`/` を付けることで、将来引数名を変更しても呼び出し側のコードを壊さない安全なAPIが構築できます。
- **kwargs との衝突回避**:
  引数名を位置専用にすることで、可変長キーワード引数 `**kwargs` に渡された同一キー名との名前衝突を完全に防止できます。
  ```python
  def format_entry(name, /, **kwargs):
      # kwargs に "name" というキーが含まれていても問題なく受容可能
      return {"entry_name": name, "extra": kwargs}

  print(format_entry("alice", name="Bob in kwargs"))
  ```
- **ドキュメントの表記**:
  Python公式ドキュメントや `help()` 出力でも `/` と `*` が明確に引数の受容形式を示します。

---

## 5. 関連リンク

- [[versions/py38|Python 3.8 概要]]
- [[language/py38/401-walrus-operator|401-walrus-operator]]

---

## 6. 参考資料

- [PEP 570 – Python Positional-Only Parameters](https://peps.python.org/pep-0570/)
- [Python 3.8 What's New: PEP 570](https://docs.python.org/3/whatsnew/3.8.html#positional-only-parameters)
