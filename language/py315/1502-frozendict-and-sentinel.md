---
title: "不変辞書 frozendict と sentinel 組み込み型"
aliases: [frozendict, sentinel, PEP 814, PEP 661]
category: language
python_version: "3.15"
peps: [814, 661]
created: 2026-10-26
updated: 2026-10-26
tags: [py315, language, builtins, immutability, data-structures]
---

# 不変辞書 frozendict と sentinel 組み込み型

## 1. 概要と動機

1. **`frozendict`（PEP 814）**:
   Pythonには `set` に対する `frozenset`、`list` に対する `tuple` が存在していましたが、`dict` に対する不変（イミュータブル）かつハッシュ可能な組み込み型が長年欠落していました。従来の `types.MappingProxyType` は元の辞書の変更を反映してしまう読み取り専用ビューに過ぎず、純粋な不変性やハッシュ化が困難でした。
2. **`sentinel`（PEP 661）**:
   未指定引数や見つからない値の番兵値（Sentinel value）として `object()` を用いる慣行がありましたが、`repr` が `<object object at 0x...>` となり読みにくく、コピーや pickle 時に同一性が失われるという課題がありました。

Python 3.15 ではこれらが組み込み（`builtins`）型として正式に導入されました。

---

## 2. 仕様と構文

- **`frozendict(**kwargs)` / `frozendict(mapping)`**:
  - `builtins` に直接配置。`dict` のサブクラスではなく `object` から直接継承。
  - 生成後の要素の追加・削除・更新は不可（`TypeError`）。
  - キーと値の双方がハッシュ可能である場合、`frozendict` 自体もハッシュ可能（辞書キーや集合の要素に利用可能）。
  - 挿入順序は保持されるが、同値比較（`==`）において順序の違いは同一とみなされる。
- **`sentinel(name, repr=None, *, module=None)`**:
  - 一意な番兵オブジェクトを生成。
  - `str` や `repr` で指定された名前が表示される。型式の中で `int | MISSING` のように使用可能。

```python
# 組み込み不変辞書と番兵値の基本宣言
EMPTY = sentinel("EMPTY")
static_config = frozendict({"host": "localhost", "port": 8080})
```

---

## 3. 最小コード例

Python 3.15 組み込みの `frozendict` と `sentinel` の利用例です。

```python
# Python 3.15 builtins
MISSING = sentinel("MISSING")

# 不変辞書の作成
config = frozendict(env="production", workers=4)
print("Config:", config)

# ハッシュ可能であるため辞書のキーにできる
cache = {config: "CacheEntry#1"}
print("Cache lookup:", cache[frozendict(workers=4, env="production")])

# 番兵値の利用
def get_setting(key: str, default: object = MISSING) -> object:
    if key not in config:
        if default is MISSING:
            raise KeyError(f"Setting '{key}' is required")
        return default
    return config[key]

print("Env setting:", get_setting("env"))
print("MISSING repr:", repr(MISSING))
```

### 実行出力
```text
Config: frozendict({'env': 'production', 'workers': 4})
Cache lookup: CacheEntry#1
Env setting: production
MISSING repr: <sentinel 'MISSING'>
```

---

## 4. 実務上の指針と落とし穴

- **`isinstance(x, dict)` の判定**:
  `frozendict` は `dict` のサブクラスではないため、`isinstance(x, dict)` では `False` になります。辞書風のオブジェクト全般を受け付ける関数では `isinstance(x, collections.abc.Mapping)` または `isinstance(x, (dict, frozendict))` と記述してください。
  ```python
  from collections.abc import Mapping

  # 辞書全般を安全に受け付ける型判定
  def dump(data: Mapping):
      assert isinstance(data, Mapping)
  ```
- **ミュータブルな値を格納した場合**:
  値にリスト等のミュータブルオブジェクトを含んでいる場合、`frozendict` 自体への代入は防げますが、内部リストの変更は防げず、`hash()` を呼ぶと `TypeError: unhashable type` になります。

---

## 5. 関連リンク

- [[versions/py315|Python 3.15 概要]]
- [[language/py315/1501-lazy-imports|1501-lazy-imports]]
- [[typing/py315/1504-typeform-and-typeddict-extra|1504-typeform-and-typeddict-extra]]

---

## 6. 参考資料

- [PEP 814 – Add frozendict built-in type](https://peps.python.org/pep-0814/)
- [PEP 661 – Sentinel Values](https://peps.python.org/pep-0661/)
- [What's New In Python 3.15: frozendict & sentinel](https://docs.python.org/3.15/whatsnew/3.15.html#whatsnew315-frozendict)
