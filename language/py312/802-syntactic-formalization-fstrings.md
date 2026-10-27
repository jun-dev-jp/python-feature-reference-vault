---
title: "f-strings の文法制限緩和（PEP 701）"
aliases: [f-strings構文緩和, PEP 701, ネストf-strings]
category: language
python_version: "3.12"
peps: [701]
created: 2026-10-26
updated: 2026-10-26
tags: [py312, language, syntax, strings, fstrings]
---

# f-strings の文法制限緩和（PEP 701）

## 1. 概要と動機

Python 3.6 で導入された f-strings は、字句解析器（Tokenizer）レベルの特殊なハックとして実装されていたため、以下の不自然な構文制限が存在していました。
- 外側と同じクォート記号を波括弧 `{}` 内で使用できない（例: `f"{data['key']}"` で外側をダブル、内側をシングルに分ける必要があった）。
- 式の内部でバックスラッシュ（`\n` など）を記述できない。
- 式の内部にコメント（`#`）を書けない。
- f-strings の多段ネストが極めて制限されていた。

PEP 701 は、f-strings をPythonの公式構文規則（PEGパーサ）に正式に組み込み、これらの制約を完全に撤廃しました。

---

## 2. 仕様と構文

- **引用符の再利用**: 外側の文字列と同じクォート（`"` や `'`）を内側の式で自由に使用可能。
- **バックスラッシュ解禁**: `{'\n'.join(lines)}` のように式内部でエスケープシーケンスを直接記述可能。
- **複数行コメントと無制限ネスト**: 式の内部にコメントや任意の深さのネストした f-strings を記述可能。

```python
# 同一クォートのネストとバックスラッシュ利用
output = f"Value: {data["key"] + '\n'}"  # 外側・内側ともにダブルクォート
```

---

## 3. 従来制限との比較

Python 3.11 以前で構文エラーとなっていたパターンと、Python 3.12（PEP 701）で正式に解禁された記法の比較です。

```python
data = {"title": "Python 3.12 Guide", "author": "Alice"}
items = ["apple", "banana"]

# --- 従来（Python 3.11以前）の制約 ---
# 1. クォートの使い分けが必要: f"{data['title']}" のように内外で引用符を変えなければ SyntaxError
# 2. バックスラッシュ禁止: f"{'\n'.join(items)}" は構文エラーになるため、事前に改行文字を変数化する必要があった
newline = "\n"
legacy_joined = f"{newline.join(items)}"
legacy_quote = f"{data['title']} by {data['author']}"

# --- Python 3.12+: PEP 701 による制限撤廃 ---
# 1. 内外で同一のクォート記号をそのまま再利用可能
# 2. 波括弧内でバックスラッシュ（\n 等）を直接記述可能
modern_joined = f"{'\n'.join(items)}"
modern_quote = f"{data["title"]} by {data["author"]}"

print("Modern quote:", modern_quote)
print("Modern joined:\n" + modern_joined)
```

---

## 4. 最小コード例

Python 3.12 以降で動作する、制限緩和された f-strings の例です。

```python
records = {"title": "Vault Guide", "tags": ["python", "obsidian"]}

# 同じダブルクォートを内側で利用し、バックスラッシュ改行をjoinする
result = f"Article: {records["title"]}\nTags:\n{'\n'.join(f" - {tag.upper()}" for tag in records["tags"])}"

print(result)
```

### 実行出力
```text
Article: Vault Guide
Tags:
 - PYTHON
 - OBSIDIAN
```

---

## 5. 実務上の指針と落とし穴

- **Python 3.11以前との互換性**:
  Python 3.11以下の環境でこのコードを実行すると `SyntaxError` になります。ライブラリ開発などで 3.11以前をサポートする場合は、従来の「外側と内側で異なるクォートを使用する」ルールを維持してください。
  ```python
  # Python 3.11 以前との互換記法
  legacy = f"Key: {data['key']}"  # 外側ダブル、内側シングルで区別
  ```

---

## 6. 関連リンク

- [[versions/py312|Python 3.12 概要]]
- [[language/py36/201-fstrings|201-fstrings]]
- [[language/py314/1401-template-strings|1401-template-strings]]

---

## 7. 参考資料

- [PEP 701 – Syntactic formalization of f-strings](https://peps.python.org/pep-0701/)
- [Python 3.12 What's New: PEP 701](https://docs.python.org/3/whatsnew/3.12.html#pep-701-syntactic-formalization-of-f-strings)
