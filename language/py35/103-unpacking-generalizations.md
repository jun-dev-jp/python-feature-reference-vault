---
title: "辞書・リストのアンパック一般化（PEP 448）"
aliases: [アンパック一般化, PEP 448, 辞書アンパック]
category: language
python_version: "3.5"
peps: [448]
created: 2026-10-26
updated: 2026-10-26
tags: [py35, language, syntax, unpacking]
---

# 辞書・リストのアンパック一般化（PEP 448）

## 1. 概要と動機

Python 3.4 までは、アンパック演算子（`*` や `**`）は関数呼び出しの引数位置でのみ使用可能であり、1回の呼び出しにつきそれぞれ1つしか指定できませんでした。タプルやリストの結合には `+` や `extend()`、辞書の結合には `dict.update()` や `copy()` の多段階呼び出しが必要で冗長でした。

PEP 448 は、タプル・リスト・セット・辞書のリテラル構築式において、任意の回数の `*` および `**` アンパックを許可しました。

---

## 2. 仕様と構文

- **シーケンス・リテラル**: `[*iterable_1, *iterable_2, item]`
- **辞書リテラル**: `{**dict_1, **dict_2, "key": value}`
  - 重複するキーがある場合は、後から現れた値が上書きされます。
- **関数呼び出し**: `func(*a, *b, **kw1, **kw2)` のように複数のアンパック引数を指定可能。

```python
# リテラル内での複数アンパック記述
combined_list = [*list_a, 100, *list_b]
merged_dict = {**dict_a, "extra": True, **dict_b}
```

---

## 3. 従来記法との比較

Python 3.4 以前のコレクション結合・辞書マージ手法と、PEP 448 によるアンパック一般化の比較です。

```python
# --- 1. リスト / シーケンスの結合 ---
base = [1, 2]
extra = (3, 4)

# 従来（Python 3.4以前）: + 演算子では型の一致が必要、または extend() で破壊的操作が必要
legacy_list = list(base) + list(extra) + [5]

# Python 3.5+ (PEP 448): 異なるイテラブルも1つのリテラル内で直接展開
modern_list = [0, *base, *extra, 5]


# --- 2. 辞書のマージ ---
defaults = {"host": "localhost", "port": 8080}
custom = {"port": 9000, "debug": True}

# 従来（Python 3.4以前）: copy() と update() の複数行手続きが必要
legacy_dict = defaults.copy()
legacy_dict.update(custom)
legacy_dict["version"] = "1.0"

# Python 3.5+ (PEP 448): 辞書リテラル内で非破壊的に1行で構築
modern_dict = {**defaults, **custom, "version": "1.0"}

print("Modern list:", modern_list)
print("Modern dict:", modern_dict)
```

---

## 4. 最小コード例

```python
default_config = {"host": "localhost", "port": 8080, "debug": False}
user_config = {"port": 9000, "debug": True}

# 辞書のマージ（user_configが優先）
final_config = {**default_config, **user_config, "version": "1.0"}
print("Config:", final_config)

# リストの結合
base_items = [1, 2]
extra_items = (3, 4)
all_items = [0, *base_items, *extra_items, 5]
print("All items:", all_items)
```

### 実行出力
```text
Config: {'host': 'localhost', 'port': 9000, 'debug': True, 'version': '1.0'}
All items: [0, 1, 2, 3, 4, 5]
```

---

## 5. 実務上の指針と落とし穴

- **辞書結合演算子（Python 3.9+）との関係**:
  Python 3.9 で辞書マージ演算子 `d1 | d2`（PEP 584）が導入されました。現在では辞書の単純な結合には `|` の方が好まれる傾向がありますが、インラインで新しいキーを追加しながら結合する際には `{**d1, "new": 1}` が依然として重宝されます。
- **キーの競合と優先順位**:
  辞書アンパックでは、同じキーが存在する場合に「右側（後からアンパックされた側）」の値で常に上書きされます。デフォルト設定とユーザー設定をマージする際は記述順序に注意が必要です。
  ```python
  defaults = {"timeout": 30, "retry": 3}
  custom = {"timeout": 10}

  # custom を右側に置くことでデフォルト値を上書き
  config = {**defaults, **custom}  # {'timeout': 10, 'retry': 3}
  ```

---

## 6. 関連リンク

- [[versions/py35|Python 3.5 概要]]
- [[language/py39/501-dict-union-operators|501-dict-union-operators]]

---

## 7. 参考資料

- [PEP 448 – Additional Unpacking Generalizations](https://peps.python.org/pep-0448/)
