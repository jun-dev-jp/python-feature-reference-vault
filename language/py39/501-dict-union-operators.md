---
title: "辞書結合・更新演算子（| と |=）"
aliases: [辞書マージ演算子, PEP 584, dict union]
category: language
python_version: "3.9"
peps: [584]
created: 2026-10-26
updated: 2026-10-26
tags: [py39, language, syntax, dict, operators]
---

# 辞書結合・更新演算子（| と |=）

## 1. 概要と動機

辞書のマージ操作において、従来の `d1.update(d2)` は破壊的（in-place）な変更であり、非破壊的なマージには `{**d1, **d2}` というアンパックハックが必要でした。しかし、この記法は直感的でなく型も失われやすいという指摘がありました。

PEP 584 は、集合（`set`）型と同様にパイプ演算子 `|` および `|=` を `dict` クラスに実装し、自然で簡潔なマージとインプレース更新を可能にしました。

---

## 2. 仕様と構文

- **新結合 (`d1 | d2`)**: 2つの辞書を結合した新しい `dict` オブジェクトを返す。キーの競合時は右辺（`d2`）の値が優先される。
- **インプレース更新 (`d1 |= d2`)**: `d1` を直接更新する（`d1.update(d2)` と同等）。右辺にはマッピングだけでなくキー・バリューのイテラブルも指定可能。

```python
# 辞書マージ演算子の基本利用
base = {"a": 1, "b": 2}
extra = {"b": 99, "c": 3}
merged = base | extra        # 新しい辞書を生成（b は 99）
base |= extra               # base を直接破壊的更新
```

---

## 3. 従来記法との比較

Python 3.8 以前の辞書マージ手法と、Python 3.9 のパイプ演算子（`|` / `|=`）の比較です。

```python
d1 = {"a": 1, "b": 2}
d2 = {"b": 99, "c": 3}

# --- 従来 1: {**d1, **d2}（非破壊的だが直感的でなく型が dict に限定されがち） ---
legacy_merged = {**d1, **d2}

# --- 従来 2: copy() + update()（複数行の手続きが必要） ---
legacy_update = d1.copy()
legacy_update.update(d2)

# --- Python 3.9+: 辞書結合演算子 | および インプレース更新 |= ---
modern_merged = d1 | d2
d1 |= d2  # d1 を直接更新

print("Legacy merged:", legacy_merged)
print("Modern merged:", modern_merged)
```

---

## 4. 最小コード例

```python
defaults = {"theme": "light", "notifications": True, "limit": 20}
user_settings = {"theme": "dark", "limit": 50}

# 非破壊マージ
combined = defaults | user_settings
print("Combined:", combined)
print("Defaults unchanged:", defaults["theme"])

# インプレース更新
defaults |= {"notifications": False}
print("Updated defaults:", defaults)
```

### 実行出力
```text
Combined: {'theme': 'dark', 'notifications': True, 'limit': 50}
Defaults unchanged: light
Updated defaults: {'theme': 'light', 'notifications': False, 'limit': 20}
```

---

## 5. 実務上の指針と落とし穴

- **型の維持**:
  `defaultdict` やカスタムの `dict` サブクラスで `|` を呼ぶと、左辺の型が保持される設計になっています。
- **Python 3.8以前との互換性**:
  古い環境では `TypeError` となるため、3.8以前をターゲットにするライブラリでは `{**d1, **d2}` を維持する必要があります。
- **右辺優先の評価順序**:
  同一キーが存在する場合、必ず右辺の値で上書きされます。デフォルト値と上書き値の配置順に注意してください。
  ```python
  defaults = {"timeout": 30}
  options = {"timeout": 10}

  # options が右辺にあるため timeout は 10 になる
  active_cfg = defaults | options  # {'timeout': 10}
  ```

---

## 6. 関連リンク

- [[versions/py39|Python 3.9 概要]]
- [[language/py35/103-unpacking-generalizations|103-unpacking-generalizations]]
- [[typing/py310/602-union-operator|602-union-operator]]

---

## 7. 参考資料

- [PEP 584 – Add Union Operators To dict](https://peps.python.org/pep-0584/)
