---
title: "代入式（Walrus Operator :=）"
aliases: [セイウチ演算子, walrus operator, 代入式, PEP 572]
category: language
python_version: "3.8"
peps: [572]
created: 2026-10-26
updated: 2026-10-26
tags: [py38, language, syntax, expressions]
---

# 代入式（Walrus Operator :=）

## 1. 概要と動機

Pythonでは従来、代入文（`name = expr`）は「文（Statement）」であり、「式（Expression）」ではありませんでした。そのため、以下のシチュエーションでコードが冗長化していました。
- `while` ループの条件判定前に一度読み込み、ループ末尾で再度読み込む重複コード。
- `if` 分岐で正規表現マッチや計算結果を変数に保存しつつ条件判定したい場合に、`if` の手前で事前に変数定義が必要。
- リスト内包表記で高コストな計算結果を複数回再利用したい場合に、二重内包表記や外部ループが必要。

PEP 572 は、式の中で値を変数に代入できる代入演算子 `:=`（見た目がセイウチの牙と目に似ていることからセイウチ演算子 / Walrus Operatorと呼ばれる）を導入しました。

---

## 2. 仕様と構文

- **構文**: `NAME := expr`
- **動作**: `expr` を評価してその値を `NAME` にバインドし、同時にその評価値を式全体の値として返す。
- **優先順位**: 代入演算子は他の多くの演算子よりも優先度が低いため、明示的に丸括弧 `(x := ...)` で囲むことが推奨・要求されるケースが多い。

```python
# while ループや if 判定内での代入式
while (line := file.readline()) != "":
    process(line)
```

---

## 3. 従来記法との比較

Python 3.7 以前の事前代入方式と、Python 3.8 の代入式（`:=`）の比較です。

```python
import re

pattern = re.compile(r"^id:\s*(\d+)$")
line = "id: 1042"

# --- 従来（Python 3.7以前）: if文の手前で事前代入が必要 ---
# 変数 match のスコープが広く残り、代入と条件判定が2行に分かれる
match_leg = pattern.match(line)
if match_leg:
    val_leg = int(match_leg.group(1))
    print("Legacy matched:", val_leg)

# --- Python 3.8+: 代入式 := ---
# 条件判定の式の中で直接代入を行い、行数を削減し意図を明確化
if match_mod := pattern.match(line):
    val_mod = int(match_mod.group(1))
    print("Modern matched:", val_mod)
```

---

## 4. 最小コード例

Python 3.8 以降で動作する、正規表現マッチおよび内包表記での `:=` の活用例です。

```python
import re

text_samples = ["id: 1042", "invalid line", "id: 9981"]
pattern = re.compile(r"^id:\s*(\d+)$")

# if文内での代入と判定
matched_ids = []
for sample in text_samples:
    if match := pattern.match(sample):
        matched_ids.append(int(match.group(1)))

print("Matched IDs:", matched_ids)

# 内包表記内での計算結果の再利用
numbers = [1, 2, 3, 4, 5]
squares = [y for x in numbers if (y := x * x) > 10]
print("Squares > 10:", squares)
```

### 実行出力
```text
Matched IDs: [1042, 9981]
Squares > 10: [16, 25]
```

---

## 5. 実務上の指針と落とし穴

- **過剰な使用による可読性低下**:
  1行に詰め込みすぎるとコードの可読性が著しく低下します。「同じ計算の重複を防ぐ」「ループの初期化と更新の二重化を防ぐ」目的に絞って使用するのがベストプラクティスです。
- **内包表記のスコープ**:
  内包表記内で `:=` を使ってバインドされた変数は、内包表記を抜けた外側のスコープ（関数スコープ等）に漏れ出します。意図しないシャドーイングや変数の上書きに注意が必要です。
  ```python
  # 内包表記の外側でも変数が参照可能（スコープの漏れ）
  results = [val for x in [1, 2, 3] if (val := x * 10) > 15]
  print(val)  # 30 が出力される（外側のスコープに束縛が残る）
  ```
- **代入文との使い分け**:
  最上位の単純代入（`x := 10`）は、通常の代入文 `x = 10` と比較して推奨されず、Linter（Ruff / flake8等）で警告される場合があります。

---

## 6. 関連リンク

- [[versions/py38|Python 3.8 概要]]
- [[language/py38/402-positional-only-parameters|402-positional-only-parameters]]
- [[language/py310/601-pattern-matching|601-pattern-matching]]

---

## 7. 参考資料

- [PEP 572 – Assignment Expressions](https://peps.python.org/pep-0572/)
- [Python 3.8 What's New: PEP 572](https://docs.python.org/3/whatsnew/3.8.html#assignment-expressions)
