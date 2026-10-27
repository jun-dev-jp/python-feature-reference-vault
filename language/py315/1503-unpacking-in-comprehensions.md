---
title: "内包表記・ジェネレータでのアンパック構文（PEP 798）"
aliases: [内包表記アンパック, PEP 798, unpacking in comprehensions]
category: language
python_version: "3.15"
peps: [798]
created: 2026-10-26
updated: 2026-10-26
tags: [py315, language, syntax, unpacking, comprehensions]
---

# 内包表記・ジェネレータでのアンパック構文（PEP 798）

## 1. 概要と動機

リストのリストや辞書のリストを平坦化（Flatten）する際、Pythonでは二重の `for` 節（`[x for row in matrix for x in row]`）や `itertools.chain.from_iterable()` を記述する必要がありました。しかし、直感的なアンパック構文 `*` や `**` は式リテラル内（PEP 448）でのみ許されており、内包表記内では使えませんでした。

Python 3.15 は **PEP 798** により、リスト内包表記、セット内包表記、辞書内包表記、ジェネレータ式において `*` および `**` によるアンパックを正式にサポートしました。

---

## 2. 仕様と構文

- **リスト内包表記**: `[*row for row in matrix]`（`[x for row in matrix for x in row]` と等価）
- **セット内包表記**: `{*s for s in set_list}`
- **辞書内包表記**: `{**d for d in dict_list}`
- **ジェネレータ式**: `(*row for row in matrix)`
- **非同期ジェネレータ式**: `(*a async for a in agen())` にも対応。

```python
# 内包表記でのアンパック記述
flattened = [*sublist for sublist in nested_lists]
merged_map = {**d for d in dict_stream}
```

---

## 3. 従来記法との比較

Python 3.14 以前の平坦化・内包表記記法と、Python 3.15（PEP 798）のアンパック内包表記の比較です。

```python
matrix = [[1, 2], [3, 4]]
dict_list = [{"a": 1}, {"b": 2}]

# --- 1. リストの平坦化 ---
# 従来（Python 3.14以前）: 二重 for 節（順序が直感と逆に見えやすい）
legacy_flat = [item for row in matrix for item in row]

# Python 3.15+: リスト内包表記内での * アンパック
modern_flat = [*row for row in matrix]

# --- 2. 辞書リストの統合 ---
# 従来（Python 3.14以前）: functools.reduce や 辞書内包表記での複雑な展開が必要
import functools
legacy_merged = functools.reduce(lambda acc, d: acc | d, dict_list, {})

# Python 3.15+: 辞書内包表記内での ** アンパック
modern_merged = {**d for d in dict_list}

print("Flat:", modern_flat)
print("Merged:", modern_merged)
```

---

## 4. 最小コード例

```python
# Python 3.15 PEP 798 内包表記アンパック
matrix = [[1, 2], [3, 4], [5]]
flattened = [*row for row in matrix]
print("Flattened list:", flattened)

tag_sets = [{"python", "docs"}, {"vault", "python"}]
unique_tags = {*s for s in tag_sets}
print("Unique tags:", unique_tags)

overrides = [{"env": "dev", "debug": True}, {"debug": False, "port": 8000}]
merged_dict = {**d for d in overrides}
print("Merged dict:", merged_dict)
```

### 実行出力
```text
Flattened list: [1, 2, 3, 4, 5]
Unique tags: {'docs', 'python', 'vault'}
Merged dict: {'env': 'dev', 'debug': False, 'port': 8000}
```

---

## 5. 実務上の指針と落とし穴

- **可読性の向上**:
  `itertools.chain` をインポートすることなく、リストの平坦化が直感的な1行で記述可能になります。
- **後方互換性**:
  Python 3.14 以前のバージョンでは `SyntaxError` となるため、3.14以前をサポートするコードベースでは従来の二重ループ内包表記を使用してください。
  ```python
  # Python 3.14 以前との互換記法
  flattened = [x for row in matrix for x in row]
  ```

---

## 6. 関連リンク

- [[versions/py315|Python 3.15 概要]]
- [[language/py35/103-unpacking-generalizations|103-unpacking-generalizations]]
- [[language/py39/501-dict-union-operators|501-dict-union-operators]]

---

## 7. 参考資料

- [PEP 798 – Unpacking in Comprehensions](https://peps.python.org/pep-0798/)
- [What's New In Python 3.15: PEP 798](https://docs.python.org/3.15/whatsnew/3.15.html#whatsnew315-unpacking-in-comprehensions)
