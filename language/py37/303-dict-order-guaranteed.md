---
title: "辞書順序の言語仕様としての保証"
aliases: [辞書順序保証, dict order guaranteed, 挿入順序]
category: language
python_version: "3.7"
peps: []
created: 2026-10-26
updated: 2026-10-26
tags: [py37, language, dict, data-structures]
---

# 辞書順序の言語仕様としての保証

## 1. 概要と動機

Python 3.5 以前の辞書（`dict`）は、ハッシュテーブルの内部スロットの空き状況に応じて要素が配置されていたため、キーの挿入順序が保持されず、イテレーション順序は実質ランダムでした（順序保持には `collections.OrderedDict` が必要）。

Python 3.6 において CPython 実装のコンパクトハッシュテーブル刷新により副次的に順序が保持されるようになり、**Python 3.7 においてすべてのPython実装（PyPy等を含む）が満たすべき言語仕様として正式に保証**されました。

---

## 2. 仕様と構文

- **保証内容**: `dict` インスタンスは、キーが挿入された順序（Insertion Order）をイテレーション時およびイテレータ取得時に完全に保持する。
- **影響範囲**: `dict.keys()`, `dict.values()`, `dict.items()`, およびキーワード引数辞書（`**kwargs`）。
- **`OrderedDict` との差分**:
  通常の `dict` で大半の用途は満たせますが、逆順イテレーションや `move_to_end()`、厳格な順序付き同値比較（順序が異なると等しくないと判定）を行う場合は現在でも `collections.OrderedDict` を使用します。

```python
# 辞書生成時の挿入順序保持
mapping = {"first": 1, "second": 2, "third": 3}
# keys() の順序は常に挿入順 ['first', 'second', 'third']
```

---

## 3. 最小コード例

```python
data = {}
data["banana"] = 100
data["apple"] = 200
data["orange"] = 150

print("Keys in insertion order:", list(data.keys()))
print("Iterating items:")
for k, v in data.items():
    print(f" - {k}: {v}")
```

### 実行出力
```text
Keys in insertion order: ['banana', 'apple', 'orange']
Iterating items:
 - banana: 100
 - apple: 200
 - orange: 150
```

---

## 4. 実務上の指針と落とし穴

- **JSONシリアライズの安定性**:
  辞書順序の保証により、JSON等のシリアライズ結果のキー順序が予測可能になり、デバッグやキャッシュの再現性が格段に向上しました。
- **削除と再挿入**:
  一度キーを `del` または `pop()` で削除し、後から同じキーを再代入した場合、そのキーの順序は**末尾**に移動します。
  ```python
  d = {"a": 1, "b": 2}
  del d["a"]       # "a" を削除
  d["a"] = 10      # 再代入すると順序は ['b', 'a'] の末尾になる
  print(list(d))   # ['b', 'a']
  ```

---

## 5. 関連リンク

- [[versions/py37|Python 3.7 概要]]
- [[language/py39/501-dict-union-operators|501-dict-union-operators]]
- [[language/py315/1502-frozendict-and-sentinel|1502-frozendict-and-sentinel]]

---

## 6. 参考資料

- [Python 3.7 What's New: Dict insertion order guarantee](https://docs.python.org/3/whatsnew/3.7.html)
