---
title: "テンプレート文字列（t-strings: PEP 750）"
aliases: [t-strings, テンプレート文字列, PEP 750, Template Strings]
category: language
python_version: "3.14"
peps: [750]
created: 2026-10-26
updated: 2026-10-26
tags: [py314, language, syntax, strings, security, experimental]
---

# テンプレート文字列（t-strings: PEP 750）

## 1. 概要と動機

Python 3.6で導入された f-strings は非常に便利ですが、文字列構築時に即座に式が評価され通常の `str` オブジェクトに結合されます。このため、**SQLクエリ、HTML、OSシェルコマンド**などの構築に使用すると、エスケープ漏れによるSQLインジェクションやXSSなどの重大なセキュリティ脆弱性を引き起こす原因となっていました。

PEP 750 が提案するテンプレート文字列（プレフィックス `t"..."`）は、JavaScriptのタグ付きテンプレートリテラル（Tagged Templates）に類似した仕組みをPythonに提供します。即座に文字列化せず、**構文解析された構造（文字列リテラル片と式の値・コンテキスト）を持つオブジェクト**を生成し、専用のハンドラ関数に引き渡して安全に処理できるようにします。

---

## 2. 仕様と構文

> [!NOTE] バージョンステータス: 採択・開発中（Accepted / In Development）
> Python 3.14 におけるコア実装が進められており、正式リリースに向けて細部のAPI仕様が確定・調整されます。

- **構文**: `t"Hello, {name}!"`
- **生成オブジェクト**: `Template` インスタンス
  - 文字列部分（静的なプレースホルダ周辺テキスト）
  - 補間部分（`Interpolation` オブジェクト: 式の評価値、式表現、フォーマット指定子）
- **利用モデル**:
  ```python
  def sql(template: Template) -> tuple[str, list[Any]]:
      # 静的SQLとパラメータリストを安全に分離生成
      ...
  ```

---

## 3. 最小コード例

Python 3.14 テンプレート文字列（PEP 750 仕様準拠）による安全なSQLジェネレータの概念コード例です。

```python
# Python 3.14 (t-strings / PEP 750 概念実装)
class Interpolation:
    def __init__(self, value, expression: str):
        self.value = value
        self.expression = expression

class Template:
    def __init__(self, strings: tuple[str, ...], interpolations: tuple[Interpolation, ...]):
        self.strings = strings
        self.interpolations = interpolations

def sql_escape(t: Template) -> tuple[str, list]:
    query_parts = []
    params = []
    for i, s in enumerate(t.strings):
        query_parts.append(s)
        if i < len(t.interpolations):
            query_parts.append("?")
            params.append(t.interpolations[i].value)
    return "".join(query_parts), params

if __name__ == "__main__":
    user_id = 42
    role = "admin"
    # t"SELECT * FROM users WHERE id = {user_id} AND role = {role}" のシミュレーション
    fake_template = Template(
        strings=("SELECT * FROM users WHERE id = ", " AND role = ", ""),
        interpolations=(Interpolation(user_id, "user_id"), Interpolation(role, "role"))
    )
    query, params = sql_escape(fake_template)
    print("Prepared Query:", query)
    print("Parameters:", params)
```

### 実行出力
```text
Prepared Query: SELECT * FROM users WHERE id = ? AND role = ?
Parameters: [42, 'admin']
```

---

## 4. 実務上の指針と落とし穴

- **f-strings との住み分け**:
  - `f"..."`: 単純なログ出力、CLIメッセージ、デバッグなど安全性が問題にならない場面で最高速に動作。
  - `t"..."`: SQLクエリ、HTMLレンダリング、GraphQLクエリ、正規表現など、エスケープや構文木構築を伴うDSL用途。
- **通常の文字列としての直接利用不可**:
  `Template` オブジェクトは自動的に `str` に暗黙変換されないため、`str(t)` やハンドラを通さずに `open(t)` などへ渡すと `TypeError` となります。
  ```python
  # t-string は str ではないため直接ファイルオープン等は不可
  tmpl = t"data_{user_id}.txt"
  # open(tmpl)  # TypeError: expected str, bytes or os.PathLike object
  ```

---

## 5. 関連リンク

- [[versions/py314|Python 3.14 概要]]
- [[language/py36/201-fstrings|201-fstrings]]
- [[language/py312/802-syntactic-formalization-fstrings|802-syntactic-formalization-fstrings]]

---

## 6. 参考資料

- [PEP 750 – Template Strings](https://peps.python.org/pep-0750/)
- [Python 3.14 What's New (Draft)](https://docs.python.org/3.14/whatsnew/3.14.html)
