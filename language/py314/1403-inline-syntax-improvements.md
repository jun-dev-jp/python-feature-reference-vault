---
title: "複数インライン文・構文の刷新（3.14 構文拡張構想）"
aliases: [インライン文構想, 3.14構文刷新, PEP 779]
category: language
python_version: "3.14"
peps: [779]
created: 2026-10-26
updated: 2026-10-26
tags: [py314, language, syntax, grammar, experimental]
---

# 複数インライン文・構文の刷新（3.14 構文拡張構想）

## 1. 概要と動機

Pythonはコードブロックの構造化にインデント（オフサイドルール）を厳格に適用していますが、単純なガード節やショートサーキット文（例: `if condition: return val`）をインラインで簡潔に記述する際の規則や、セミコロン `;` を含む複数文の混在について、長年パーサの不整合やLinter間の解釈不一致が存在していました。

Python 3.14 では新PEGパーサの表現力をさらに洗練させ、複合文におけるインライン記述の曖昧性を解消し、構文規則の合理化を進めています。

---

## 2. 仕様と構文

- **構文検証の厳格化**:
  インラインでの複数文連結や不自然なセミコロン使用に関するパーサエラーメッセージの親切化。
- **トレースバック・ASTノードの精密化**:
  インラインで複数の処理が1行に記述されている場合でも、エラー発生箇所の正確なカラム範囲（下線ハイライト）が特定される。

```python
# ガード節のインライン記述
if condition: return False
```

---

## 3. 最小コード例

```python
def process_flag(flag: bool) -> str:
    # 簡潔なガード節のインライン記述例
    if not flag: return "aborted"
    return "proceeded"

if __name__ == "__main__":
    print(process_flag(False))
    print(process_flag(True))
```

### 実行出力
```text
aborted
proceeded
```

---

## 4. 実務上の指針と落とし穴

- **PEP 8 スタイルガイド**:
  文法上許可される場合でも、1行に複数の複雑な文を詰め込むことは可読性を損ねるため、原則として各文をインデントブロックとして改行することが推奨されます。
  ```python
  # 可読性推奨（インデントブロック）
  if not flag:
      return "aborted"
  ```

---

## 5. 関連リンク

- [[versions/py314|Python 3.14 概要]]
- [[language/py314/1401-template-strings|1401-template-strings]]

---

## 6. 参考資料

- [Python 3.14 What's New (Draft)](https://docs.python.org/3.14/whatsnew/3.14.html)
