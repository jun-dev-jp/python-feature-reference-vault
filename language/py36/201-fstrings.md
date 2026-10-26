---
title: "フォーマット済み文字列リテラル（f-strings）"
aliases: [f-strings, フォーマット済み文字列, PEP 498]
category: language
python_version: "3.6"
peps: [498]
created: 2026-10-26
updated: 2026-10-26
tags: [py36, language, syntax, strings]
---

# フォーマット済み文字列リテラル（f-strings）

## 1. 概要と動機

Python 3.5以前の文字列フォーマットには `%` 演算子（`"%s %d" % (name, age)`）や `str.format()`（`"{} {}".format(name, age)`）が存在しましたが、以下の問題点がありました。
- **冗長性**: 変数名や式を文字列の末尾で再度繰り返す必要があり、引数が増えるほど読みづらい。
- **実行オーバーヘッド**: `str.format()` は関数呼び出しのオーバーヘッドがあり低速。
- **エラー誘発**: `%` フォーマットはタプルの扱いや型の不一致によるバグが起きやすい。

PEP 498 により導入された f-strings は、文字列リテラルの先頭に `f` または `F` を付けることで、文字列内に直接 Python 式を埋め込む記法を提供しました。

---

## 2. 仕様と構文

- プレフィックス: `f"..."` または `F"..."`
- 置換フィールド: 波括弧 `{expression}` 内に任意の有効な式を記述可能。
- 書式指定子: コロンに続けて書式を指定可能（例: `{value:.2f}`, `{number:04d}`）。
- 変換フラグ: `!r` (`repr`), `!s` (`str`), `!a` (`ascii`) を末尾に指定可能。

```python
# f-strings の基本書式指定とフラグ指定
pi = 3.141592
msg = f"円周率: {pi:.2f}"     # "円周率: 3.14"（小数点以下2桁）
raw = f"デバッグ: {msg!r}"    # repr形式で展開
```

---

## 3. 最小コード例

Python 3.6 以降で動作する f-strings の基本例です。

```python
name = "Alice"
score = 87.654
items = ["apple", "banana"]

# 式の埋め込みと書式指定
message = f"User: {name.upper()}, Score: {score:.1f}, Items: {len(items)}"
print(message)

# repr変換フラグ
debug_info = f"Raw items: {items!r}"
print(debug_info)
```

### 実行出力
```text
User: ALICE, Score: 87.7, Items: 2
Raw items: ['apple', 'banana']
```

---

## 4. 実務上の指針と落とし穴

- **セキュリティ（SQLインジェクション等のリスク）**:
  SQLクエリやシェルコマンドの構築に f-strings を直接使用してはいけません。エスケープ処理が行われないため、SQLインジェクション等の脆弱性を生みます（安全な構造化テンプレートは Python 3.14 の PEP 750 `t-strings` を参照）。
  ```python
  # 危険: SQLインジェクション脆弱性
  # query = f"SELECT * FROM users WHERE name = '{user_input}'"

  # 安全: パラメータ化クエリを使用する
  cursor.execute("SELECT * FROM users WHERE name = ?", (user_input,))
  ```
- **Python 3.6〜3.11における構文制約**:
  - Python 3.11 までは `{}` 内部で文字列と同じ引用符を使用できず（`f"{data['key']}"` で内側と外側のクォートを分ける必要があった）、バックスラッシュ（`\n` 等）を波括弧内に書けませんでした。これらは Python 3.12（PEP 701）で正式に撤廃されました。
- **デバッグ表示（Python 3.8+）**:
  Python 3.8 以降では `f"{variable=}"` と記述することで、変数名と値を同時に出力できるようになりました。

---

## 5. 関連リンク

- [[versions/py36|Python 3.6 概要]]
- [[language/py312/802-syntactic-formalization-fstrings|802-syntactic-formalization-fstrings]]
- [[language/py314/1401-template-strings|1401-template-strings]]

---

## 6. 参考資料

- [PEP 498 – Literal String Interpolation](https://peps.python.org/pep-0498/)
- [Python 3.6 What's New: PEP 498](https://docs.python.org/3/whatsnew/3.6.html#pep-498-formatted-string-literals)
